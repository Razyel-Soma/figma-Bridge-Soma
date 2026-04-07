# review-gates

Skill de verificación automática que corre después de cualquier skill de generación. Evalúa calidad, detecta problemas y decide si el resultado pasa o necesita re-trabajo. Puede usarse standalone o como parte de `run-mission`.

## Cuando usar este skill
- Automáticamente después de cualquier skill de generación (si se usa con `run-mission`)
- Manualmente cuando quieres verificar calidad antes de continuar
- Como QA rápido antes de un handoff

## Gates disponibles

### Gate 1: `tokens` — Verificar sistema de diseño
**Cuándo:** Después de `setup-radix-base`, `generate-industry`
**Qué verifica:**
```
- figma_execute (timeout: 10000) → figma.variables.getLocalVariablesAsync()
  ✅ PASS: >= 20 variables en al menos 3 colecciones
  ⚠️ WARNING: 10-19 variables (DS parcial)
  ❌ FAIL: < 10 variables (DS incompleto)
```

### Gate 2: `components` — Verificar componentes creados
**Cuándo:** Después de `generate-library`, `generate-industry`
**Qué verifica:**
```
- figma_search_components → listar componentes
- figma_execute (timeout: 15000) → para cada componente:
  - ¿Tiene component properties? (TEXT, BOOLEAN, INSTANCE_SWAP)
  - ¿Usa variables del DS? (boundVariables en fills)
  - ¿Tiene Auto Layout?
  - ¿Nombre correcto? (no "Component 1")

  ✅ PASS: Todos los componentes tienen properties + variables
  ⚠️ WARNING: Componentes sin properties o sin variables
  ❌ FAIL: 0 componentes encontrados
```

### Gate 3: `screen` — Verificar pantalla generada
**Cuándo:** Después de `generate-screen`, `generate-onboarding`
**Qué verifica:**
```
- figma_capture_screenshot → inspección visual
- figma_execute (timeout: 20000) → escanear el frame:

  LAYOUT:
  - ¿Frame colapsado? (height < 20px) → CRITICAL
  - ¿Elementos fuera del frame? (x < 0 || x > frame.width) → CRITICAL
  - ¿Overlaps entre elementos hermanos? → WARNING

  TOKENS:
  - ¿Colores hardcodeados sin boundVariables? → WARNING
  - ¿Fuentes no disponibles en el archivo? → WARNING

  ACCESIBILIDAD:
  - ¿Touch targets < 44px? (botones, inputs, links) → CRITICAL
  - ¿Textos < 12px? → WARNING
  - ¿Contraste estimado insuficiente? → WARNING

  NOMENCLATURA:
  - ¿Capas con nombres genéricos? (Frame N, Rectangle N, Group N, Ellipse N) → WARNING

  RESULTADO:
  ✅ PASS: 0 critical, ≤3 warnings
  ⚠️ PASS con warnings: 0 critical, >3 warnings
  ❌ FAIL: ≥1 critical → trigger retry
```

### Gate 4: `a11y` — Verificar specs de accesibilidad
**Cuándo:** Después de `create-voice`
**Qué verifica:**
```
- ¿Se generó orden de lectura? → CRITICAL si no
- ¿Todos los elementos interactivos tienen label? → CRITICAL
- ¿Elementos decorativos marcados como aria-hidden? → WARNING
- ¿Touch targets documentados? → WARNING

  ✅ PASS: Orden de lectura + labels completos
  ⚠️ WARNING: Faltan decorativos marcados
  ❌ FAIL: Sin orden de lectura o sin labels
```

### Gate 5: `handoff` — Verificar preparación para desarrollo
**Cuándo:** Después de `prepare-handoff`
**Qué verifica:**
```
- ¿Capas renombradas? (0 nombres genéricos) → WARNING si quedan
- ¿Metadata de handoff guardada? → WARNING si no
- ¿Tokens documentados? → INFO

  ✅ PASS: 0 nombres genéricos + metadata presente
  ⚠️ WARNING: Quedan nombres genéricos
  ❌ FAIL: No se ejecutó el handoff
```

