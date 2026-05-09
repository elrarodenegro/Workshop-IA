# Definir las herramientas

## Las dos tools del agente

Para replicar el flujo del Caso 1 en Python, nuestro agente necesita dos herramientas:

1. **`leer_correos`** — devuelve los correos no leídos del inbox
2. **`crear_tarea`** — crea una tarea en el sistema de gestión

Vamos a definirlas en dos partes:

- **El esquema** (JSON Schema) — lo que el LLM ve
- **La implementación** (función Python) — lo que realmente se ejecuta

## Cómo se ven las tools en formato Anthropic

El SDK de Anthropic espera las tools como una lista de dicts con esta estructura:

```python
{
    "name": "nombre_de_la_tool",
    "description": "Descripción clara y específica de qué hace",
    "input_schema": {
        "type": "object",
        "properties": {
            "param1": {
                "type": "string",
                "description": "Qué representa este parámetro"
            }
        },
        "required": ["param1"]
    }
}
```

**Las tres claves importantes:**

| Clave | Para qué sirve |
|-------|----------------|
| `name` | Identificador único. El LLM lo usa cuando decide invocarla. |
| `description` | El LLM lee esto para entender **cuándo usarla**. Crítico que sea claro. |
| `input_schema` | JSON Schema estándar. Define qué parámetros recibe. |

> ⚠️ **Cuidado**
>
> La calidad de tu agente depende de la calidad de tus descripciones. Si dices "lee correos", el LLM no sabe si lee todos, solo no leídos, los importantes, etc. **Sé específico.**

## Tool 1 · leer_correos

### Esquema

Agrega esto a tu `agente.py` después de la sección de imports y setup:

```python
# === DEFINICIÓN DE TOOLS ===

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
```

**Lo que el LLM va a ver y decidir**:
- "¿Tengo que leer correos? Sí → llamo `leer_correos`."
- "¿Cuántos? Si el usuario no especifica, uso el default 10."

### Implementación (mock)

Para el workshop, vamos a simular los correos con datos en memoria:

```python
# === IMPLEMENTACIÓN DE TOOLS ===

def leer_correos(limite: int = 10) -> list:
    """
    Mock: devuelve correos simulados para el workshop.
    En producción, conectarías a Gmail API aquí.
    """
    # Datos simulados — variados a propósito para que el agente clasifique
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
```

> 💡 **Tip**
>
> En producción, esta función llamaría a la API real de Gmail con OAuth. La estructura del código del agente **no cambiaría**: solo cambiarías el cuerpo de esta función. Esa es la belleza de aislar las tools en funciones independientes.

## Tool 2 · crear_tarea

### Esquema

```python
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
```

**Lo que el LLM ve y decide**:
- "Este correo es urgente. Necesito crear una tarea."
- "Voy a invocar `crear_tarea` con un título descriptivo, un resumen, prioridad alta, y el email del remitente."

### Implementación (mock)

```python
def crear_tarea(titulo: str, descripcion: str, prioridad: str, origen_email: str) -> dict:
    """
    Mock: simula crear una tarea. En producción, llamaría a la API de Notion/Trello.
    """
    print(f"\n✅ TAREA CREADA en el sistema:")
    print(f"   Título: {titulo}")
    print(f"   Prioridad: {prioridad}")
    print(f"   Descripción: {descripcion[:100]}...")
    print(f"   Origen: {origen_email}\n")
    
    # Simulamos un ID de tarea creada
    return {
        "exito": True,
        "tarea_id": f"task_{abs(hash(titulo)) % 10000}",
        "url": f"https://notion.so/task_{abs(hash(titulo)) % 10000}"
    }
```

## La lista completa de tools

Junta los dos esquemas en una lista:

```python
TOOLS = [tool_leer_correos, tool_crear_tarea]
```

Esta `TOOLS` es la que vamos a pasar al LLM en cada llamada.

## El dispatcher (puente entre el LLM y las funciones)

Cuando el LLM decida invocar una tool, recibimos un objeto con el nombre de la tool y los inputs. Necesitamos un "dispatcher" que mapee ese nombre a la función Python correspondiente:

```python
def ejecutar_tool(nombre: str, inputs: dict):
    """
    Ejecuta la tool solicitada por el LLM con los inputs dados.
    Devuelve el resultado en formato string (lo que el LLM espera de vuelta).
    """
    if nombre == "leer_correos":
        resultado = leer_correos(**inputs)
        return str(resultado)
    
    elif nombre == "crear_tarea":
        resultado = crear_tarea(**inputs)
        return str(resultado)
    
    else:
        return f"Error: tool '{nombre}' no reconocida."
```

> 📌 **Nota**
>
> El `**inputs` desempaqueta el dict de inputs como argumentos nombrados de la función. Si `inputs = {"titulo": "X", "prioridad": "alta", ...}`, eso equivale a llamar `crear_tarea(titulo="X", prioridad="alta", ...)`.

## El system prompt

El último elemento antes del loop: las instrucciones que recibe el LLM.

```python
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
```

> 💡 **Tip**
>
> Cambiar este system prompt cambia el comportamiento del agente. **Es donde más vas a iterar** cuando los resultados no sean los esperados. Considéralo el manual del agente.

## Lo que tienes ahora

Tu `agente.py` ahora tiene:

```python
# 1. Imports y setup
# 2. Definición de tools (esquemas JSON)
# 3. Implementación de tools (funciones mock)
# 4. Dispatcher
# 5. System prompt
# (falta el loop)
```

Ya casi tienes todo. **Lo único que falta es el corazón del agente**: el loop. Ese es el siguiente capítulo.

---

**Siguiente: [El loop de tool_use →](03-loop.md)**
