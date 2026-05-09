# 1.1 · LLM vs Chatbot vs Agente

## Tres conceptos que la gente confunde

Si vas a una conferencia de tecnología y mezclas estas tres palabras, los técnicos van a hacer cara rara. Es como decir "motor", "auto" y "carrera" como si fueran lo mismo.

Vamos a ordenarlas.

## El LLM · "el cerebro"

**LLM** significa **Large Language Model** — modelo de lenguaje grande.

Es un programa entrenado con cantidades masivas de texto que ha aprendido a predecir qué palabra viene después en una frase. Eso suena simple, pero esa capacidad de predicción, escalada a billones de parámetros, produce algo que parece comprensión.

### Lo que un LLM hace

- Recibe texto (un "prompt")
- Devuelve texto (una "respuesta")

Eso es todo. **Un LLM es una función pura**: input texto → output texto.

### Lo que un LLM NO hace

- ❌ No tiene memoria entre llamadas. Cada llamada es independiente.
- ❌ No puede ejecutar acciones (no abre tu Gmail, no hace búsquedas, no envía correos).
- ❌ No tiene acceso a información en tiempo real (a menos que se la des explícitamente).
- ❌ No "sabe" cosas — produce texto que estadísticamente parece correcto.

### Ejemplos

GPT-4, Claude, Gemini, Llama, Mistral.

Cuando llamas a la API de Anthropic con `client.messages.create(...)`, le estás hablando directamente a un LLM. Sin envoltorios.

## El Chatbot · "el mesero"

Un **chatbot** es un LLM **+ una interfaz conversacional + memoria del chat actual**.

### Lo que un chatbot hace

- Mantiene un historial de mensajes en una conversación
- Te muestra una interfaz amigable (ventana de chat)
- Responde con contexto de mensajes previos en la misma sesión

### Lo que un chatbot sigue sin hacer

- ❌ No actúa por ti fuera de responder texto
- ❌ Su memoria es típicamente solo de la conversación actual
- ❌ Si le dices "envía este correo", no lo envía — solo te dice cómo enviarlo

### Ejemplos

ChatGPT (la app web), claude.ai, Gemini (la app), Perplexity.

> 💡 **Tip**
>
> Cuando tu mamá dice "hablé con la IA", probablemente se refiere a un chatbot. Cuando un técnico habla de "el modelo", suele referirse al LLM detrás. La distinción importa cuando estás construyendo.

## El Agente · "el asistente"

Un **agente** es un LLM **+ herramientas + memoria + un loop de decisión**.

La diferencia clave: **un agente decide qué hacer y lo hace por ti**.

### Lo que un agente hace

- Recibe un objetivo (no necesariamente el paso a paso)
- Decide qué herramienta usar para avanzar
- Ejecuta la herramienta
- Observa el resultado
- Decide el siguiente paso
- Repite hasta cumplir el objetivo

### El cambio crítico

Mientras un chatbot responde lo que le preguntas, **un agente actúa para resolver lo que le pides**.

Si le dices a un chatbot: *"¿Cuál es el clima en Bogotá?"*, te puede responder con su conocimiento estancado en su fecha de entrenamiento.

Si le dices a un agente: *"¿Cuál es el clima en Bogotá?"*, va a:
1. Decidir que necesita una herramienta meteorológica
2. Llamar a una API real de clima
3. Recibir la respuesta actual
4. Reportarte el dato

### Ejemplos

- **Cursor** y **Claude Code**: agentes que escriben código en tu repo
- **Devin**: agente autónomo de desarrollo
- **n8n con AI Agent**: agente low-code que orquesta flujos
- **AutoGPT, BabyAGI**: experimentos tempranos de agentes autónomos

## Tabla resumen

| Aspecto | LLM | Chatbot | Agente |
|---------|-----|---------|--------|
| ¿Qué es? | Función texto→texto | LLM + UI + chat history | LLM + tools + memory + loop |
| ¿Mantiene contexto? | No | Sí (sesión) | Sí (sesión y/o largo plazo) |
| ¿Ejecuta acciones? | No | No | Sí |
| ¿Decide pasos? | No | No | Sí |
| Ejemplo | API de Claude | claude.ai | Cursor, n8n+AI |

## El error mental que vas a ver mucho

Mucha gente con buena intención dice cosas como:

> "Hicimos un agente con ChatGPT que automatiza nuestros tickets."

A veces es cierto. A veces simplemente conectaron ChatGPT a un Zapier que clasifica tickets — eso es un **chatbot integrado**, no necesariamente un agente.

**La pregunta filtro**: ¿el sistema decide en runtime qué hacer, o sigue un guión predefinido?

- Sigue guión → automatización + LLM (chatbot integrado)
- Decide en runtime → agente

Ambos son válidos. Pero llamarles igual confunde a todo el mundo, incluido tú mismo cuando intentes diseñarlos.

## Por qué importa la distinción

Cuando alguien te pide *"hagamos un agente para X"*, lo primero que debes preguntarte es: **¿realmente necesitan un agente, o un chatbot integrado les sirve?**

Un chatbot bien diseñado:
- Es más predecible
- Es más barato (menos llamadas al LLM)
- Es más fácil de debuggear
- Es más rápido

Un agente:
- Maneja casos no previstos
- Toma decisiones contextuales
- Pero es más caro, lento y difícil de probar

> 📌 **Nota**
>
> En la **Parte 1.5 · Cuándo NO usar agentes** profundizamos en esta decisión. Es uno de los capítulos más importantes del libro.

## Recapitulando

- **LLM** = el cerebro (función texto→texto)
- **Chatbot** = LLM + UI + memoria de chat (mesero que conversa)
- **Agente** = LLM + tools + memoria + loop de decisión (asistente que actúa)

> 🎯 **Reto**
>
> Antes de pasar al siguiente capítulo, identifica tres herramientas que uses (o conozcas) y clasifícalas: ¿son LLMs puros, chatbots, o agentes? Justifica tu respuesta usando los criterios de arriba.

---

**Siguiente: [1.2 · El loop agéntico →](12-loop-agentico.md)**
