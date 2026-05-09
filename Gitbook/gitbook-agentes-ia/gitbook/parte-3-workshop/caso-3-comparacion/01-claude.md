# Implementación con Claude (Anthropic)

## Las tools en formato Anthropic

Recuerda del Caso 2: las tools son dicts con `name`, `description`, `input_schema`.

```python
# archivo: caso-3-comparacion/claude_agent.py (parte 1)

import os
from anthropic import Anthropic
from dotenv import load_dotenv
from mocks import obtener_info_cve, sugerir_remediacion, SYSTEM_PROMPT

load_dotenv("../.env")
client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


# === DEFINICIÓN DE TOOLS (formato Anthropic) ===

TOOLS = [
    {
        "name": "obtener_info_cve",
        "description": "Busca información de un CVE específico en la base de datos.",
        "input_schema": {
            "type": "object",
            "properties": {
                "cve_id": {
                    "type": "string",
                    "description": "ID del CVE en formato 'CVE-YYYY-NNNN'"
                }
            },
            "required": ["cve_id"]
        }
    },
    {
        "name": "sugerir_remediacion",
        "description": "Sugiere pasos de remediación basados en el CVE y su severidad.",
        "input_schema": {
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
]
```

## El dispatcher

```python
# archivo: caso-3-comparacion/claude_agent.py (parte 2)

def ejecutar_tool(nombre: str, inputs: dict) -> str:
    if nombre == "obtener_info_cve":
        return str(obtener_info_cve(**inputs))
    elif nombre == "sugerir_remediacion":
        return str(sugerir_remediacion(**inputs))
    else:
        return f"Tool desconocida: {nombre}"
```

## El loop

```python
# archivo: caso-3-comparacion/claude_agent.py (parte 3)

def correr_agente_claude(prompt_usuario: str, max_iter: int = 10):
    messages = [{"role": "user", "content": prompt_usuario}]
    
    for iteracion in range(1, max_iter + 1):
        print(f"\n--- Iteración {iteracion} ---")
        
        response = client.messages.create(
            model="claude-opus-4-5",
            max_tokens=2048,
            system=SYSTEM_PROMPT,
            tools=TOOLS,
            messages=messages
        )
        
        # Terminamos si el LLM dice end_turn
        if response.stop_reason == "end_turn":
            print("\n🏁 Agente terminó.")
            for block in response.content:
                if block.type == "text":
                    print(f"\nRespuesta:\n{block.text}")
            return
        
        # Si pidió tools, las procesamos
        if response.stop_reason == "tool_use":
            messages.append({
                "role": "assistant",
                "content": response.content
            })
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"🔧 [Claude] Llamando: {block.name}({block.input})")
                    resultado = ejecutar_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": resultado
                    })
            
            messages.append({"role": "user", "content": tool_results})
            continue
        
        # Otro stop_reason
        print(f"⚠️  stop_reason inesperado: {response.stop_reason}")
        break


# === ENTRY POINT ===

if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE DE ANÁLISIS DE CVE · CLAUDE")
    print("=" * 60)
    
    correr_agente_claude("Analiza el CVE-2024-12345 y dime qué hacer.")
```

## Ejecutar

```bash
(venv) $ python claude_agent.py
```

Salida esperada:

```
============================================================
AGENTE DE ANÁLISIS DE CVE · CLAUDE
============================================================

--- Iteración 1 ---
🔧 [Claude] Llamando: obtener_info_cve({'cve_id': 'CVE-2024-12345'})

--- Iteración 2 ---
🔧 [Claude] Llamando: sugerir_remediacion({'cve_id': 'CVE-2024-12345', 'severidad': 'CRÍTICA'})

--- Iteración 3 ---
🏁 Agente terminó.

Respuesta:
Análisis del CVE-2024-12345:

**Descripción**: Remote Code Execution en componente X via deserialización insegura.
**Severidad**: CRÍTICA (CVSS 9.8)
**Versiones afectadas**: v1.0 - v2.5

**Urgencia**: Inmediata (<24h)

**Pasos prioritarios**:
1. Aplicar parche oficial del vendor inmediatamente
2. Aislar sistemas afectados de la red
3. Buscar IOCs en logs de los últimos 30 días
4. Notificar al CISO y al equipo de IR
```

## Lo que destaca de Anthropic

### Lo bueno

✅ **Estructura más limpia**: `name`, `description`, `input_schema`. Sin envolturas extras.
✅ **Bloques de contenido**: poder mezclar texto y tool_use en una sola respuesta es elegante
✅ **stop_reason explícito**: claro saber por qué terminó
✅ **Documentación clara** y consistente

### Lo menos bueno

❌ **`tool_result` va en `role: "user"`** — confunde al inicio. Conceptualmente "le devuelves al asistente lo que pidió", pero formalmente vives en el lado del usuario.
❌ **Hay que recordar el `tool_use_id`** para mantener la coherencia del historial.

## Características técnicas

| Aspecto | Detalle |
|---------|---------|
| **Modelos populares** | Claude Opus, Sonnet, Haiku |
| **Contexto máximo** | 200,000 tokens |
| **Tool use stable** | Sí, GA |
| **Streaming con tools** | Sí |
| **Cache de prompts** | Sí (reduce costos) |
| **Costo (referencia)** | Opus $15/$75 per MTok in/out |

> 📌 **Nota**
>
> Los precios y modelos cambian. Verifica los actuales en [anthropic.com/pricing](https://www.anthropic.com/pricing).

## Cuándo elegir Claude

✅ **Tareas que requieren razonamiento profundo** (Opus es famoso por esto)
✅ **Tool use con muchas herramientas** (~10+ tools)
✅ **Largos contextos** (procesar documentos completos)
✅ **Cuando la calidad importa más que el costo**

❌ Quizá no lo elijas si:
- Volumen masivo con margen ajustado
- Necesitas embedding nativo (Anthropic no tiene embeddings, OpenAI sí)
- Necesitas integraciones específicas que solo OpenAI tiene

---

**Siguiente: [Implementación con GPT-4 →](02-gpt.md)**
