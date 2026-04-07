# nen-analyze

Sistema de auto-mejora para Murdoc. Analiza métricas acumuladas de misiones ejecutadas, detecta patrones de error, identifica cuellos de botella, y genera recomendaciones para mejorar los skills.

Inspirado en el sistema Nen de Hunter x Hunter — capacidades que se fortalecen con la práctica.

## Cuando usar este skill
- Después de varias misiones ejecutadas (recomendado: cada 5-10 misiones)
- Cuando un skill falla repetidamente y no se sabe por qué
- Para optimizar tiempos de ejecución
- Para descubrir errores de API recurrentes que deberían documentarse
- Como parte de mantenimiento periódico del sistema

## Fuentes de datos

Las métricas se almacenan en `pluginData` de las páginas del archivo de Figma:

```javascript
// Cada misión guarda esto (lo hace run-mission)
page.setPluginData("nen_mission_TIMESTAMP", JSON.stringify({
  mission_id: "mission_1712444800000",
  pipeline: "ds-completo",
  date: "2026-04-06T19:00:00Z",
  phases: [
    {
      skill: "setup-radix-base",
      time_ms: 45000,
      errors: 0,
      retries: 0,
      gate: "pass",
      error_types: []
    },
    {
      skill: "generate-library",
      time_ms: 210000,
      errors: 3,
      retries: 2,
      gate: "pass",
      error_types: ["layoutSizingVertical:AUTO", "blendMode:missing", "collapsed-frame"]
    }
  ],
  total_time_ms: 720000,
  total_errors: 3,
  total_retries: 2,
  gate_results: { pass: 3, fail: 0, skip: 4 }
}));
```

## Pasos de ejecución

### Paso 1 — Recolectar métricas

```javascript
// ✅ timeout: 15000 — leer todas las métricas de Nen del archivo
const pages = figma.root.children;
const missions = [];

for (const page of pages) {
  const keys = page.getPluginDataKeys();
  for (const key of keys) {
    if (key.startsWith('nen_mission_')) {
      try {
        const data = JSON.parse(page.getPluginData(key));
        missions.push(data);
      } catch(e) {}
    }
  }
}

return {
  total_missions: missions.length,
  missions: missions.sort((a,b) => new Date(b.date) - new Date(a.date))
};
```

Si no hay métricas, informar al usuario que necesita ejecutar misiones con `run-mission` primero.

### Paso 2 — Análisis de errores recurrentes

Agrupar errores por tipo y contar frecuencia:

```
## 🔍 Errores recurrentes

| Error | Frecuencia | Skills afectados | ¿Documentado en figma-use? |
|---|---|---|---|
| layoutSizingVertical:"AUTO" | 12 veces en 5 misiones | generate-library, generate-screen | ✅ Sí |
| blendMode:missing | 8 veces en 4 misiones | generate-library, generate-screen | ✅ Sí |
| collapsed-frame | 6 veces en 3 misiones | generate-library | ✅ Sí |
| font-not-available | 3 veces en 2 misiones | generate-screen | ❌ No documentado |

### Hallazgo:
"font-not-available" apareció 3 veces y NO está documentado en figma-use.md.
→ Recomendación: Agregar regla de verificación de fuentes disponibles
  antes de crear textos. Usar figma.listAvailableFontsAsync() como check.
```

### Paso 3 — Análisis de tiempos

Identificar skills lentos y cuellos de botella:

```
## ⏱️ Tiempos de ejecución

| Skill | Promedio | Mediana | Máximo | % del tiempo total |
|---|---|---|---|---|
| generate-library | 3.2 min | 3.0 min | 5.1 min | 42% |
| generate-screen | 1.8 min | 1.5 min | 3.0 min | 24% |
| setup-radix-base | 0.8 min | 0.7 min | 1.2 min | 10% |
| create-voice | 0.5 min | 0.5 min | 0.6 min | 7% |
| audit-quality | 0.5 min | 0.4 min | 0.8 min | 7% |
| prepare-handoff | 0.3 min | 0.3 min | 0.5 min | 4% |
| generate-showcase | 0.3 min | 0.2 min | 0.4 min | 4% |

### Hallazgo:
generate-library consume 42% del tiempo total y tiene la mayor tasa de reintentos (60%).
→ Recomendación: Dividir la creación de componentes en llamadas más pequeñas.
  Crear 1 componente por llamada en vez de intentar crear 2-3.
```

### Paso 4 — Análisis de review gates

Evaluar efectividad de los gates:

