# Qué vamos a construir

## El problema real

Mucha gente recibe entre 30 y 100 correos por día. La mayoría son ruido (newsletters, notificaciones, spam). Una fracción menor son urgentes y requieren acción inmediata.

**El costo del ruido**: revisar todo lo no leído consume tiempo y atención. Y los pocos correos urgentes a veces se mezclan con el ruido y no se atienden a tiempo.

## La solución agéntica

Un agente que:

1. Cada cierto tiempo, **revisa los correos nuevos**
2. Para cada uno, **decide** si es urgente, normal o spam
3. Para los urgentes, **crea una tarea automática** en tu sistema de gestión (Notion o Trello)
4. **Te notifica** que lo hizo, con un resumen

Tú ya no abres tu inbox para "revisar qué hay" — solo cuando hay algo realmente importante.

## Por qué es un buen primer agente

Cumple las 6 señales de "SÍ usa un agente" del [Capítulo 1.5](../../parte-1-fundamentos/15-cuando-no-usar.md):

| Señal | Aplica al caso |
|-------|----------------|
| Input es texto libre | ✅ Cuerpo del correo es texto sin estructura |
| Pasos cambian según contexto | ✅ Acción depende del contenido del correo |
| Hay que combinar tools | ✅ Gmail, Notion, Slack |
| Hay que interpretar | ✅ Distinguir urgente real de "marketing diciendo URGENTE" |
| Decisiones con criterio | ✅ Subjetivo qué cuenta como urgente |
| Razonamiento | ✅ El LLM debe leer y razonar sobre cada correo |

## Especificación detallada

### Input al agente

Para cada correo no leído:

- **De**: dirección del remitente
- **Asunto**: línea de asunto
- **Cuerpo**: texto completo (snippet o completo, según preferencia)
- **Fecha**: timestamp del correo

### Decisión del LLM

El agente clasifica en una de tres categorías:

#### 🔴 Urgente
- Comunicaciones de tu jefe directo
- Correos con fechas límite cercanas (próximas 24-48h)
- Solicitudes específicas que esperan tu respuesta
- Problemas técnicos críticos

#### 🟡 Normal
- Comunicaciones de trabajo regulares
- Actualizaciones que vale la pena leer pero no requieren acción inmediata
- Confirmaciones, recordatorios, seguimientos

#### ⚪ Spam / ignorar
- Newsletters
- Promociones comerciales
- Notificaciones automáticas no críticas

### Acciones según clasificación

| Clasificación | Acción |
|---------------|--------|
| Urgente | Crea tarea en Notion + notifica en Slack con detalles |
| Normal | (no hacer nada — opcionalmente, agregar a una lista de revisión) |
| Spam | (no hacer nada) |

### Frecuencia de ejecución

El workflow corre **cada 15 minutos**. Eso es suficiente para "casi tiempo real" sin ser excesivo en costos de API.

## Datos de prueba

Para probar el agente sin esperar correos reales, vas a poder ejecutar el workflow manualmente con correos existentes en tu inbox. Te explico cómo en el paso a paso.

## Variantes y extensiones

Una vez funcione la versión base, puedes extender:

- **Más categorías**: agrega "factura por pagar", "personal", "social", etc.
- **Auto-respuesta sugerida**: el LLM redacta una propuesta de respuesta (que tú apruebas)
- **Resumen diario**: al final del día, te manda un resumen de todos los correos clasificados
- **Aprendizaje**: si reclasificas manualmente algo, el agente lo "aprende" para próxima vez

Estas variantes están fuera del alcance del workshop pero son ejercicios excelentes para después.

## Lo que el agente NO va a hacer (por seguridad)

❌ **No envía correos** — eso es alto riesgo de error
❌ **No borra correos** — irreversible
❌ **No marca como leído** — confunde el flujo, mejor que tú lo decidas
❌ **No reenvía a otras personas** — riesgo de fuga de info
❌ **No descarga adjuntos** — fuera de alcance

Si más adelante quieres alguna de estas, agrégalas con cuidado y siempre con un humano-en-el-loop.

## Costos estimados

Para Claude Sonnet (lo más rentable para clasificación) y un volumen típico de 50 correos/día:

- ~50 llamadas/día × ~$0.003 por llamada = **~$0.15/día**
- **~$5 USD/mes** en costos de API

n8n.cloud free tier permite:
- 5,000 ejecuciones de workflow al mes (más que suficiente)
- 1 workflow activo simultáneamente

Si excedes el tier gratuito de n8n cloud, considera self-hostearlo (gratis sin límite).

---

**Siguiente: [Setup de n8n →](02-setup.md)**
