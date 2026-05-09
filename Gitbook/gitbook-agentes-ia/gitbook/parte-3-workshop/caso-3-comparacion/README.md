# Caso 3 · El mismo agente, 3 modelos

> *Mismo problema, tres ergonomías distintas.*

## Qué vamos a hacer

Vamos a construir **el mismo agente** con tres modelos diferentes:

- **Claude** (Anthropic) usando Tool Use
- **GPT-4** (OpenAI) usando Function Calling
- **Gemini** (Google) usando Function Calling

Y vamos a compararlos lado a lado: formato de los mensajes, ergonomía del SDK, errores comunes, costos.

## El agente que vamos a construir

Para variar del caso anterior, vamos a hacer un agente más **técnico** y útil para alguien en seguridad:

> **Agente analizador de CVEs**: recibe un CVE-ID, busca su severidad, y sugiere remediación.

Tools que necesita:

1. **`obtener_info_cve(cve_id)`** — devuelve la severidad, descripción y CVSS score
2. **`sugerir_remediacion(cve_id, severidad)`** — devuelve pasos sugeridos de mitigación

Vamos a usar mocks (datos simulados) para no depender de APIs externas.

## ¿Por qué hacer esto?

Es la forma más rápida de **realmente entender** las diferencias entre proveedores. Las explicaciones en blogs son genéricas. Cuando construyes lo mismo tres veces, ves de primera mano:

- Cuál SDK es más ergonómico
- Cuál tiene mejor manejo de errores
- Cuál te ahorra más código boilerplate
- Cuál es más rápido / barato para tu caso
- Cuál tiene mejor calidad en tool calling

Después de este caso vas a poder elegir LLM **con criterio técnico**, no solo por marketing.

## Lo que aprenderás

- ✅ Cómo se define una tool en cada SDK (Anthropic, OpenAI, Google)
- ✅ Cómo se procesa la respuesta en cada uno
- ✅ Diferencias en estructura de mensajes
- ✅ Manejo de errores específico de cada plataforma
- ✅ Criterios para elegir un proveedor

## Tiempo estimado

| Sección | Tiempo |
|---------|--------|
| Setup del proyecto | 5 min |
| Implementación con Claude | 8 min |
| Implementación con GPT-4 | 10 min |
| Implementación con Gemini | 10 min |
| Comparación lado a lado | 7 min |
| **Total** | **~40 min** |

## Stack

| Componente | Versión mínima |
|------------|----------------|
| Python | 3.10+ |
| anthropic | 0.40.0 |
| openai | 1.50.0 |
| google-generativeai | 0.8.0 |
| python-dotenv | 1.0.0 |

Las cuatro deberían estar ya en tu `requirements.txt` del [capítulo 0.4](../../parte-0-setup/04-venv.md).

## Las tres APIs en una vista

Antes de meternos al código, tabla rápida de cómo cada SDK nombra las cosas:

| Concepto | Anthropic | OpenAI | Google |
|----------|-----------|--------|--------|
| **Definir tool** | `tools=[{...}]` | `tools=[{type:"function", function:{...}}]` | `tools=[{function_declarations:[{...}]}]` |
| **Schema de input** | `input_schema` | `parameters` | `parameters` |
| **El LLM pide tool** | `stop_reason: "tool_use"` | `finish_reason: "tool_calls"` | `parts[].function_call` |
| **Devolver resultado** | `role: "user"` con `tool_result` | `role: "tool"` con `tool_call_id` | `parts[].function_response` |
| **Identificador de la llamada** | `id` (en el tool_use block) | `tool_call_id` | (no aplica explícito) |

Ya solo de esa tabla puedes intuir que **OpenAI tiene la estructura más anidada** (`tools[].function.parameters`), Anthropic es la más simple (`tools[].input_schema`), y Google está entremedias.

## Lo que NO vamos a hacer

