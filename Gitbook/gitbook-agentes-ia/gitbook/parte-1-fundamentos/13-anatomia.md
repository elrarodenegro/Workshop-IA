# 1.3 · Anatomía de un agente

## Los 5 componentes

Cualquier agente que mires por dentro, sin importar su nivel de sofisticación, tiene estos cinco componentes:

```
1. Modelo
2. Instrucciones
3. Herramientas
4. Memoria
5. Orquestador
```

La mejor analogía: **piensa en un asistente humano nuevo en su primer día de trabajo.**

## 1 · Modelo (Model / LLM)

**El cerebro.** El LLM que razona, decide y genera el lenguaje.

> Como el cerebro de la persona contratada.

Decisiones que tomas aquí:

- **Qué proveedor**: Anthropic, OpenAI, Google, modelos open-source
- **Qué modelo específico**: el más capaz (caro y lento) o uno más rápido (barato y razonable)
- **Parámetros**: temperatura (creatividad vs. precisión), max_tokens (límite de respuesta)

> 💡 **Tip**
>
> Para agentes en producción, **empieza con el modelo más capaz** (Claude Opus, GPT-4, etc.). Una vez funcione, considera bajar a modelos más baratos (Sonnet, GPT-4o-mini) para reducir costos. Optimizar prematuramente con un modelo débil te hace perseguir errores que vienen de la limitación del modelo, no de tu diseño.

## 2 · Instrucciones (System Prompt)

**Las reglas, el tono, los objetivos.** Es el "manual del puesto" del agente.

> Como el documento que le entregan al empleado nuevo: aquí está tu rol, estos son los procedimientos, esto es lo que NO debes hacer.

Un buen system prompt define:

- **Identidad**: "Eres un asistente de soporte técnico para [empresa]"
- **Objetivo**: "Tu meta es resolver el ticket del usuario o escalarlo a un humano"
- **Restricciones**: "No prometas reembolsos sin escalar primero"
- **Tono**: "Responde de forma cordial pero concisa, máximo 3 párrafos"
- **Formato**: "Cuando uses la herramienta `crear_ticket`, siempre incluye el campo prioridad"

### Ejemplo de system prompt simple

```python
system_prompt = """
Eres un asistente que ayuda a procesar correos electrónicos.

Tu trabajo es:
1. Leer el correo recibido
2. Clasificarlo como: urgente, normal, o spam
3. Si es urgente, crear una tarea inmediata en Notion
4. Si es normal, agregarlo a una lista de revisión
5. Si es spam, ignorarlo

Reglas:
- Nunca respondas directamente al correo
- Siempre justifica tu clasificación
- Si tienes duda entre normal y urgente, elige urgente
"""
```

> 📌 **Nota**
>
> El system prompt **no es una sugerencia para el LLM, es su norte**. Cambiar una palabra puede cambiar drásticamente el comportamiento. Itera tu prompt como iterarías código.

## 3 · Herramientas (Tools)

**Las APIs y funciones que el agente puede invocar.** Las "credenciales y accesos" del empleado.

> Como darle al empleado nuevo: el login del email corporativo, acceso al CRM, contraseñas del Slack.

Una herramienta tiene típicamente:

- **Nombre**: cómo se llama (ej. `enviar_correo`, `buscar_cliente`)
- **Descripción**: para qué sirve (le ayuda al LLM a decidir cuándo usarla)
- **Esquema de inputs**: qué parámetros recibe y de qué tipo
- **Esquema de outputs**: qué devuelve

### Ejemplo de definición de herramienta (formato Anthropic)

```python
tool = {
    "name": "obtener_clima",
    "description": "Obtiene el pronóstico del clima para una ciudad y fecha específica.",
    "input_schema": {
        "type": "object",
        "properties": {
            "ciudad": {
                "type": "string",
                "description": "Nombre de la ciudad. Ejemplo: 'Bogotá', 'Medellín'."
            },
            "fecha": {
                "type": "string",
                "description": "Fecha en formato YYYY-MM-DD. 'hoy' o 'mañana' también son válidos."
            }
        },
        "required": ["ciudad", "fecha"]
    }
}
```

El LLM lee la descripción y los parámetros, y decide cuándo invocarla. **Por eso las descripciones deben ser claras y específicas** — son la forma en que el LLM "entiende" para qué sirve cada herramienta.

> ⚠️ **Cuidado**
>
> No le des al agente más herramientas de las necesarias. Cada herramienta extra es:
> - Más tokens en el contexto (más caro)
> - Más decisiones que tomar (más errores)
> - Más superficie de ataque (más riesgo)
>
> Empieza con 1-3 herramientas. Crece solo cuando necesites.