## Lógica de retry

Cuando un gate retorna FAIL:

```
Intento 1: Identificar qué falló → regenerar SOLO esos elementos → re-verificar
Intento 2: Si sigue fallando → regenerar con enfoque diferente → re-verificar
Intento 3: NO HAY. Después de 2 reintentos, reportar al usuario:

"❌ Gate [nombre] falló después de 2 intentos.
Issues pendientes:
- [lista de issues críticos]

Opciones:
1. Reintentar manualmente
2. Saltar este gate y continuar
3. Detener la misión"
```

**Regla de reintentos:** Solo regenerar lo que falló, no toda la fase. Si un input está colapsado, regenerar ESE input, no toda la pantalla.

## Patrón de código para verificación de pantalla

```javascript
// ✅ Gate: screen — timeout: 20000
const frame = await figma.getNodeByIdAsync("FRAME_ID");
const issues = { critical: [], warning: [], info: [] };

function check(node, depth = 0) {
  // Frame colapsado
  if (node.height < 20 && node.type === 'FRAME' && node.children?.length > 0) {
    issues.critical.push({ id: node.id, name: node.name, issue: 'collapsed', height: node.height });
  }

  // Touch target pequeño
  const nameLower = (node.name || '').toLowerCase();
  const isInteractive = nameLower.includes('button') || nameLower.includes('btn') ||
    nameLower.includes('input') || nameLower.includes('link') || nameLower.includes('tab');
  if (isInteractive && node.height < 44) {
    issues.critical.push({ id: node.id, name: node.name, issue: 'touch-target', height: node.height });
  }

  // Texto pequeño
  if (node.type === 'TEXT' && node.fontSize < 12) {
    issues.warning.push({ id: node.id, name: node.name, issue: 'small-text', fontSize: node.fontSize });
  }

  // Nombre genérico
  if (/^(Frame|Rectangle|Group|Ellipse|Line|Vector)\s*\d*$/i.test(node.name)) {
    issues.warning.push({ id: node.id, name: node.name, issue: 'generic-name' });
  }

  // Colores hardcodeados (sin variable)
  if (node.fills && node.fills.length > 0 && node.fills[0].type === 'SOLID') {
    if (!node.boundVariables?.fills) {
      issues.warning.push({ id: node.id, name: node.name, issue: 'hardcoded-fill' });
    }
  }

  if ('children' in node) {
    for (const child of node.children) {
      if (child.visible !== false) check(child, depth + 1);
    }
  }
}

check(frame);

const result = issues.critical.length === 0 ? 'PASS' : 'FAIL';
return {
  result,
  critical: issues.critical.length,
  warnings: issues.warning.length,
  issues: {
    critical: issues.critical.slice(0, 10),
    warning: issues.warning.slice(0, 10)
  }
};
```

## Formato de reporte del gate

```
## 🔍 Review Gate: screen

**Resultado:** ✅ PASS (2 warnings)

### Critical (0)
Ninguno

### Warnings (2)
- `line` (1:45) — nombre genérico → renombrar a "separator/divider"
- `input-email` (1:52) — color hardcodeado → bindear a variable

### Recomendaciones
- Correr `prepare-handoff` para arreglar nombres automáticamente
```

## Integración con run-mission

`run-mission` llama a review-gates automáticamente. Pero el usuario también puede invocar un gate manualmente:

```
"Corre un review gate en la pantalla de login"
→ use_skill review-gates → gate: screen sobre el frame seleccionado

"Verifica que el DS está completo"
→ use_skill review-gates → gate: tokens

"Audita los componentes de la librería"
→ use_skill review-gates → gate: components
```

## Ejemplos de uso
- "Verifica la pantalla que acabo de crear"
- "¿Pasa el review gate de accesibilidad?"
- "Corre todos los gates sobre esta página"
- "¿Los componentes tienen properties correctas?"