```
## 🚦 Review Gates

| Gate | Ejecuciones | Pass | Fail | Retry exitoso | Retry fallido |
|---|---|---|---|---|---|
| verify-variables | 5 | 5 | 0 | — | — |
| verify-components | 5 | 4 | 1 | 1 | 0 |
| review-quality | 10 | 7 | 3 | 2 | 1 |
| a11y | 3 | 3 | 0 | — | — |

### Hallazgo:
review-quality falla 30% de las veces, pero el retry tiene 67% de éxito.
Los fallos restantes son por frames colapsados que no se arreglan con retry.
→ Recomendación: Agregar verificación pre-emptiva de altura mínima
  ANTES de la creación, no solo en el gate post-creación.
```

### Paso 5 — Análisis de pipelines

Comparar rendimiento entre pipelines:

```
## 📊 Pipelines

| Pipeline | Ejecuciones | Tasa de éxito | Tiempo promedio | Skill más problemático |
|---|---|---|---|---|
| ds-completo | 3 | 100% | 12 min | generate-library |
| pantallas | 5 | 80% | 4 min | generate-screen |
| libreria | 2 | 100% | 6 min | generate-library |
| audit-completo | 4 | 100% | 2 min | — |

### Hallazgo:
pipeline "pantallas" tiene 80% de éxito. El 20% que falla es por
falta de DS (gate ds-exists falla). Los usuarios olvidan crear tokens primero.
→ Recomendación: Si ds-exists falla, ofrecer ejecutar setup-radix-base
  automáticamente en vez de detener la misión.
```

### Paso 6 — Generar recomendaciones priorizadas

```
## 🎯 Recomendaciones de Nen (priorizadas por impacto)

### P0 — Impacto alto, esfuerzo bajo
1. **Documentar "font-not-available"** en figma-use.md
   - Agregar check de `listAvailableFontsAsync()` antes de crear textos
   - Impacto: evita 3+ errores por misión
   - Esfuerzo: 5 líneas en el skill

2. **Auto-crear DS si falta** en pipeline "pantallas"
   - Si ds-exists falla, ofrecer setup-radix-base automático
   - Impacto: sube tasa de éxito de 80% a ~95%
   - Esfuerzo: 3 líneas en run-mission

### P1 — Impacto alto, esfuerzo medio
3. **Dividir generate-library** en sub-fases
   - 1 componente por llamada reduce errores de timeout
   - Impacto: reduce tiempo de 3.2min a ~2min estimado
   - Esfuerzo: reestructurar el skill

### P2 — Impacto medio, esfuerzo bajo
4. **Pre-check de altura mínima** antes de crear frames
   - Verificar que resize() se llama con min 44px
   - Impacto: elimina 100% de "collapsed-frame" errors
   - Esfuerzo: agregar snippet en figma-use
```

### Paso 7 — Guardar reporte de Nen

Guardar el análisis como pluginData para tracking histórico:

```javascript
const page = figma.currentPage;
page.setPluginData("nen_report_" + Date.now(), JSON.stringify({
  date: new Date().toISOString(),
  missions_analyzed: missions.length,
  top_errors: [...],
  recommendations: [...],
  metrics_summary: {...}
}));
```

## Visualización de tendencias

Si hay suficientes datos (>5 misiones), mostrar tendencias:

```
## 📈 Tendencias

Errores por misión:
  Misión 1: ████████ 8 errores
  Misión 2: ██████ 6 errores
  Misión 3: ████ 4 errores  ← fix de effects
  Misión 4: ███ 3 errores
  Misión 5: ██ 2 errores    ← fix de layout sizing

Tendencia: ↓ 75% reducción de errores en 5 misiones.
Los fixes de figma-use.md están funcionando.

Tiempo promedio por misión:
  Misión 1: ████████████████ 16 min
  Misión 2: ██████████████ 14 min
  Misión 3: ████████████ 12 min
  Misión 4: ██████████ 10 min
  Misión 5: █████████ 9 min

Tendencia: ↓ 44% reducción de tiempo. Menos reintentos = más rápido.
```

## Ciclo de mejora continua

```
  run-mission ejecuta skills
         ↓
  Métricas se guardan (pluginData)
         ↓
  nen-analyze detecta patrones
         ↓
  Genera recomendaciones
         ↓
  El equipo actualiza los skills
         ↓
  Siguiente misión tiene menos errores
         ↓
  nen-analyze confirma la mejora
         ↓ (ciclo)
```

## Ejemplos de uso
- "Analiza las métricas de las últimas misiones"
- "¿Cuáles son los errores más frecuentes de Murdoc?"
- "¿Qué skills son los más lentos?"
- "Dame recomendaciones para mejorar los skills"
- "¿Están funcionando los últimos fixes?"
- "Muéstrame la tendencia de errores"
