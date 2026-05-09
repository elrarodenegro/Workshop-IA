# El loop de tool_use

## El concepto, una vez más

Antes de meternos al código, recuerda los cuatro pasos del loop agéntico (de la [Parte 1.2](../../parte-1-fundamentos/12-loop-agentico.md)):

```
Percibir → Razonar → Actuar → Observar → (volver a Percibir)
```

En Tool Use de Anthropic, esto se materializa así:

| Paso del loop | Lo que pasa en el código |
|---------------|--------------------------|
| **Percibir** | Construir el array `messages` con todo lo conocido hasta ahora |
| **Razonar** | Llamar a `client.messages.create(...)` |
| **Actuar** | Si la respuesta tiene `tool_use`, ejecutar la función Python correspondiente |
| **Observar** | Agregar el resultado de la tool al array `messages` como `tool_result` |
| **Repetir** | Si `stop_reason != "end_turn"`, volver a llamar al LLM |

## El esquema de mensajes

Anthropic usa un array de **messages** alternando entre roles. Cada mensaje puede tener contenido de varios tipos:

```python
messages = [
    {"role": "user", "content": "Procesa mis correos no leídos"},
    
    {"role": "assistant", "content": [
        # El LLM puede mezclar texto Y tool_use en una sola respuesta
        {"type": "text", "text": "Voy a leer tus correos."},
        {"type": "tool_use", "id": "toolu_abc", "name": "leer_correos", "input": {"limite": 10}}
    ]},
    
    {"role": "user", "content": [
        # Le devolvemos al LLM el resultado de la tool
        {"type": "tool_result", "tool_use_id": "toolu_abc", "content": "[lista de correos...]"}
    ]},
    
    # ... el LLM responde de nuevo, posiblemente con más tool_use o el texto final
]
```

Cosas importantes a notar:

- **El `tool_use_id`** es crítico — conecta el `tool_use` con su `tool_result` correspondiente. Si no coincide, el LLM se confunde.
- **El contenido del `assistant`** puede ser texto, tool_use, o ambos.
- **El `tool_result` va en un mensaje del rol `user`** (sí, suena contraintuitivo — es como si tú, el "usuario" del LLM, le estuvieras pasando datos).

## Los stop_reason que tenemos que distinguir

Cuando el LLM responde, devuelve un campo `stop_reason` que nos dice **por qué paró de generar**:

| stop_reason | Qué significa |
|-------------|---------------|
| `end_turn` | El LLM terminó. Ya no quiere usar más tools, su respuesta final está completa. |
| `tool_use` | El LLM quiere ejecutar una o más tools. Hay que dárselas y volver a llamar. |
| `max_tokens` | Se acabó el límite de tokens de salida. Probablemente la respuesta está truncada. |
| `stop_sequence` | Se encontró una stop sequence definida. (No la usamos en este workshop.) |

**El loop se rige por estos dos casos**:

```python
while True:
    response = client.messages.create(...)
    
    if response.stop_reason == "end_turn":
        break  # El LLM terminó
    
    if response.stop_reason == "tool_use":
        # Procesamos las tools que pidió y volvemos a llamar
        ...
        continue
    
    # Si llegamos aquí, fue otro stop_reason raro
    print(f"⚠️  stop_reason inesperado: {response.stop_reason}")
    break
```

## Construyendo el loop, paso a paso

### Paso 1 · Inicializar mensajes

```python
def correr_agente(prompt_usuario: str, max_iteraciones: int = 10):
    messages = [
        {"role": "user", "content": prompt_usuario}
    ]
    
    iteracion = 0
    # ...
```

> ⚠️ **Cuidado**
>
> El `max_iteraciones` es **crítico**. Sin él, un agente puede entrar en un loop infinito. Empieza con un número bajo (10 es razonable para este workshop) y subes solo si necesitas.

### Paso 2 · Llamar al LLM

```python
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
```

### Paso 3 · Verificar el stop_reason

```python
        # Si el LLM terminó, salimos del loop
        if response.stop_reason == "end_turn":
            print("\n🏁 El agente terminó.")
            
            # Extraer el último texto de la respuesta
            for block in response.content:
                if block.type == "text":
                    print(f"\nRespuesta final:\n{block.text}")
            
            break
```

