# 2.1 · Nivel 1 · Tool Use simple

## El punto de partida

Casi todos los agentes que vas a construir empiezan aquí. Es la arquitectura más simple que cuenta como "agente":

**Un LLM con acceso a 1-3 herramientas, sin memoria persistente, en un loop básico.**

Si entiendes este nivel a fondo, los siguientes son extensiones naturales.

## El diagrama

```
┌──────────┐   input   ┌─────────────┐   tool_call   ┌─────────┐
│ USUARIO  │──────────▶│   LLM       │──────────────▶│ TOOLS   │
└──────────┘           │             │               │         │
                       │  + tools=[] │◀──────────────│ - api_a │
                       └─────────────┘   result      │ - api_b │
                              │                      │ - api_c │
                              ▼                      └─────────┘
                       respuesta final
```

## Caso típico

> *"Mira el clima de mañana y agéndame una reunión con Pedro a las 9am si no llueve."*

El agente decide en tiempo real:

1. Necesito información del clima → llama `obtener_clima(fecha="mañana")`
2. Recibo la respuesta → "soleado, 0% lluvia"
3. Necesito agendar → llama `crear_evento(con="Pedro", hora="9am")`
4. Confirmación recibida → respondo al usuario

**Sin memoria entre conversaciones.** Cada nueva sesión empieza limpia.

## Cómo se ve en código (Anthropic Tool Use)

Una versión mínima:

```python
import os
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()
client = Anthropic()

# 1 · Define las herramientas
tools = [
    {
        "name": "obtener_clima",
        "description": "Obtiene el pronóstico meteorológico de una ciudad y fecha.",
        "input_schema": {
            "type": "object",
            "properties": {
                "ciudad": {"type": "string"},
                "fecha": {"type": "string"}
            },
            "required": ["ciudad", "fecha"]
        }
    },
    {
        "name": "crear_evento",
        "description": "Crea un evento de calendario.",
        "input_schema": {
            "type": "object",
            "properties": {
                "titulo": {"type": "string"},
                "fecha_hora": {"type": "string"}
            },
            "required": ["titulo", "fecha_hora"]
        }
    }
]

# 2 · Llama al LLM con las tools
response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=1024,
    tools=tools,
    messages=[
        {"role": "user", "content": "Mira el clima mañana y agendame con Pedro 9am si no llueve"}
    ]
)

# 3 · El LLM responde con un tool_use, no con texto
print(response.stop_reason)  # "tool_use"
print(response.content)       # contiene el tool_use bloque
```

## El loop completo (lo que vas a implementar en el Caso 2)

El código de arriba es solo la primera iteración. Para que sea un agente real, necesitas el **loop**:

```python
# Loop de Tool Use (versión simplificada)
messages = [{"role": "user", "content": user_input}]

while True:
    response = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )
    
    # Si no quiere usar más tools, termina
    if response.stop_reason == "end_turn":
        break
    
    # Agrega la respuesta al historial
    messages.append({"role": "assistant", "content": response.content})
    
    # Procesa los tool_use blocks
    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            # Aquí TÚ ejecutas la función real
            result = ejecutar_tool(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result
            })
    
    # Devuélvele los resultados al LLM
    messages.append({"role": "user", "content": tool_results})
```

> 📌 **Nota**
>
> En el **Caso 2** del workshop construyes este loop completo paso a paso, con explicación detallada de cada parte. No te preocupes si ahora no se entiende todo.

## Características del Nivel 1

| Aspecto | Característica |
|---------|----------------|
| **Herramientas** | 1 a 3 típicamente |
| **Memoria** | Solo de la conversación actual |
| **Iteraciones promedio** | 1-5 vueltas del loop |
| **Costo por ejecución** | Bajo (~$0.01-0.10) |
| **Complejidad de código** | Baja (~50-100 líneas) |
| **Casos de uso ideales** | Asistentes personales, automatizaciones simples, demos |

## Cuándo el Nivel 1 es suficiente

✅ **Sí, te basta con Nivel 1 cuando:**

- Las tareas son cortas (terminan en pocos pasos)
- No necesitas recordar conversaciones previas
- Tienes pocas herramientas
- No necesitas integraciones complejas

✅ **Ejemplos reales que viven en Nivel 1:**

- Bot de Slack que crea tareas en Notion según comandos
- Asistente de email que clasifica y responde
- Comando de terminal con IA que ejecuta acciones en tu sistema
- Demos y prototipos de cualquier cosa

## Cuándo el Nivel 1 se queda corto

❌ **Necesitas subir de nivel cuando:**

- El agente debe recordar conversaciones de hace días o semanas
- Hay que consultar grandes volúmenes de documentación
- El sistema debe escalar a miles de usuarios concurrentes
- Hay tareas largas que requieren múltiples especialidades

Cada uno de esos te lleva al **Nivel 2, 3 o 4**.

## Lo que enseña este nivel

Si dominas el Nivel 1 entiendes:

- Cómo se definen herramientas con JSON Schema
- Cómo el LLM elige cuál tool llamar
- Cómo manejar el `tool_use` y `tool_result` en mensajes
- Cómo cerrar el loop cuando el LLM decide que terminó

Esos cuatro conocimientos son **transferibles a cualquier nivel superior**. Los Niveles 2-4 agregan capas, pero el corazón es el mismo Tool Use.

> 💡 **Tip**
>
> Resiste la tentación de saltar directo a multi-agente o frameworks complejos antes de dominar el Nivel 1. He visto a equipos perder semanas en LangGraph porque no entendían qué pasaba en una llamada simple de Tool Use. Domina lo simple primero.

## Recapitulando

- Nivel 1 = LLM + 1-3 tools + loop básico, sin memoria persistente
- Es el punto de partida de prácticamente todos los agentes en producción
- Suficiente para muchos casos reales (no asumas que necesitas más)
- Lo construimos paso a paso en el **Caso 2** del workshop

---

**Siguiente: [2.2 · Nivel 2 · RAG + memoria →](22-nivel-2-rag.md)**
