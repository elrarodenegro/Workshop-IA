# Código completo

Aquí está el archivo `agente.py` completo, todo junto y comentado. Puedes copiarlo tal cual a tu proyecto.

## El archivo completo

```python
# archivo: caso-2-claude/agente.py
"""
Agente de procesamiento de correos
==================================
Lee correos no leídos, los clasifica y crea tareas para los urgentes.

Stack: Python + Anthropic Tool Use
"""

import os
from anthropic import Anthropic
from dotenv import load_dotenv

# === SETUP ===
load_dotenv("../.env")
client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


# === DEFINICIÓN DE TOOLS (esquemas que ve el LLM) ===

tool_leer_correos = {
    "name": "leer_correos",
    "description": (
        "Lee los correos no leídos del inbox del usuario. "
        "Devuelve una lista con remitente, asunto, fecha y un snippet del cuerpo. "
        "Usa esta herramienta cuando necesites obtener la información de los correos pendientes."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "limite": {
                "type": "integer",
                "description": "Cantidad máxima de correos a devolver. Por defecto 10.",
                "default": 10
            }
        },
        "required": []
    }
}

tool_crear_tarea = {
    "name": "crear_tarea",
    "description": (
        "Crea una tarea en el sistema de gestión (Notion/Trello). "
        "Solo úsala para correos clasificados como URGENTE. "
        "Debes proveer un título claro y un resumen del contexto."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "titulo": {
                "type": "string",
                "description": "Título corto y accionable de la tarea (máximo 80 caracteres)."
            },
            "descripcion": {
                "type": "string",
                "description": "Resumen del correo y contexto relevante para entender qué hay que hacer."
            },
            "prioridad": {
                "type": "string",
                "enum": ["alta", "media", "baja"],
                "description": "Nivel de prioridad. Para correos URGENTES siempre 'alta'."
            },
            "origen_email": {
                "type": "string",
                "description": "Dirección de correo del remitente original."
            }
        },
        "required": ["titulo", "descripcion", "prioridad", "origen_email"]
    }
}

TOOLS = [tool_leer_correos, tool_crear_tarea]


# === IMPLEMENTACIÓN DE TOOLS (lo que se ejecuta) ===

def leer_correos(limite: int = 10) -> list:
    """
    Mock: devuelve correos simulados para el workshop.
    En producción, conectarías a Gmail API aquí.
    """
    correos_mock = [
        {
            "id": "msg_001",
            "de": "jefe@miempresa.com",
            "asunto": "URGENTE: Necesito el reporte hoy antes de las 3pm",
            "fecha": "2024-12-15T08:30:00Z",
            "snippet": "Hola, necesito que me envíes el reporte de ventas Q4 antes de las 3pm hoy. Tengo reunión con el cliente y es crítico tenerlo. Avísame si hay algún problema."
        },
        {
            "id": "msg_002",
            "de": "newsletter@medium.com",
            "asunto": "Top 10 articles you might like this week",
            "fecha": "2024-12-15T07:15:00Z",
            "snippet": "Discover this week's most popular articles handpicked for you based on your reading history..."
        },
        {
            "id": "msg_003",
            "de": "colega@miempresa.com",
            "asunto": "Cambio de horario en la reunión del jueves",
            "fecha": "2024-12-15T06:00:00Z",
            "snippet": "Hola, te aviso que la reunión del jueves se movió de las 10am a las 11am. Misma sala. Saludos."
        },
        {
            "id": "msg_004",
            "de": "promociones@tienda.com",
            "asunto": "Última oportunidad: 50% de descuento HOY",
            "fecha": "2024-12-15T05:00:00Z",
            "snippet": "Solo por hoy, todos los productos al 50%. No te pierdas esta oferta única..."
        },
        {
            "id": "msg_005",
            "de": "cliente@empresacliente.com",
            "asunto": "Problema crítico en producción - sistema caído",
            "fecha": "2024-12-15T09:00:00Z",
            "snippet": "Buenos días, nuestro sistema lleva 30 minutos caído y los usuarios no pueden acceder. ¿Puedes revisar urgente? Es crítico."
        }
    ]
    
    return correos_mock[:limite]


def crear_tarea(titulo: str, descripcion: str, prioridad: str, origen_email: str) -> dict:
    """
    Mock: simula crear una tarea. En producción, llamaría a Notion/Trello API.
    """
    print(f"\n  ✅ TAREA CREADA en el sistema:")
    print(f"     Título: {titulo}")
    print(f"     Prioridad: {prioridad}")
    print(f"     Descripción: {descripcion[:100]}...")
    print(f"     Origen: {origen_email}")
    
    return {
        "exito": True,
        "tarea_id": f"task_{abs(hash(titulo)) % 10000}",
        "url": f"https://notion.so/task_{abs(hash(titulo)) % 10000}"
    }


# === DISPATCHER (puente entre el LLM y las funciones) ===

def ejecutar_tool(nombre: str, inputs: dict) -> str:
    """
    Ejecuta la tool solicitada por el LLM con los inputs dados.
    Devuelve el resultado en formato string para el LLM.
    """
    if nombre == "leer_correos":
        resultado = leer_correos(**inputs)
        return str(resultado)
    
    elif nombre == "crear_tarea":
        resultado = crear_tarea(**inputs)
        return str(resultado)
    
    else:
        return f"Error: tool '{nombre}' no reconocida."


# === SYSTEM PROMPT ===

SYSTEM_PROMPT = """
Eres un asistente que procesa correos electrónicos automáticamente.

Tu trabajo:
1. Leer los correos no leídos usando la herramienta leer_correos
2. Para CADA correo, decidir si es URGENTE, NORMAL o SPAM
3. Para los URGENTES, crear una tarea con crear_tarea (prioridad="alta")
4. Para NORMAL y SPAM, no hacer nada (solo mencionarlos en tu resumen final)

Criterios de clasificación:
- URGENTE: solicitudes específicas con fecha límite cercana, problemas críticos, comunicaciones del jefe directo o clientes importantes
- NORMAL: comunicaciones de trabajo regulares, recordatorios, actualizaciones sin urgencia
- SPAM: newsletters, promociones, notificaciones automáticas

Reglas:
- Procesa TODOS los correos en una sola sesión
- Sé conciso en los títulos y descripciones de tareas
- Al terminar, da un resumen breve de cuántos correos procesaste y de qué tipo
"""


# === LOOP PRINCIPAL DEL AGENTE ===

def correr_agente(prompt_usuario: str, max_iteraciones: int = 10):
    """
    Corre el loop del agente hasta que termine o alcance el límite.
    """
    messages = [
        {"role": "user", "content": prompt_usuario}
    ]
    
    iteracion = 0
    
    while iteracion < max_iteraciones:
        iteracion += 1
        print(f"\n--- Iteración {iteracion} ---")
        
        response = client.messages.create(
            model="claude-opus-4-5",
            max_tokens=2048,
            system=SYSTEM_PROMPT,
            tools=TOOLS,
            messages=messages
        )
        
        # Caso 1: El LLM terminó
        if response.stop_reason == "end_turn":
            print("\n🏁 El agente terminó.")
            for block in response.content:
                if block.type == "text":
                    print(f"\nRespuesta final:\n{block.text}")
            
            # Imprimir uso de tokens
            usage = response.usage
            print(f"\n📊 Tokens usados (esta llamada): {usage.input_tokens} in, {usage.output_tokens} out")
            return response
        
        # Caso 2: El LLM quiere usar tools
        if response.stop_reason == "tool_use":
            # Guardar la respuesta del assistant en el historial
            messages.append({
                "role": "assistant",
                "content": response.content
            })
            
            # Ejecutar cada tool y recolectar resultados
            tool_results = []
            for block in response.content:
                if block.type == "text" and block.text.strip():
                    print(f"💭 Razonamiento: {block.text}")
                
                if block.type == "tool_use":
                    print(f"🔧 Ejecutando: {block.name}({block.input})")
                    resultado = ejecutar_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": resultado
                    })
            
            # Devolver los resultados al LLM en el siguiente mensaje
            messages.append({
                "role": "user",
                "content": tool_results
            })
            continue
        
        # Caso 3: stop_reason inesperado
        print(f"⚠️  stop_reason inesperado: {response.stop_reason}")
        return response
    
    print(f"\n⚠️  Se alcanzó el máximo de iteraciones ({max_iteraciones})")
    print("El agente no terminó por sí solo. Revisa el flujo.")


# === ENTRY POINT ===

if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE DE PROCESAMIENTO DE CORREOS")
    print("=" * 60)
    
    prompt = "Procesa todos mis correos no leídos y créame tareas para los urgentes."
    
    correr_agente(prompt, max_iteraciones=10)
    
    print("\n" + "=" * 60)
    print("Sesión terminada.")
    print("=" * 60)
```