### Paso 4 · Si pidió tool_use, agregamos la respuesta del assistant a messages

```python
        # Si llegamos aquí, response.stop_reason == "tool_use"
        # IMPORTANTE: agregamos la respuesta completa al historial
        # antes de procesar las tools
        messages.append({
            "role": "assistant",
            "content": response.content  # ← incluye text Y tool_use blocks
        })
```

### Paso 5 · Ejecutar cada tool_use

Una respuesta del LLM puede contener **múltiples tool_use blocks** en paralelo. Tenemos que ejecutarlos todos y devolver un solo mensaje con todos los `tool_result`.

```python
        # Recolectar resultados de todas las tools que el LLM pidió
        tool_results = []
        
        for block in response.content:
            if block.type == "tool_use":
                tool_name = block.name
                tool_input = block.input
                tool_use_id = block.id
                
                print(f"🔧 Ejecutando: {tool_name}({tool_input})")
                
                # Ejecutar la tool
                resultado = ejecutar_tool(tool_name, tool_input)
                
                # Agregar el resultado a la lista
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": tool_use_id,
                    "content": resultado
                })
        
        # Mandar todos los tool_results en un solo mensaje
        messages.append({
            "role": "user",
            "content": tool_results
        })
```

### Paso 6 · El loop continúa

Al `continue` (implícito en el `while`), se vuelve a llamar al LLM con los mensajes actualizados. El LLM ahora ve los resultados de las tools y decide el siguiente paso.

### Paso 7 · Manejar el límite de iteraciones

```python
    if iteracion >= max_iteraciones:
        print(f"\n⚠️  Se alcanzó el máximo de iteraciones ({max_iteraciones})")
        print("El agente no terminó por sí solo. Revisa el flujo.")
```

## Visualización del flujo (real, con datos)

Imagina que el usuario dice "procesa mis correos urgentes":

### Iteración 1
- **Mensaje a Claude**: "procesa mis correos urgentes"
- **Respuesta de Claude**: 
  - text: "Voy a leer los correos no leídos"
  - tool_use: `leer_correos({"limite": 10})`
- **Tu código**: ejecuta `leer_correos()`, obtiene 5 correos mock
- **Mensaje de respuesta**: tool_result con los 5 correos

### Iteración 2
- **Mensaje a Claude**: (todo el historial + tool_result)
- **Respuesta de Claude**:
  - text: "Detecté 2 urgentes. Creo las tareas."
  - tool_use: `crear_tarea({titulo: "Reporte Q4 antes 3pm", ...})`
  - tool_use: `crear_tarea({titulo: "Sistema caído cliente X", ...})`
- **Tu código**: ejecuta ambas `crear_tarea()`
- **Mensaje de respuesta**: dos tool_results

### Iteración 3
- **Mensaje a Claude**: (todo el historial + 2 tool_results)
- **Respuesta de Claude**:
  - text: "Procesé 5 correos: 2 urgentes con tareas creadas, 1 normal, 2 spam ignorados."
  - stop_reason: `end_turn`
- **Tu código**: termina el loop, imprime la respuesta final

**Tres iteraciones**. Tres llamadas a la API. El agente decidió en cada paso qué hacer.

> 💡 **Tip**
>
> Mucha gente espera que un agente "haga todo en una sola llamada". **No funciona así.** Cada decisión del LLM es una llamada. Por eso la latencia es de segundos, no milisegundos. Y por eso vigilar el costo importa.

## El loop completo (sin envoltura)

Aquí está el loop entero, listo para integrar al `agente.py`:

```python
def correr_agente(prompt_usuario: str, max_iteraciones: int = 10):
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
    
    print(f"⚠️  Se alcanzó el máximo de iteraciones ({max_iteraciones})")
```

## Lo que tienes ahora

- ✅ Entiendes la estructura de mensajes en Tool Use
- ✅ Sabes qué hacer con cada `stop_reason`
- ✅ Tienes el loop completo escrito
- ✅ Sabes manejar tool_use múltiples en una sola respuesta

**Siguiente: [Código completo →](04-codigo-completo.md)**
