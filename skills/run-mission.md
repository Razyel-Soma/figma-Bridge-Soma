# run-mission

Skill orquestador que ejecuta múltiples skills en secuencia automática con resolución de dependencias y review gates. Recibe un plan de misión (de `mission-planner` o directo del usuario) y ejecuta cada fase verificando calidad entre pasos.

## Cuando usar este skill
- Cuando el usuario pide algo que requiere múltiples skills encadenados
- Después de `mission-planner` cuando el plan está aprobado
- Cuando el usuario pide un pipeline completo: "hazme todo el DS"

## Pipelines predefinidos

### Pipeline: `ds-completo` — Sistema de diseño desde cero
```
Phase 1: setup-radix-base        | deps: none      | gate: verify-variables
Phase 2: generate-industry       | deps: [1]       | gate: verify-components
Phase 3: generate-library        | deps: [1]       | gate: review-quality
Phase 4: generate-screen         | deps: [2,3]     | gate: review-quality
Phase 5: create-voice            | deps: [4]       | gate: none
Phase 6: prepare-handoff         | deps: [4,5]     | gate: none
Phase 7: generate-showcase-page  | deps: [3,5,6]   | gate: none
```

### Pipeline: `pantallas` — Generar pantallas con DS existente
```
Phase 1: verify-ds               | deps: none      | gate: ds-exists
Phase 2: generate-screen         | deps: [1]       | gate: review-quality
Phase 3: create-voice            | deps: [2]       | gate: none
Phase 4: prepare-handoff         | deps: [2,3]     | gate: none
```

### Pipeline: `libreria` — Crear librería de componentes
```
Phase 1: setup-radix-base        | deps: none      | gate: verify-variables
Phase 2: generate-library        | deps: [1]       | gate: review-quality
Phase 3: migrate-to-slots        | deps: [2]       | gate: none
Phase 4: generate-showcase-page  | deps: [2,3]     | gate: none
```

### Pipeline: `audit-completo` — Auditoría integral
```
Phase 1: audit-quality           | deps: none      | gate: none
Phase 2: create-voice            | deps: none      | gate: none
Phase 3: migrate-to-slots        | deps: none      | gate: none
Phase 4: prepare-handoff         | deps: [1,2,3]   | gate: none
```

### Pipeline: `custom` — El usuario define las fases
```
El usuario lista los skills y sus dependencias.
run-mission los ejecuta en el orden correcto.
```

## Pasos de ejecución

### Paso 1 — Recibir o seleccionar pipeline

Si viene de `mission-planner`:
```
Usar el plan de misión aprobado por el usuario.
```

Si el usuario pide directamente:
```
- "Hazme un DS completo de fintech" → pipeline: ds-completo + industry: fintech
- "Genera las pantallas del checkout" → pipeline: pantallas
- "Crea la librería desde mi código" → pipeline: libreria
- "Audita todo el archivo" → pipeline: audit-completo
```

Si no matchea ningún pipeline:
```
Construir un pipeline custom basado en lo que el usuario pide.
```

### Paso 2 — Resolver dependencias

Construir el grafo de ejecución. Las fases sin dependencias van primero:

```
Ejemplo para ds-completo:

Nivel 0: [setup-radix-base]              ← sin deps, ejecutar primero
Nivel 1: [generate-industry, generate-library]  ← dependen de nivel 0
Nivel 2: [generate-screen]                ← depende de nivel 1
Nivel 3: [create-voice]                   ← depende de nivel 2
Nivel 4: [prepare-handoff]               ← depende de nivel 2 y 3
Nivel 5: [generate-showcase-page]         ← depende de nivel 1, 3, 4
```

> Nota: Como la Plugin API de Figma es single-threaded, las fases del mismo nivel se ejecutan en secuencia (no en paralelo). Pero el orden de dependencias se respeta.

### Paso 3 — Ejecutar fases en orden

Para cada fase:

```
1. Anunciar al usuario: "⏳ Fase N/total: [skill] — [objetivo]"
2. Cargar el skill: use_skill [nombre]
3. Ejecutar el skill siguiendo sus instrucciones
4. Si la fase tiene review gate → ejecutar el gate (ver paso 4)
5. Registrar métricas: { skill, time, errors, retries, gate_result }
6. Anunciar resultado: "✅ Fase N completada" o "❌ Fase N falló"
7. Si falló y es bloqueante → detener la misión y reportar
8. Si falló y no es bloqueante → marcar como warning y continuar
```

### Paso 4 — Review Gates

Los review gates son verificaciones automáticas entre fases. Si un gate falla, se puede reintentar:

#### Gate: `verify-variables`
Después de `setup-radix-base`:
```
- figma_get_variables o figma_execute con getLocalVariablesAsync
- Verificar que existen colecciones de Colors, Typography, Spacing, Radius
- Si < 20 variables → FAIL: "Tokens insuficientes"
- Si >= 20 → PASS
```

