# generate-showcase-page

Skill para generar una página web de showcase con todos los componentes documentados, tokens, variantes y casos de uso. Produce HTML estático listo para desplegar.

## Prerequisito
- Carga figma-use antes de ejecutar este skill
- generate-documentation debe haberse ejecutado primero

## Cuando usar este skill
- Para compartir el sistema de diseño con el equipo o cliente
- Para generar referencia visual de todos los componentes
- Para documentar un proyecto terminado con su DS completo

## Parámetros
- formato: html · react · next (default: html)
- nombre_ds: nombre del sistema de diseño
- industria: travel · fintech · ecommerce · health · saas
- incluir: componentes · tokens · flujos · todo (default: todo)

## Qué genera

### Sección 1 — Tokens
- Paleta de colores con nombre, valor hex y uso semántico
- Escala tipográfica con preview de cada tamaño
- Escala de espaciado visual con representación gráfica
- Border radius con ejemplos
- Toggle light/dark si existe

### Sección 2 — Componentes
Para cada componente:
- Nombre y descripción
- Preview visual (screenshot de Figma)
- Variantes disponibles
- Propiedades
- Ejemplo de uso
- Notas de accesibilidad WAI-ARIA

### Sección 3 — Flujos
- Capturas de cada pantalla del flujo principal
- Descripción de cada paso
- Componentes usados por pantalla

## Pasos de ejecución

### Paso 1 — Recopilar info del DS
- figma_get_variables → tokens con valores
- figma_get_design_system_summary → lista de componentes
- figma_get_styles → estilos de color y tipografía
- figma_capture_screenshot → imagen de cada componente

### Paso 2 — Generar el HTML
- Crear index.html con CSS variables de los tokens de Figma
- Sidebar con navegación por sección
- Sección tokens con swatches y escalas visuales
- Sección componentes con previews y documentación
- Sección flujos con capturas de pantallas
- Toggle de modo claro/oscuro

### Paso 3 — Entregar
- Mostrar el HTML generado en la conversación
- Indicar cómo desplegarlo: GitHub Pages, Vercel, Netlify
- Sugerir guardarlo en /docs del repositorio

## Ejemplos de uso
- "Genera la página de showcase del sistema de diseño de travel"
- "Crea el sitio de documentación de todos los componentes"
- "Genera el showcase en React para subirlo a Vercel"
- "Necesito una página para mostrarle el DS al cliente"
