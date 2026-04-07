# mission-planner

Skill de pre-vuelo que analiza el pedido del usuario desde múltiples perspectivas de diseño antes de ejecutar cualquier skill de generación. Actúa como un equipo de especialistas que hace las preguntas correctas para que el brief esté completo.

## Cuando usar este skill
- Antes de cualquier tarea compleja de generación (pantallas, librerías, DS completos)
- Cuando el usuario da una instrucción vaga: "hazme una app de fintech"
- Cuando se quiere asegurar calidad desde el inicio en vez de corregir después
- Cuando el usuario no tiene claro qué necesita exactamente

## Cuando NO usar este skill
- Tareas simples y específicas: "cambia el color del botón a rojo"
- Cuando el usuario ya proporcionó un brief detallado con specs claras
- Tareas de auditoría o handoff (el diseño ya existe)

## Personas disponibles

Cada persona tiene un área de expertise y hace preguntas desde su perspectiva:

### 🔍 UX Researcher
**Enfoque:** Entender al usuario final y el contexto de uso.
**Preguntas típicas:**
- ¿Quién es el usuario? ¿Qué problema resuelve esta pantalla?
- ¿Cuántos pasos tiene el flujo? ¿Cuál es el happy path?
- ¿Hay flujos edge case? (error, vacío, sin conexión, primer uso)
- ¿Guest experience o requiere autenticación?
- ¿Mobile-first o desktop-first?

### 🎨 Visual Designer
**Enfoque:** Estilo visual, marca y sistema de diseño.
**Preguntas típicas:**
- ¿Hay brand guidelines? ¿Paleta de colores? ¿Logo?
- ¿Estilo: minimal, bold, corporate, playful?
- ¿Hay un DS existente o empezamos de cero?
- ¿Tipografía preferida? (Inter, SF Pro, custom)
- ¿Referentes visuales? (apps o sitios que te gusten)

### ♿ Accessibility Specialist
**Enfoque:** Accesibilidad, inclusión y compliance.
**Preguntas típicas:**
- ¿Nivel de contraste target? (AA mínimo, AAA ideal)
- ¿Requiere soporte de screen readers? ¿Qué plataformas? (iOS, Android, Web)
- ¿Touch targets mínimos? (44px standard, 48px Android)
- ¿Soporte de modo oscuro?
- ¿Idiomas con RTL? (árabe, hebreo)

### ⚙️ Developer Advocate
**Enfoque:** Implementación, handoff y viabilidad técnica.
**Preguntas típicas:**
- ¿Framework de destino? (React, Vue, Swift, Kotlin, Flutter)
- ¿Hay componentes existentes en código que debamos reflejar?
- ¿Se necesita exportar tokens? ¿En qué formato? (CSS, Tailwind, JSON)
- ¿Hay un design-to-code pipeline? (Code Connect, Storybook)
- ¿Quién consume el handoff? ¿Frontend, mobile, ambos?

### 📊 Product Manager
**Enfoque:** Alcance, prioridad y entrega.
**Preguntas típicas:**
- ¿Cuáles son las pantallas críticas (MVP) vs nice-to-have?
- ¿Hay deadline? ¿Cuánto detalle necesitas ahora vs después?
- ¿Este diseño reemplaza algo existente o es nuevo?
- ¿Se necesita documentación o showcase para stakeholders?

## Pasos de ejecución

### Paso 1 — Analizar el pedido

Leer el mensaje del usuario y clasificar:
- **Complejidad:** baja (1 pantalla) / media (flujo 3-5 pantallas) / alta (DS completo + flujo)
- **Claridad:** completo (tiene todas las specs) / parcial (faltan datos) / vago (solo una idea)
- **Tipo:** generación / auditoría / documentación / handoff

> Si complejidad es baja Y claridad es completa → no activar personas, ir directo al skill correspondiente.
> Si complejidad es media/alta O claridad es parcial/vaga → activar personas relevantes.

### Paso 2 — Activar personas relevantes

No todas las personas son necesarias para cada pedido. Seleccionar las que aplican:

