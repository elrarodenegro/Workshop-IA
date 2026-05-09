# Implementación con Gemini (Google)

## El SDK de Google

Google llama a esto **Function Calling** también. La librería oficial es `google-generativeai`, y tiene su propia personalidad — más cercana a un patrón de "chat continuo" que al modelo de "request/response" de Anthropic y OpenAI.

## Definir tools en formato Google

```python
# archivo: caso-3-comparacion/gemini_agent.py (parte 1)

import os
import json
import google.generativeai as genai
from dotenv import load_dotenv
from mocks import obtener_info_cve, sugerir_remediacion, SYSTEM_PROMPT

load_dotenv("../.env")
genai.configure(api_key=os.getenv("GEMINI_API_KEY"))


# === DEFINICIÓN DE TOOLS (formato Google) ===

TOOLS = [
    genai.protos.Tool(
        function_declarations=[
            genai.protos.FunctionDeclaration(
                name="obtener_info_cve",
                description="Busca información de un CVE específico en la base de datos.",
                parameters=genai.protos.Schema(
                    type=genai.protos.Type.OBJECT,
                    properties={
                        "cve_id": genai.protos.Schema(
                            type=genai.protos.Type.STRING,
                            description="ID del CVE en formato 'CVE-YYYY-NNNN'"
                        )
                    },
                    required=["cve_id"]
                )
            ),
            genai.protos.FunctionDeclaration(
                name="sugerir_remediacion",
                description="Sugiere pasos de remediación basados en el CVE y su severidad.",
                parameters=genai.protos.Schema(
                    type=genai.protos.Type.OBJECT,
                    properties={
                        "cve_id": genai.protos.Schema(
                            type=genai.protos.Type.STRING,
                            description="ID del CVE"
                        ),
                        "severidad": genai.protos.Schema(
                            type=genai.protos.Type.STRING,
                            enum=["CRÍTICA", "ALTA", "MEDIA", "BAJA"],
                            description="Nivel de severidad"
                        )
                    },
                    required=["cve_id", "severidad"]
                )
            )
        ]
    )
]
```

> ⚠️ **Diferencia principal con los otros**
>
> Google usa **clases protobuf** (`genai.protos.Tool`, `Schema`, `Type.STRING`) en lugar de dicts puros. Esto es más verbose pero también más type-safe.
>
> **Hay también una forma "shortcut" más simple usando dicts** (más adelante muestro), pero la versión protobuf es la canónica.

## El dispatcher (igual)

```python
def ejecutar_tool(nombre: str, inputs: dict) -> str:
    if nombre == "obtener_info_cve":
        return str(obtener_info_cve(**inputs))
    elif nombre == "sugerir_remediacion":
        return str(sugerir_remediacion(**inputs))
    else:
        return f"Tool desconocida: {nombre}"
```

## El loop con Gemini

Aquí es donde Gemini se diferencia más. Usa un **objeto `chat`** que mantiene el historial automáticamente:

```python
# archivo: caso-3-comparacion/gemini_agent.py (parte 2)

def correr_agente_gemini(prompt_usuario: str, max_iter: int = 10):
    model = genai.GenerativeModel(
        model_name="gemini-1.5-pro",
        system_instruction=SYSTEM_PROMPT,
        tools=TOOLS
    )
    
    # Iniciar un chat (mantiene historial automáticamente)
    chat = model.start_chat(enable_automatic_function_calling=False)
    
    # Primera llamada con el prompt del usuario
    response = chat.send_message(prompt_usuario)
    
    for iteracion in range(1, max_iter + 1):
        print(f"\n--- Iteración {iteracion} ---")
        
        # Ver si hay function calls en la respuesta
        function_calls = []
        for part in response.parts:
            if hasattr(part, 'function_call') and part.function_call.name:
                function_calls.append(part.function_call)
        
        # Si no hay function calls, terminamos
        if not function_calls:
            print("\n🏁 Agente terminó.")
            print(f"\nRespuesta:\n{response.text}")
            return
        
        # Procesar cada function call
        function_responses = []
        for fc in function_calls:
            nombre = fc.name
            # Convertir los args (que son protobuf) a dict de Python
            inputs = dict(fc.args)
            
            print(f"🔧 [Gemini] Llamando: {nombre}({inputs})")
            resultado = ejecutar_tool(nombre, inputs)
            
            # Construir el FunctionResponse
            function_responses.append(
                genai.protos.Part(
                    function_response=genai.protos.FunctionResponse(
                        name=nombre,
                        response={"resultado": resultado}
                    )
                )
            )
        
        # Mandar todas las respuestas de tools de vuelta al chat
        response = chat.send_message(function_responses)
    
    print(f"\n⚠️  Se alcanzó max iteraciones ({max_iter})")


# === ENTRY POINT ===

if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE DE ANÁLISIS DE CVE · GEMINI")
    print("=" * 60)
    
    correr_agente_gemini("Analiza el CVE-2024-12345 y dime qué hacer.")
```