#### Gate: `verify-components`
Después de `generate-industry` o `generate-library`:
```
- figma_search_components → contar componentes creados
- Verificar que tienen component properties (TEXT, BOOLEAN, INSTANCE_SWAP)
- Si 0 componentes → FAIL
- Si componentes sin properties → WARNING
- Si todo ok → PASS
```

#### Gate: `review-quality`
Después de cualquier skill de generación:
```
- figma_capture_screenshot → verificar visualmente
- figma_execute → escanear por:
  - Colores hardcodeados (sin boundVariables)
  - Touch targets < 44px
  - Frames colapsados (height < 10px)
  - Capas con nombres genéricos (Frame N, Rectangle N)
- Clasificar issues por severidad:
  - CRITICAL (bloquea): frames colapsados, 0 tokens
  - WARNING (no bloquea): nombres genéricos, hardcoded colors
  - INFO: sugerencias de mejora
- Si 0 critical → PASS (con warnings si hay)
- Si >0 critical → RETRY (max 2 reintentos)
  1. Regenerar los elementos con issues críticos
  2. Volver a verificar
  3. Si falla de nuevo → FAIL con reporte detallado
```

#### Gate: `ds-exists`
Verificar que hay DS antes de generar pantallas:
```
- figma_get_variables → ¿hay tokens?
- figma_search_components → ¿hay componentes?
- Si ambos vacíos → FAIL: "No hay DS. ¿Ejecutar setup-radix-base primero?"
- Si al menos uno → PASS (con warning si falta el otro)
```

### Paso 5 — Reportar resultado de la misión

Al finalizar todas las fases:

```
## 📋 Reporte de Misión

### Resumen
- Pipeline: ds-completo (fintech)
- Fases: 7/7 completadas
- Tiempo total: ~12 min
- Review gates: 3 passed, 0 failed

### Detalle por fase
| # | Skill | Tiempo | Errors | Gate | Resultado |
|---|---|---|---|---|---|
| 1 | setup-radix-base | 45s | 0 | verify-variables ✅ | ✅ |
| 2 | generate-industry | 2m | 0 | verify-components ✅ | ✅ |
| 3 | generate-library | 3m | 2 | review-quality ✅ | ✅ (1 retry) |
| 4 | generate-screen | 1.5m | 0 | review-quality ✅ | ✅ |
| 5 | create-voice | 30s | 0 | — | ✅ |
| 6 | prepare-handoff | 20s | 0 | — | ✅ |
| 7 | generate-showcase | 15s | 0 | — | ✅ |

### Issues pendientes (warnings)
- 2 capas con nombres genéricos en la pantalla de login
- 1 input sin estado de error

### Métricas para Nen
Guardadas en pluginData del frame raíz para análisis posterior.
```

### Paso 6 — Guardar métricas para Nen

Después de cada misión, guardar métricas en el archivo de Figma:

```javascript
// Guardar en el frame raíz de la página
const page = figma.currentPage;
const metrics = {
  mission_id: "mission_" + Date.now(),
  pipeline: "ds-completo",
  date: new Date().toISOString(),
  phases: [
    { skill: "setup-radix-base", time_ms: 45000, errors: 0, retries: 0, gate: "pass" },
    { skill: "generate-library", time_ms: 180000, errors: 2, retries: 1, gate: "pass" },
    // ...
  ],
  total_time_ms: 720000,
  total_errors: 3,
  total_retries: 1,
  gate_results: { pass: 3, fail: 0, skip: 4 }
};

page.setPluginData("nen_mission_" + Date.now(), JSON.stringify(metrics));
```

Estas métricas las consume el skill `nen-analyze`.

## Manejo de errores

### Error no-crítico (WARNING)
```
Fase continúa. Se registra el warning. Se reporta al final.
Ejemplo: 2 capas sin nombre → warning, la fase no se detiene.
```

### Error crítico con retry
```
Fase se reintenta máximo 2 veces. Cada reintento solo regenera lo que falló.
Ejemplo: Frame colapsado → regenerar el frame → verificar de nuevo.
```

### Error fatal (FAIL)
```
La fase falla definitivamente. Fases dependientes se cancelan.
Fases independientes siguen ejecutándose.
Se reporta al usuario con opciones:
- "¿Quieres que reintente esta fase?"
- "¿Quieres saltar esta fase y continuar?"
- "¿Quieres detener la misión?"
```

## Ejemplos de uso
- "Ejecuta el pipeline ds-completo para fintech"
- "Corre el plan de misión que acabamos de definir"
- "Genera las pantallas con review gates activados"
- "Haz una auditoría completa del archivo"