| Tipo de pedido | Personas a activar |
|---|---|
| "Hazme una pantalla de X" | UX Researcher + Visual Designer |
| "Crea un DS para X" | Visual Designer + Developer Advocate + Product Manager |
| "Genera la librería desde mi código" | Developer Advocate + Visual Designer |
| "Necesito una app completa de X" | Todas las personas |
| "Prepara esto para los devs" | Developer Advocate + A11y Specialist |

### Paso 3 — Hacer preguntas consolidadas

**NO hacer 20 preguntas.** Consolidar en máximo 5-7 preguntas agrupadas por tema, indicando qué persona la necesita:

```
Para darte el mejor resultado, necesito entender un par de cosas:

1. 🔍 **Usuarios y flujo:** ¿Quién usa esto? ¿Cuántos pasos tiene el proceso?
   (Si no lo tienes claro, puedo sugerir un flujo estándar)

2. 🎨 **Estilo visual:** ¿Hay marca/colores definidos o empezamos con Radix base?
   ¿Algún referente visual que te guste?

3. ♿ **Accesibilidad:** ¿Necesitas specs de screen reader? ¿Modo oscuro?

4. ⚙️ **Implementación:** ¿Framework target? ¿Componentes existentes en código?

5. 📊 **Alcance:** ¿Cuáles son las pantallas prioritarias?

→ Si prefieres, puedo empezar con defaults razonables y ajustamos después.
```

**IMPORTANTE:** Siempre ofrecer la opción de empezar con defaults. No bloquear al usuario con preguntas.

### Paso 4 — Construir el brief

Con las respuestas (o los defaults), armar un brief estructurado:

```
## Mission Brief

### Contexto
- Proyecto: [nombre]
- Industria: [fintech/travel/health/etc]
- Plataforma: [mobile/desktop/ambos]
- Framework: [React/Vue/etc]

### Usuarios
- Persona primaria: [descripción]
- Flujo principal: [pasos]
- Edge cases: [error, vacío, offline]

### Diseño
- Estilo: [minimal/bold/corporate]
- Colores: [marca o Radix base]
- Tipografía: [Inter/custom]
- DS existente: [sí/no]

### Accesibilidad
- Contraste: [AA/AAA]
- Screen readers: [sí/no] [plataformas]
- Touch targets: [44px/48px]

### Entrega
- Skills a ejecutar: [lista ordenada por dependencias]
- Prioridad: [MVP screens]
- Output: [Figma + tokens + showcase]
```

### Paso 5 — Proponer el plan de misión

Basándose en el brief, proponer qué skills ejecutar en qué orden (esto lo toma `run-mission`):

```
## Plan de misión propuesto

1. setup-radix-base → Tokens base
2. generate-industry:fintech → DS de fintech
3. generate-library → Componentes del brief
4. generate-screen → Pantallas prioritarias
   └── REVIEW GATE: audit-quality
5. create-voice → Specs de screen reader
6. prepare-handoff → Preparar para devs
7. generate-showcase-page → Documentación visual

¿Apruebas este plan o quieres ajustar algo?
```

Esperar confirmación del usuario antes de pasar a `run-mission`.

## Defaults razonables (si el usuario no responde)

| Parámetro | Default |
|---|---|
| Plataforma | Mobile-first (375×812) |
| Estilo | Minimal, neutro |
| Colores | Radix base (blue primary, gray neutral) |
| Tipografía | Inter |
| Contraste | AA (4.5:1 texto, 3:1 grande) |
| Touch targets | 44px |
| Framework | React |
| Screen readers | Sí, todas las plataformas |

## Ejemplos de uso

**Usuario vago:**
```
"Necesito una app de delivery"
→ Activa todas las personas, hace 5 preguntas, ofrece defaults
```

**Usuario parcial:**
```
"Crea las pantallas de checkout para mi app React de ecommerce, estilo minimal"
→ Activa UX Researcher (¿cuántos pasos?) + Developer (¿componentes existentes?)
→ Visual Designer no pregunta (ya dijo minimal)
```

**Usuario detallado:**
```
"Genera un login mobile 375x812 con email, password, social login, usando Radix tokens, 
necesito VoiceOver specs y handoff para React"
→ No activa personas, va directo al plan de misión
```