## Ejecutar

```bash
(venv) $ python gemini_agent.py
```

## Diferencias importantes vs Claude y GPT

### 1 · Modelo "stateful" con `chat`

```python
# Claude / OpenAI: cada llamada es stateless, tú manejas el historial
messages = [...]
response = client.messages.create(messages=messages)

# Gemini: el chat mantiene el historial por ti
chat = model.start_chat()
response = chat.send_message(prompt)
response = chat.send_message(siguiente_input)  # historial automático
```

### 2 · Estructura `parts` en lugar de `content`

Cada respuesta tiene `parts`, donde cada part puede ser texto, function_call, etc.:

```python
for part in response.parts:
    if part.text:
        print(part.text)
    if part.function_call:
        ...
```

### 3 · Auto function calling

Gemini ofrece un modo **automático** donde el SDK ejecuta las tools por ti:

```python
chat = model.start_chat(enable_automatic_function_calling=True)
# Gemini llama internamente a tus funciones Python si están registradas
```

Lo desactivamos arriba (`=False`) para tener control explícito y poder ver el loop. Pero para ciertos casos productivos, el modo automático es útil.

### 4 · Estructura más verbose con protobuf

OpenAI y Anthropic usan dicts. Google usa clases. Más boilerplate, pero te da type checking.

> 💡 **Tip**
>
> Si el código protobuf te parece pesado, `google-generativeai` también acepta tools como **funciones Python directamente**:
>
> ```python
> def obtener_info_cve(cve_id: str):
>     """Busca info de un CVE."""
>     return ...
>
> model = genai.GenerativeModel(
>     "gemini-1.5-pro",
>     tools=[obtener_info_cve]  # tu función Python
> )
> ```
>
> El SDK extrae automáticamente el schema de los type hints y docstrings. Es mucho más limpio para casos simples. Lo usamos verbose arriba para que veas el formato canónico.

## Lo que destaca de Gemini

### Lo bueno

✅ **Contexto gigantesco**: hasta **2 millones de tokens** en Gemini 1.5 Pro (vs 200k Claude, 128k GPT-4)
✅ **Auto function calling**: modo conveniente para prototipos rápidos
✅ **Tier gratuito muy generoso**: ideal para aprender y MVPs
✅ **Multimodal nativo**: video, audio, imágenes en el mismo modelo
✅ **API stateful** (chat) más natural para conversaciones largas

### Lo menos bueno

❌ **API más verbose** con protobuf (a menos que uses el shortcut de funciones Python)
❌ **Documentación menos consistente** comparada con Anthropic
❌ **Parámetros con nombres distintos** entre diferentes versiones del SDK
❌ **Soporte de tools más reciente** que Claude/GPT (más bugs ocasionalmente)

## Características técnicas

| Aspecto | Detalle |
|---------|---------|
| **Modelos populares** | Gemini 1.5 Pro, 1.5 Flash, 2.0 Flash |
| **Contexto máximo** | 2,000,000 tokens (Pro) / 1M (Flash) |
| **Function calling stable** | Sí (estabilizándose) |
| **Streaming con tools** | Sí |
| **Multimodal** | Sí (líder en video/audio) |
| **Tier gratuito** | Generoso |
| **Costo (referencia)** | Pro $1.25/$5 per MTok, Flash $0.075/$0.30 |

## Cuándo elegir Gemini

✅ **Si necesitas procesar contextos enormes** (libros completos, miles de docs)
✅ **Si tu app es multimodal** (video, audio, imágenes mezclados)
✅ **Si tienes presupuesto ajustado** (Flash es muy barato)
✅ **Si ya estás en el ecosistema Google Cloud**

❌ Quizá no si:
- Necesitas la máxima precisión en tool calling complejo (Claude suele ganar)
- Tu equipo es 100% nuevo en LLMs (la doc de Anthropic es más amigable)
- Necesitas modelos de razonamiento profundo (o1, Opus)

---

**Siguiente: [Comparación lado a lado →](04-comparacion.md)**
