# Caso 1 · Agente personal en n8n

> *El agente sin código.*

## Qué vamos a construir

Un agente que automáticamente:

1. Lee correos no leídos en tu Gmail
2. Los clasifica según urgencia (alta, normal, spam)
3. Si son urgentes, crea una tarea en Notion (o Trello)
4. Te avisa por Slack que hizo el trabajo

**Sin escribir una sola línea de código.**

## Por qué empezamos aquí

Hay una idea común de que "low-code es de juguete". **Falso.** Para muchos casos reales, low-code resuelve en 30 minutos lo que en código tomaría días.

Empezamos con n8n porque:

✅ **No necesitas saber Python** — perfecto para audiencia no técnica
✅ **Ves el agente "funcionar" rápido** — el feedback visual ayuda a entender el flujo
✅ **La lógica es exactamente la misma** que en código — solo cambia la sintaxis
✅ **Es 100% real** — gente usa n8n en producción todos los días

## Quién es n8n

[n8n](https://n8n.io) es una herramienta de automatización tipo "Zapier open source", con un nodo de **AI Agent** que te permite construir agentes de IA visualmente.

Tienes dos formas de usarla:
- **n8n.cloud** — versión hosted, tier gratuito disponible
- **n8n self-hosted** — gratis sin límites si lo corres en tu máquina o servidor

Para el workshop usamos n8n.cloud porque es lo más rápido para empezar.

## Tiempo estimado

| Sección | Tiempo |
|---------|--------|
| Setup de n8n | 10 min |
| Construcción paso a paso | 25 min |
| Pruebas y troubleshooting | 10 min |
| **Total** | **~45 min** |

## Lo que vas a aprender

Al terminar este caso podrás:

- ✅ Crear un workflow en n8n desde cero
- ✅ Conectar Gmail, Slack y Notion como nodos
- ✅ Usar el nodo **AI Agent** con un LLM
- ✅ Pasar datos entre nodos
- ✅ Probar y debuggear un workflow
- ✅ Activar el workflow para que corra automáticamente

Y, lo más importante: **vas a ver con tus propios ojos** los conceptos del Bloque 1 funcionando — herramientas, tool use, decisión del LLM, etc.

## Lo que NO vamos a hacer aquí

❌ No vamos a auto-responder los correos (peligroso si te equivocas)
❌ No vamos a borrar correos (irreversible)
❌ No vamos a manejar adjuntos (queda fuera de alcance)
❌ No vamos a escalar a miles de correos (es un caso personal, ~10-50 correos/día)

Esos casos son posibles con n8n, pero no son apropiados para un primer agente.

## Diagrama del flujo

```
┌──────────────┐
│ Gmail        │  Trigger: cada 15 min, busca correos no leídos
│ Trigger      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ AI Agent     │  Clasifica: ¿urgente, normal, spam?
│ (Classifier) │  Decide qué hacer
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Switch       │  Ramifica según la clasificación
│ (by urgency) │
└──┬───────┬───┘
   │       │
   ▼       ▼
┌───────┐ ┌──────────┐
│Notion │ │ Slack    │  Si urgente: crea tarea + avisa
│Task   │ │ Notify   │
└───────┘ └──────────┘
```

## Estructura de este caso

1. **[Qué vamos a construir](01-objetivo.md)** — más detalle del objetivo y por qué (ya lo leíste arriba, pero el archivo expande)
2. **[Setup de n8n](02-setup.md)** — crear cuenta, abrir el editor
3. **[Construcción paso a paso](03-paso-a-paso.md)** — cada nodo, con screenshots conceptuales
4. **[Troubleshooting](04-troubleshooting.md)** — qué hacer cuando algo falla

> 📌 **Nota sobre screenshots**
>
> Este libro está en formato Markdown puro. Donde dice "ve la captura aquí", reemplaza mentalmente con la captura que tomes en tu pantalla — la UI de n8n es muy visual y se entiende sola siguiendo las instrucciones.

---

**Siguiente: [Qué vamos a construir →](01-objetivo.md)**