## 4 · Memoria (Memory)

**Lo que el agente recuerda.** La libreta de notas del empleado.

Hay dos tipos de memoria:

### Memoria de corto plazo (Conversation history)

El historial de la conversación actual. Va dentro del contexto que envías al LLM en cada llamada.

**Limitada por**: el tamaño del contexto del modelo (200k tokens para Claude, 128k para GPT-4, 1M para Gemini).

### Memoria de largo plazo (Persistent memory)

Información que persiste entre conversaciones. Se guarda en una base de datos (típicamente una **vector DB** como Pinecone, Chroma o pgvector) y se recupera por relevancia semántica.

**Útil para**:
- Recordar interacciones previas con un usuario específico
- Mantener una base de conocimiento que crece con el tiempo
- Recordar feedback recibido sobre respuestas anteriores

> 📌 **Nota**
>
> En el Caso 2 del workshop usamos solo memoria de corto plazo (la conversación actual). En agentes de producción es común tener ambas.

## 5 · Orquestador (Orchestrator)

**Quién decide el flujo entre los componentes.** El supervisor que coordina al empleado.

Hay dos enfoques principales:

### Orquestación por el LLM

El LLM decide qué herramienta usar, en qué orden, y cuándo terminar. Tu código solo ejecuta lo que el LLM diga.

```
Usuario → LLM (decide) → Tool → LLM (siguiente decisión) → Tool → ... → respuesta
```

**Es lo que hace Tool Use de Anthropic, function calling de OpenAI.**

### Orquestación por código

Tu código define el flujo (workflow). El LLM se invoca solo en pasos específicos: clasificar, extraer datos, generar texto. **Cada paso del workflow está predefinido.**

```
Usuario → tu_código → LLM clasifica → tu_código → LLM extrae datos → tu_código → respuesta
```

**Es lo que hacen herramientas como n8n, Zapier con AI, LangChain chains.**

### ¿Cuál usar?

| Si... | Usa... |
|-------|--------|
| El flujo cambia mucho según el caso | Orquestación por LLM |
| El flujo es relativamente fijo | Orquestación por código |
| Necesitas máxima predictibilidad | Orquestación por código |
| Necesitas máxima flexibilidad | Orquestación por LLM |

En la práctica, casi siempre acabas mezclando: **el flujo macro lo controla tu código**, pero **dentro de un paso le das al LLM autonomía con tools**.

## La analogía completa

Imagina contratar un asistente nuevo para tu empresa:

| Componente | Equivalente humano |
|------------|--------------------|
| **Modelo** | El cerebro y experiencia que tiene |
| **Instrucciones** | El manual del puesto que le entregas |
| **Herramientas** | Las credenciales y accesos a sistemas |
| **Memoria** | Su libreta de notas y archivos |
| **Orquestador** | El jefe que coordina sus tareas |

Si te falta cualquiera de los cinco, el asistente es ineficaz.

## En el Caso 2 del workshop

Cuando construyamos el agente en Python, vas a ver explícitamente cada componente:

```python
# 1. Modelo
client = Anthropic()

# 2. Instrucciones
system_prompt = "Eres un asistente que..."

# 3. Herramientas
tools = [{"name": "leer_correos", ...}, {"name": "crear_tarea", ...}]

# 4. Memoria (la conversación actual)
messages = []

# 5. Orquestador (nuestro propio loop while)
while not terminado:
    response = client.messages.create(...)
    # ...
```

Cinco bloques claros. Cada uno haciendo su trabajo.

## Recapitulando

- Todo agente tiene **5 componentes**: Modelo, Instrucciones, Herramientas, Memoria, Orquestador
- Cada uno tiene un análogo humano que ayuda a razonar sobre el diseño
- Faltarte uno solo hace que el agente no funcione bien
- El **orquestador** puede vivir en el LLM o en tu código — o en ambos

> 🎯 **Reto**
>
> Para un agente que automatice una tarea real de tu trabajo, define los 5 componentes en una hoja de papel:
>
> - Qué modelo usarías
> - Qué reglas le darías en el system prompt
> - Qué 3-5 herramientas necesitaría
> - Qué tendría que recordar
> - Quién orquesta el flujo
>
> Si no puedes definir alguno claramente, ese agente todavía no está listo para construirse.

---

**Siguiente: [1.4 · Agentes vs automatización tradicional →](14-agentes-vs-tradicional.md)**
