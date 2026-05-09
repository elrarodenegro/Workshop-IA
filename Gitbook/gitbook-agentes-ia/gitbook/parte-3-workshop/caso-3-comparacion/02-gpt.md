# Implementación con GPT-4 (OpenAI)

## El SDK de OpenAI

OpenAI usa **Function Calling** — el equivalente a Tool Use de Anthropic, pero con una estructura distinta.

## Definir tools en formato OpenAI

```python
# archivo: caso-3-comparacion/gpt_agent.py (parte 1)

import os
import json
from openai import OpenAI
from dotenv import load_dotenv
from mocks import obtener_info_cve, sugerir_remediacion, SYSTEM_PROMPT

load_dotenv("../.env")
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))


# === DEFINICIÓN DE TOOLS (formato OpenAI) ===
# Nota la estructura anidada: tools[].function.parameters

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "obtener_info_cve",
            "description": "Busca información de un CVE específico en la base de datos.",
            "parameters": {
                "type": "object",
                "properties": {
                    "cve_id": {
                        "type": "string",
                        "description": "ID del CVE en formato 'CVE-YYYY-NNNN'"
                    }
                },
                "required": ["cve_id"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "sugerir_remediacion",
            "description": "Sugiere pasos de remediación basados en el CVE y su severidad.",
            "parameters": {
                "type": "object",
                "properties": {
                    "cve_id": {
                        "type": "string",
                        "description": "ID del CVE"
                    },
                    "severidad": {
                        "type": "string",
                        "enum": ["CRÍTICA", "ALTA", "MEDIA", "BAJA"],
                        "description": "Nivel de severidad del CVE"
                    }
                },
                "required": ["cve_id", "severidad"]
            }
        }
    }
]
```

> ⚠️ **Diferencia clave con Anthropic**
>
> - Anthropic: `tools=[{"name", "description", "input_schema"}]`
> - OpenAI: `tools=[{"type": "function", "function": {"name", "description", "parameters"}}]`
>
> OpenAI envuelve cada tool en un wrapper `{type, function}`. Diseñado así para soportar futuros tipos de tools, pero hoy es solo overhead.

## El dispatcher (igual que en Claude)

```python
def ejecutar_tool(nombre: str, inputs: dict) -> str:
    if nombre == "obtener_info_cve":
        return str(obtener_info_cve(**inputs))
    elif nombre == "sugerir_remediacion":
        return str(sugerir_remediacion(**inputs))
    else:
        return f"Tool desconocida: {nombre}"
```

## El loop con OpenAI

La estructura es similar pero hay diferencias importantes en cómo se construyen los mensajes y se identifican las tool calls:

```python
# archivo: caso-3-comparacion/gpt_agent.py (parte 2)

def correr_agente_gpt(prompt_usuario: str, max_iter: int = 10):
    # OpenAI: el system_prompt es un mensaje más en la lista (no un parámetro aparte)
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": prompt_usuario}
    ]
    
    for iteracion in range(1, max_iter + 1):
        print(f"\n--- Iteración {iteracion} ---")
        
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=TOOLS,
            tool_choice="auto",  # deja que GPT decida si usar tools
            max_tokens=2048
        )
        
        message = response.choices[0].message
        finish_reason = response.choices[0].finish_reason
        
        # OpenAI usa "stop" cuando termina (equivalente a "end_turn" en Anthropic)
        if finish_reason == "stop":
            print("\n🏁 Agente terminó.")
            print(f"\nRespuesta:\n{message.content}")
            return
        
        # OpenAI usa "tool_calls" en finish_reason cuando quiere usar tools
        if finish_reason == "tool_calls":
            # Agregar el mensaje del assistant al historial
            # Nota: hay que serializar a dict, OpenAI usa objetos pydantic
            messages.append({
                "role": "assistant",
                "content": message.content,
                "tool_calls": [
                    {
                        "id": tc.id,
                        "type": tc.type,
                        "function": {
                            "name": tc.function.name,
                            "arguments": tc.function.arguments
                        }
                    }
                    for tc in message.tool_calls
                ]
            })
            
            # Procesar cada tool_call
            for tool_call in message.tool_calls:
                nombre = tool_call.function.name
                # OpenAI devuelve los argumentos como string JSON, hay que parsearlos
                inputs = json.loads(tool_call.function.arguments)
                
                print(f"🔧 [GPT-4] Llamando: {nombre}({inputs})")
                resultado = ejecutar_tool(nombre, inputs)
                
                # Devolver el resultado con role "tool"
                # Nota: OpenAI usa un mensaje POR cada tool_call (no agrupados como Anthropic)
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": resultado
                })
            
            continue
        
        print(f"⚠️  finish_reason inesperado: {finish_reason}")
        break


# === ENTRY POINT ===

if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE DE ANÁLISIS DE CVE · GPT-4")
    print("=" * 60)
    
    correr_agente_gpt("Analiza el CVE-2024-12345 y dime qué hacer.")
```

