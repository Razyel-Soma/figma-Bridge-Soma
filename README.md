# Figma Murdoc

Fork del [Figma Console MCP](https://github.com/southleft/figma-console-mcp) extendido con un sistema de **Skills** para equipos de diseño. Conecta Claude con Figma Desktop para generar pantallas, documentar componentes, preparar handoffs y auditar sistemas de diseño — todo desde lenguaje natural.

## Qué añade este fork

- **Sistema de Skills** — instrucciones en markdown que definen flujos de trabajo completos para diseñadores
- **Loader dinámico** — los skills se leen en tiempo real sin recompilar ni reiniciar el servidor
- **Instrucciones de arranque** — el servidor carga `figma-murdoc.md` al iniciar
- **11 skills listos para usar** — cubre los workflows más comunes de un equipo de diseño
- **Integración con Radix UI** — sistema de diseño base accesible con skills por industria

## Skills disponibles

### Base
| Skill | Qué hace |
|---|---|
| `figma-use` | Skill base obligatorio. Define cómo trabajar en el canvas de Figma |

### Generación de pantallas
| Skill | Qué hace |
|---|---|
| `generate-screen` | Genera cualquier pantalla: onboarding, login, dashboard, empty states o custom |
| `generate-onboarding` | Flujo de onboarding mobile de 4 pantallas con sistema de diseño |

### Documentación
| Skill | Qué hace |
|---|---|
| `generate-documentation` | Documenta componentes listos para Notion, Confluence o Storybook |
| `generate-showcase-page` | Genera una página web con componentes documentados, tokens y flujos |

### Handoff a desarrollo
| Skill | Qué hace |
|---|---|
| `prepare-handoff` | Prepara diseños para desarrollo: anotaciones, nomenclatura y "Ready for dev" |
| `sync-tokens` | Exporta variables de Figma a CSS, Tailwind, JSON o Sass |

### Auditoría y calidad
| Skill | Qué hace |
|---|---|
| `audit-quality` | Detecta drift del DS, problemas WCAG y valores hardcodeados |

### Sistema de diseño con Radix UI
| Skill | Qué hace |
|---|---|
| `setup-radix-base` | Importa tokens de Radix UI como variables en Figma |
| `generate-industry` | Genera DS completo por industria: travel, fintech, ecommerce, health, saas |
| `apply-design-system` | Conecta diseños existentes al sistema de diseño activo |

## Flujo Radix UI → Figma → Showcase

```
1. setup-radix-base
   └── Crea variables en Figma: Colors, Typography, Spacing, Radius, Shadow

2. generate-industry  (travel | fintech | ecommerce | health | saas)
   ├── Extiende los tokens base con paleta de la industria
   ├── Genera los componentes específicos del sector
   └── Arma el flujo principal de pantallas

3. generate-documentation
   └── Documenta cada componente con variantes, props y notas ARIA

4. generate-showcase-page
   └── Produce index.html con tokens, componentes y flujos listos para desplegar
```

**Ejemplo completo:**
```
"Configura Radix base"                        → setup-radix-base
"Genera el DS de travel para Wanderly"        → generate-industry (travel)
"Documenta los componentes de Wanderly"       → generate-documentation
"Genera el showcase para mostrarle al cliente" → generate-showcase-page
```

## Industrias disponibles

| Industria | Componentes | Flujo principal |
|---|---|---|
| `travel` | DateRangePicker, FlightCard, SeatMap, SearchBar | Búsqueda → Resultados → Detalle → Checkout |
| `fintech` | BalanceCard, TransactionRow, TransferForm, PinInput | Inicio → Transferir → Confirmar → PIN → Éxito |
| `ecommerce` | ProductCard, CartItem, SizeSelector, FilterSidebar | Catálogo → Detalle → Carrito → Checkout |
| `health` | DoctorCard, AppointmentSlot, HealthMetric, VitalChart | Buscar médico → Disponibilidad → Confirmar cita |
| `saas` | StatCard, DataTable, Sidebar, CommandPalette | Dashboard → Detalle → Configuración |

## Requisitos

- [Figma Desktop](https://www.figma.com/downloads/) — obligatorio, no funciona con Figma en navegador
- Node.js 18 o superior
- Claude Desktop con soporte MCP
- Plugin **Figma Desktop Bridge** instalado en Figma

## Instalación

### 1. Clonar el repo
```bash
git clone https://github.com/Razyel-Soma/figma-Murdoc.git
cd figma-Bridge-Soma
npm install
```

### 2. Instalar el plugin en Figma Desktop

1. Abre Figma Desktop
2. Ve a `Plugins → Development → Import plugin from manifest...`
3. Selecciona el archivo:
```
figma-Bridge-Soma/figma-desktop-bridge/manifest.json
```
4. El plugin aparecerá como **"Figma Desktop Bridge"**

### 3. Compilar el servidor
```bash
npm run build:local
```

### 4. Conectar con Claude Desktop

Edita `~/Library/Application Support/Claude/claude_desktop_config.json` y añade:
```json
{
  "mcpServers": {
    "figma-murdoc": {
      "command": "node",
      "args": ["/ruta/a/figma-Bridge-Soma/dist/local.js"]
    }
  }
}
```

Reemplaza `/ruta/a/` con la ruta real donde clonaste el repo. Reinicia Claude Desktop.

### 5. Verificar la instalación

En Claude Desktop escribe:
```
list_skills
```

Deberías ver los 11 skills disponibles.

## Uso

### Activar el plugin en Figma

Antes de usar cualquier skill, abre el plugin en Figma Desktop:
`Plugins → Development → Figma Desktop Bridge`

Espera a ver **"✓ Desktop Bridge active"** en el panel del plugin.

### Usar un skill

Describe lo que quieres en lenguaje natural:
```
Configura el sistema de diseño base con Radix
Genera los componentes de fintech para el cliente BancoX
Documenta el componente Button de mi sistema de diseño
Prepara el flujo de checkout para entregarlo a los devs
```

### Invocar un skill manualmente
```
use_skill generate-industry
```

## Añadir skills propios

Los skills son archivos `.md` en `/skills/`. No necesitas recompilar — se leen en tiempo real.
```bash
cat > skills/mi-skill.md << 'SKILL'
# mi-skill

Descripción de qué hace.

## Prerequisito
Carga figma-use antes de ejecutar este skill.

## Cuando usar este skill
- Situación A

## Pasos de ejecución
### Paso 1 — ...
SKILL
```

## Estructura del proyecto
```
figma-Bridge-Soma/
├── skills/                        ← Skills en markdown (sin compilar)
│   ├── figma-use.md
│   ├── generate-screen.md
│   ├── generate-onboarding.md
│   ├── generate-documentation.md
│   ├── generate-showcase-page.md
│   ├── prepare-handoff.md
│   ├── sync-tokens.md
│   ├── audit-quality.md
│   ├── apply-design-system.md
│   ├── setup-radix-base.md
│   └── generate-industry.md
├── src/
│   ├── skills/loader.ts           ← Lee los .md en tiempo real
│   ├── tools/skills.ts            ← Tools list_skills y use_skill
│   └── local.ts                   ← Servidor MCP (modificado)
├── figma-desktop-bridge/          ← Plugin de Figma (sin cambios)
├── figma-murdoc.md                ← Instrucciones de arranque
└── dist/local.js                  ← Entry point compilado
```

## Documentación adicional

- [FAQ para el equipo](docs/FAQ.md) — preguntas frecuentes, diferencias con el MCP oficial y guía de personalización

## Créditos

Basado en [figma-console-mcp](https://github.com/southleft/figma-console-mcp) por Southleft. Licencia MIT.

Desarrollado y extendido por Rz Inc.
