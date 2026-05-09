# Ejecutar y troubleshooting

## Antes de correr

Verifica que tu `agente.py` tiene el código completo del [capítulo anterior](04-codigo-completo.md). Verifica también:

- ✅ Estás en la carpeta `caso-2-claude/`
- ✅ El venv está activo (ves `(venv)` en tu prompt)
- ✅ El `.env` tiene `ANTHROPIC_API_KEY` con tu key real

## Ejecutarlo

```bash
(venv) $ python agente.py
```

## La salida esperada

Si todo está bien configurado, vas a ver algo similar a esto (los detalles exactos varían en cada ejecución):

```
============================================================
AGENTE DE PROCESAMIENTO DE CORREOS
============================================================

--- Iteración 1 ---
💭 Razonamiento: Voy a leer los correos no leídos para procesarlos.
🔧 Ejecutando: leer_correos({'limite': 10})

--- Iteración 2 ---
💭 Razonamiento: Encontré 5 correos. El #1 (jefe) y el #5 (cliente) son urgentes...
🔧 Ejecutando: crear_tarea({'titulo': 'Enviar reporte de ventas Q4 antes de las 3pm', 'descripcion': '...', 'prioridad': 'alta', 'origen_email': 'jefe@miempresa.com'})

  ✅ TAREA CREADA en el sistema:
     Título: Enviar reporte de ventas Q4 antes de las 3pm
     Prioridad: alta
     Descripción: El jefe necesita el reporte de ventas Q4 antes de las 3pm hoy...
     Origen: jefe@miempresa.com

🔧 Ejecutando: crear_tarea({'titulo': 'Atender sistema caído del cliente X', 'descripcion': '...', 'prioridad': 'alta', 'origen_email': 'cliente@empresacliente.com'})

  ✅ TAREA CREADA en el sistema:
     Título: Atender sistema caído del cliente X
     Prioridad: alta
     Descripción: Sistema en producción del cliente lleva 30 min caído...
     Origen: cliente@empresacliente.com

--- Iteración 3 ---
🏁 El agente terminó.

Respuesta final:
Procesé 5 correos. Resumen:

URGENTES (2 tareas creadas):
- Reporte Q4 para el jefe (deadline 3pm)
- Sistema caído del cliente X

NORMALES (1):
- Cambio de horario en reunión del jueves

SPAM (2):
- Newsletter de Medium
- Promoción del 50%

📊 Tokens usados (esta llamada): 1247 in, 156 out

============================================================
Sesión terminada.
============================================================
```

🎉 **Si ves algo así, tu agente funciona correctamente.** Acabas de construir tu primer agente de IA en código puro.

## Variaciones para experimentar

Una vez que el flujo base funcione, prueba modificar:

### 1 · Cambiar el system prompt

Pídele al agente que clasifique en 4 categorías en vez de 3, o que justifique cada decisión, o que use emojis. Verás cómo cambia el comportamiento.

### 2 · Agregar más correos al mock

Inventa correos ambiguos (parcialmente urgentes, mezclados con tono comercial). Mira cómo el agente decide.

### 3 · Cambiar el modelo

```python
model="claude-3-5-haiku-20241022"  # más rápido y barato
# vs
model="claude-opus-4-5"  # más capaz pero más caro
```

Compara la calidad de las decisiones con cada uno.

### 4 · Agregar una tool más

Por ejemplo, `enviar_resumen_diario` que el agente use al final para mandarte un email con el resumen. Define el esquema, implementa la función mock, agrégala a `TOOLS`. Ese ejercicio te da fluidez.

## Troubleshooting

### "AuthenticationError: invalid x-api-key"

**Causa**: tu key de Anthropic está mal o no se cargó.

**Solución**:
1. Verifica que el `.env` esté en el directorio padre (`agentes-workshop/.env`, no `caso-2-claude/.env`)
2. Verifica que `load_dotenv("../.env")` apunte correctamente
3. Verifica que la key no tenga espacios al inicio o final
4. Prueba con un script mínimo para aislar el problema:

```python
import os
from dotenv import load_dotenv
load_dotenv("../.env")
print(os.getenv("ANTHROPIC_API_KEY"))  # ¿imprime tu key o None?
```

### "ModuleNotFoundError: No module named 'anthropic'"

**Causa**: el venv no está activo, o la librería no está instalada.

**Solución**:
1. Verifica que `(venv)` aparezca en tu prompt
2. Si no aparece, actívalo: `source ../venv/bin/activate` (o el equivalente Windows)
3. Si está activo pero sigue fallando: `pip install anthropic`

### "model not found"

**Causa**: el nombre del modelo no es válido o no está disponible para tu cuenta.