## Líneas clave a entender

Si solo tuvieras tiempo para entender 5 líneas de este archivo, serían estas:

### Línea 1 · Pasar tools al LLM

```python
response = client.messages.create(
    ...,
    tools=TOOLS,        # ← aquí el LLM ve las herramientas disponibles
    messages=messages
)
```

### Línea 2 · Identificar que el LLM quiere usar una tool

```python
if response.stop_reason == "tool_use":
    # El LLM decidió que necesita una herramienta
```

### Línea 3 · Guardar la respuesta del LLM antes de procesarla

```python
messages.append({
    "role": "assistant",
    "content": response.content  # incluye texto Y tool_use blocks
})
```

### Línea 4 · Ejecutar y devolver el resultado con el ID correcto

```python
tool_results.append({
    "type": "tool_result",
    "tool_use_id": block.id,  # ← MISMO id que el LLM usó al pedir la tool
    "content": resultado
})
```

### Línea 5 · Saber cuándo terminar

```python
if response.stop_reason == "end_turn":
    break  # El LLM ya no quiere usar más tools
```

## Estructura mental para recordar

```
1. Tools = JSON Schema (lo que el LLM ve)
2. Tools = funciones Python (lo que se ejecuta)
3. Loop = while con stop_reason check
4. tool_use_id = pegamento entre lo que el LLM pidió y lo que devolviste
5. Mensajes = historial completo en cada iteración
```

Si entiendes esos 5 puntos, puedes construir cualquier agente de Tool Use sin asistencia.

## Lo que tienes ahora

Tu archivo `agente.py` está **completo y listo para correr**. Solo falta ejecutarlo y ver qué pasa.

**Siguiente: [Ejecutar y troubleshooting →](05-ejecutar.md)**