## Ejecutar

```bash
(venv) $ python gpt_agent.py
```

Si todo está bien (key configurada, créditos disponibles), verás una salida funcionalmente igual a la de Claude.

## Diferencias importantes vs Claude

### 1 · System prompt va en messages

```python
# Anthropic
client.messages.create(system=SYSTEM_PROMPT, messages=[...])

# OpenAI
client.chat.completions.create(messages=[
    {"role": "system", "content": SYSTEM_PROMPT},
    ...
])
```

### 2 · Los argumentos vienen como STRING JSON

```python
# Anthropic
block.input  # ya es un dict de Python

# OpenAI
tool_call.function.arguments  # es un string JSON
inputs = json.loads(tool_call.function.arguments)  # hay que parsearlo
```

> ⚠️ **Cuidado**
>
> Olvidar el `json.loads()` es un error muy común. El SDK de OpenAI no lo parsea por ti.

### 3 · Los tool_results van en mensajes individuales

```python
# Anthropic - todos los results en un solo mensaje user
messages.append({"role": "user", "content": [
    {"type": "tool_result", "tool_use_id": "1", "content": "..."},
    {"type": "tool_result", "tool_use_id": "2", "content": "..."}
]})

# OpenAI - cada result es un mensaje aparte con role "tool"
messages.append({"role": "tool", "tool_call_id": "1", "content": "..."})
messages.append({"role": "tool", "tool_call_id": "2", "content": "..."})
```

### 4 · finish_reason vs stop_reason

| Anthropic | OpenAI |
|-----------|--------|
| `end_turn` | `stop` |
| `tool_use` | `tool_calls` |
| `max_tokens` | `length` |

Mismos conceptos, distintos nombres. Causa confusión cuando saltas entre los dos SDKs.

### 5 · Hay que serializar los tool_calls al guardarlos

Esto es subtle. OpenAI devuelve objetos de tipo `ChatCompletionMessageToolCall` (pydantic). Cuando los agregas a `messages`, técnicamente puedes pasar el objeto directo, pero **es más seguro convertir a dict** para evitar problemas en serialización futura.

## Lo que destaca de OpenAI

### Lo bueno

✅ **El ecosistema más maduro**: muchas librerías y wrappers asumen formato OpenAI
✅ **`tool_choice="auto"|"none"|"required"`**: control fino sobre cuándo usar tools
✅ **Soporte para JSON mode estricto** (force JSON output)
✅ **Embeddings nativos** (text-embedding-3-small, etc.)
✅ **Comunidad enorme**: cualquier problema, hay un Stack Overflow con la respuesta

### Lo menos bueno

❌ **Estructura más anidada** (tools[].function.parameters)
❌ **Argumentos como string JSON** (más propensos a errores de parseo)
❌ **Múltiples mensajes "tool"** infla el array `messages` rápido
❌ **Cambios de API más frecuentes** entre versiones del SDK

## Características técnicas

| Aspecto | Detalle |
|---------|---------|
| **Modelos populares** | GPT-4o, GPT-4o-mini, o1-preview, o1-mini |
| **Contexto máximo** | 128,000 tokens |
| **Function calling stable** | Sí, GA |
| **Streaming con tools** | Sí |
| **JSON mode** | Sí (modo estricto disponible) |
| **Costo (referencia)** | GPT-4o $2.50/$10 per MTok in/out |

> 📌 **Nota**
>
> OpenAI también ofrece **Assistants API**, una abstracción de más alto nivel sobre function calling con manejo de threads y archivos. Para empezar, la Chat Completions API (la que usamos arriba) es lo correcto. Cuando necesites más, Assistants es una opción.

## Cuándo elegir GPT-4

✅ **Si ya tu equipo usa el ecosistema OpenAI** (no rompas lo que funciona)
✅ **Si necesitas embeddings** además de chat
✅ **Si necesitas modelos de reasoning** (o1)
✅ **Para muchas integraciones third-party** que asumen formato OpenAI

❌ Quizá no si:
- Necesitas razonamiento más profundo en tareas largas (Claude suele ganar ahí)
- Buscas el contexto más grande disponible (Gemini 2M tokens, Claude 200k)
- Tu prioridad es costo bajo (Gemini Flash o Claude Haiku son más baratos)

---

**Siguiente: [Implementación con Gemini →](03-gemini.md)**
