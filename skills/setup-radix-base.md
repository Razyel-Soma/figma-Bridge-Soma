# setup-radix-base

Skill para importar los tokens de Radix UI como variables en Figma. Establece la base del sistema de diseño usando Radix Colors, tipografía y espaciados.

## Prerequisito
Carga figma-use antes de ejecutar este skill.

## Cuando usar este skill
- Al iniciar un proyecto nuevo que usará Radix UI como base
- Como paso previo a cualquier skill de industria (travel, fintech, ecommerce)

## Qué instala

Variables creadas en Figma:
- Radix / Colors → Light y Dark: background, surface, border, text, accent, success, warning, error
- Radix / Typography → family, sizes (xs→3xl), weights, line-heights
- Radix / Spacing → escala 4px→64px
- Radix / Radius → none, sm, md, lg, xl, full
- Radix / Shadow → sm, md, lg

## Pasos de ejecución

### Paso 1 — Verificar archivo
- figma_get_status → confirmar conexión
- figma_get_variables → verificar que no existe ya "Radix Base"
- Si existe, preguntar al usuario si quiere sobreescribir

### Paso 2 — Crear colecciones
- figma_setup_design_tokens "Radix / Colors" modos Light y Dark con tokens semánticos
- figma_setup_design_tokens "Radix / Typography" con escala completa
- figma_setup_design_tokens "Radix / Spacing" con escala 4-64px
- figma_setup_design_tokens "Radix / Radius" con todos los valores
- figma_setup_design_tokens "Radix / Shadow" sm md lg

### Paso 3 — Reportar
- Mostrar resumen de variables creadas
- Sugerir siguiente paso: generate-industry o generate-screen

## Ejemplos de uso
- "Configura el sistema de diseño base con Radix en este archivo"
- "Instala los tokens de Radix antes de empezar el proyecto"