❌ No vamos a hacer un benchmark exhaustivo (eso requiere cientos de prompts y métricas formales)
❌ No vamos a optimizar para producción (mantenemos el código educativo)
❌ No vamos a integrar las tres en un solo agente abstracto (eso es lo que hacen frameworks como LangChain — y es justamente la abstracción que queremos entender)

## Estructura de este caso

1. **[Implementación con Claude](01-claude.md)** — Tool Use de Anthropic
2. **[Implementación con GPT-4](02-gpt.md)** — Function Calling de OpenAI
3. **[Implementación con Gemini](03-gemini.md)** — Function Calling de Google
4. **[Comparación lado a lado](04-comparacion.md)** — formato, ergonomía, costo, cuándo usar cuál

## El setup común para los tres

Vamos a crear primero los **mocks compartidos** que usarán las tres implementaciones.

```bash
$ mkdir caso-3-comparacion
$ cd caso-3-comparacion
$ touch claude_agent.py gpt_agent.py gemini_agent.py mocks.py
```

Crea `mocks.py` con esto (lo van a importar los tres archivos):

```python
# archivo: caso-3-comparacion/mocks.py
"""
Datos y funciones simuladas compartidas por las tres implementaciones.
"""

CVE_DATABASE = {
    "CVE-2024-12345": {
        "severidad": "CRÍTICA",
        "cvss_score": 9.8,
        "descripcion": "Remote Code Execution en componente X via deserialización insegura.",
        "afecta_a": ["v1.0 - v2.5"],
        "publicado": "2024-11-20"
    },
    "CVE-2024-99999": {
        "severidad": "MEDIA",
        "cvss_score": 5.4,
        "descripcion": "Cross-site scripting (XSS) reflejado en el endpoint /search.",
        "afecta_a": ["v3.x"],
        "publicado": "2024-10-15"
    },
    "CVE-2023-00001": {
        "severidad": "BAJA",
        "cvss_score": 3.2,
        "descripcion": "Information disclosure menor en logs de error.",
        "afecta_a": ["todas las versiones"],
        "publicado": "2023-08-01"
    }
}


def obtener_info_cve(cve_id: str) -> dict:
    """Mock: busca info de CVE en una "base de datos" simulada."""
    if cve_id in CVE_DATABASE:
        return {"encontrado": True, **CVE_DATABASE[cve_id]}
    return {"encontrado": False, "error": f"CVE {cve_id} no existe en la base de datos."}


def sugerir_remediacion(cve_id: str, severidad: str) -> dict:
    """Mock: devuelve pasos de remediación según severidad."""
    if severidad == "CRÍTICA":
        return {
            "urgencia": "Inmediata (<24h)",
            "pasos": [
                "1. Aplicar parche oficial del vendor inmediatamente",
                "2. Aislar sistemas afectados de la red",
                "3. Buscar IOCs en logs de los últimos 30 días",
                "4. Notificar al CISO y al equipo de IR"
            ]
        }
    elif severidad == "MEDIA":
        return {
            "urgencia": "Esta semana",
            "pasos": [
                "1. Aplicar parche en próxima ventana de mantenimiento",
                "2. Implementar WAF rules como mitigación temporal",
                "3. Documentar en risk register"
            ]
        }
    else:
        return {
            "urgencia": "Próximo ciclo de actualización",
            "pasos": [
                "1. Incluir en el ciclo regular de patching",
                "2. Documentar para tracking"
            ]
        }


SYSTEM_PROMPT = """
Eres un asistente de seguridad cibernética especializado en análisis de CVEs.

Cuando el usuario te dé un CVE-ID:
1. Usa obtener_info_cve para conseguir la información
2. Si existe, usa sugerir_remediacion con la severidad obtenida
3. Resume al usuario: descripción, severidad, urgencia y pasos prioritarios

Sé técnico pero claro. No inventes información si la base de datos no la tiene.
"""
```

Listo. Ahora podemos enfocarnos en los tres SDKs por separado, todos usando estos mocks.

---

**Siguiente: [Implementación con Claude →](01-claude.md)**