**Solución**:
1. Verifica el nombre exacto en la [documentación de Anthropic](https://docs.anthropic.com/en/docs/about-claude/models)
2. Si `claude-opus-4-5` no funciona, prueba con un modelo conocido como `claude-3-5-sonnet-20241022`
3. El nombre del modelo cambia con cada versión nueva; el SDK no lo valida hasta la llamada

### El agente entra en loop infinito

**Síntoma**: el contador de iteraciones sigue subiendo y nunca llega a `end_turn`.

**Causas comunes**:

1. **El system prompt no es claro sobre cuándo terminar**
   - Solución: agrega "Cuando termines de procesar todos los correos, responde con un resumen final SIN llamar más tools."

2. **El LLM cree que le falta información**
   - Mira los `💭 Razonamiento` que imprimimos. Si dice "necesito leer más correos", limita explícitamente: "Solo lee correos UNA vez."

3. **Hay una tool que devuelve algo confuso**
   - Verifica los outputs de `leer_correos()` y `crear_tarea()`. Si devuelven errores en string, el LLM puede intentar llamarlas de nuevo.

**Workaround inmediato**: bajaste el `max_iteraciones` a 5 en lugar de 10. Mejor que el agente diga "no pude" a que consuma toda tu cuota.

### "tool_use_id mismatch" o errores de mensajes

**Causa**: estás devolviendo `tool_result` con un `tool_use_id` que no coincide con lo que el LLM pidió.

**Solución**: usa exactamente `block.id` del `tool_use` correspondiente. Si tienes múltiples tools en una iteración, recolecta todos los `tool_result` antes de mandar el mensaje.

```python
# ✅ Correcto
for block in response.content:
    if block.type == "tool_use":
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,  # ← el id que vino del LLM
            ...
        })
```

### "La respuesta del LLM se cortó a la mitad"

**Causa**: alcanzaste el límite de `max_tokens`.

**Solución**: súbelo en la llamada:

```python
response = client.messages.create(
    model="...",
    max_tokens=4096,  # ← antes era 2048
    ...
)
```

### El agente clasifica algo mal

**Causa**: tu system prompt no es suficientemente claro, o el modelo es limitado.

**Solución**:
1. Agrega ejemplos few-shot al prompt (qué cuenta como urgente, cuál no)
2. Sube a un modelo más capaz (Sonnet → Opus)
3. Pídele que justifique cada decisión antes de actuar

### Quiero ver qué se está enviando exactamente al LLM

Agrega prints temporales antes de la llamada:

```python
import json
print(f"Mensajes enviados al LLM:")
print(json.dumps(messages, indent=2, default=str))
```

Esto te muestra el historial completo. Útil para entender qué "ve" el LLM en cada iteración.

## Verificación de costos

Agrega esto al final del loop para ver el total:

```python
# Tracking de tokens totales
total_input = 0
total_output = 0

# Dentro del loop, después de cada call:
total_input += response.usage.input_tokens
total_output += response.usage.output_tokens

# Al terminar:
costo_aprox = (total_input * 0.015 + total_output * 0.075) / 1000
print(f"Total: {total_input} in + {total_output} out tokens")
print(f"Costo aproximado: ${costo_aprox:.4f} USD")
```

> 📌 **Nota**
>
> Los precios cambian. Verifica los actuales en la [página de pricing de Anthropic](https://www.anthropic.com/pricing). El cálculo de arriba usa precios aproximados de Opus.

## Cómo conectar a APIs reales

Cuando quieras pasar de mocks a real:

### Para `leer_correos`

Reemplaza la implementación con:

```python
from googleapiclient.discovery import build
from google.oauth2.credentials import Credentials

def leer_correos(limite: int = 10) -> list:
    creds = Credentials.from_authorized_user_file('token.json')
    service = build('gmail', 'v1', credentials=creds)
    
    results = service.users().messages().list(
        userId='me',
        q='is:unread',
        maxResults=limite
    ).execute()
    
    # ... transformar a la estructura esperada
    return correos
```

(Esto requiere setup de OAuth con Google. Es un capítulo en sí mismo.)

### Para `crear_tarea`

```python
from notion_client import Client

notion = Client(auth=os.getenv("NOTION_TOKEN"))

def crear_tarea(titulo, descripcion, prioridad, origen_email):
    response = notion.pages.create(
        parent={"database_id": os.getenv("NOTION_DB_ID")},
        properties={
            "Name": {"title": [{"text": {"content": titulo}}]},
            "Priority": {"select": {"name": prioridad}},
            "Description": {"rich_text": [{"text": {"content": descripcion}}]}
        }
    )
    return {"exito": True, "url": response["url"]}
```

**El esqueleto del agente no cambia.** Solo cambian las dos funciones.

## Lo que aprendiste

Si llegaste hasta aquí, tienes en tu cabeza:

- ✅ Cómo definir tools como JSON Schema
- ✅ Cómo implementar las funciones que las ejecutan
- ✅ Cómo construir el loop de Tool Use
- ✅ Cómo manejar mensajes, stop_reasons y tool_use_ids
- ✅ Cómo debuggear cuando algo falla
- ✅ Cómo migrar de mocks a APIs reales

Ese conocimiento es **transferible a cualquier framework** (LangChain, LangGraph, CrewAI). Todos los frameworks construyen abstracciones sobre exactamente este loop. Si entiendes el loop, los frameworks son solo azúcar sintáctico.

> 💡 **Tip**
>
> Te recomiendo construir un segundo agente desde cero, sin mirar este código, para internalizar el patrón. Algo simple — un agente que haga búsquedas web y resuma. Si lo construyes en 30 minutos, ya dominas el Nivel 1.

---

**Has terminado el Caso 2.** Tu primer agente de Tool Use en Python está corriendo.

**Siguiente: [Caso 3 · El mismo agente, 3 modelos →](../caso-3-comparacion/README.md)**
