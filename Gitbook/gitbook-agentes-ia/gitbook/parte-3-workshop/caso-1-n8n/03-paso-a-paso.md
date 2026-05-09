# Construcción paso a paso

Vamos a llenar el canvas vacío con nodos, uno por uno. Cada paso te explica **qué hacer**, **qué configurar** y **por qué**.

> 💡 **Tip**
>
> n8n actualiza su UI con frecuencia. Si los nombres exactos de los menús no coinciden 1:1 con lo que ves, busca por la palabra clave (`Gmail Trigger`, `AI Agent`, `Switch`, etc.) — el concepto es estable aunque el branding cambie.

## Nodo 1 · Gmail Trigger

Este nodo es el "punto de entrada" del workflow. Cada cierto tiempo va a buscar correos no leídos y arrancar el flujo.

### Agregar el nodo

1. En el canvas, haz clic en **+** (Add first step).
2. En el panel que se abre, busca **Gmail**.
3. Selecciona **Gmail Trigger**.
4. Si te pregunta qué evento, elige **On Message Received**.

### Configurar

- **Credential**: selecciona la credencial Gmail que creaste
- **Trigger On**: `Message Received`
- **Filters**:
  - **Read Status**: `Unread only` (solo correos no leídos)
- **Polling**: `Every 15 minutes`

### Probar

Haz clic en **Execute Step** o **Test step**. Si tienes correos no leídos en tu inbox, deberías ver una lista de ellos en el panel de output a la derecha.

> ⚠️ **Cuidado**
>
> Si tu inbox tiene 500 correos no leídos viejos, el primer trigger te traerá una avalancha. Para el workshop te recomiendo:
> - Marca como leídos casi todos
> - Deja solo 2-3 correos no leídos como prueba
> - O agrega un filtro adicional: `From: tu_email@ejemplo.com` para solo procesar uno específico

## Nodo 2 · AI Agent (el clasificador)

Aquí es donde vive la inteligencia. Vamos a darle al LLM cada correo recibido y pedirle que lo clasifique.

### Agregar el nodo

1. Conecta el output del Gmail Trigger a un nuevo nodo.
2. Busca **AI Agent**.
3. Selecciona el nodo **AI Agent** (algunos lo llaman **AI: Agent**).

### Configurar

#### Conectar el modelo

El nodo AI Agent tiene un sub-conector llamado **Model**. Conéctale un nodo **Anthropic Chat Model**:

- **Credential**: la de Anthropic que creaste
- **Model**: `claude-opus-4-5` o el modelo más reciente disponible
- **Temperature**: `0.3` (queremos respuestas consistentes, no creativas)

#### Configurar el prompt del agente

En el nodo **AI Agent**, busca el campo de **System Message** o **System Prompt** y pega esto:

```
Eres un asistente que clasifica correos electrónicos.

Para cada correo recibido, debes:
1. Leer el remitente, asunto y cuerpo
2. Clasificarlo en una de tres categorías: URGENTE, NORMAL, SPAM
3. Si es URGENTE, generar un título corto para una tarea (máx 80 caracteres)
4. Si es URGENTE, generar un resumen de 1-2 frases del por qué

Criterios:
- URGENTE: pide acción específica, tiene fecha límite cercana, viene de jefe directo, problema crítico
- NORMAL: comunicación de trabajo regular, actualizaciones, recordatorios
- SPAM: newsletters, promos, notificaciones automáticas no críticas

Responde SIEMPRE en formato JSON estricto, sin texto adicional:
{
  "clasificacion": "URGENTE" | "NORMAL" | "SPAM",
  "titulo_tarea": "string o null",
  "resumen": "string o null",
  "razon": "string explicando por qué clasificaste así"
}
```

#### Configurar el user message

El **User Message** es lo que le pasamos en cada ejecución. Aquí mapeamos los datos del correo:

```
Clasifica este correo:

De: {{ $json.from.value[0].address }}
Asunto: {{ $json.subject }}
Cuerpo: {{ $json.snippet }}
```

> 📌 **Nota**
>
> Las llaves dobles `{{ }}` son la sintaxis de n8n para insertar datos del nodo anterior. La sintaxis exacta de los campos (`from.value[0].address`, `subject`, `snippet`) sale del output del Gmail Trigger. Si tu output tiene nombres distintos, ajústalos.

### Probar

Ejecuta el AI Agent con un correo de muestra. Deberías ver una respuesta JSON parecida a:

```json
{
  "clasificacion": "URGENTE",
  "titulo_tarea": "Responder propuesta de María antes del viernes",
  "resumen": "María necesita la propuesta firmada antes del viernes para cerrar el cliente.",
  "razon": "Solicitud específica con fecha límite cercana de un cliente importante."
}
```

Si el LLM responde con texto en vez de JSON puro, ajusta el prompt enfatizando "SIEMPRE responde solo el JSON, sin markdown ni explicaciones adicionales".

## Nodo 3 · Code (parsear el JSON)

El AI Agent devuelve un string JSON. Para que los nodos siguientes puedan usar los campos individuales, hay que parsearlo.

### Agregar el nodo

1. Conecta el output del AI Agent a un nuevo nodo.
2. Busca **Code** (a veces llamado **Function**).
3. Selecciona el lenguaje: **JavaScript**.

### Configurar

Pega este código:

```javascript
// Toma la respuesta del AI Agent y la parsea como JSON
const respuestaTexto = $input.first().json.output || $input.first().json.text;

let parsed;
try {
  parsed = JSON.parse(respuestaTexto);
} catch (e) {
  // Si el LLM devolvió markdown ```json ... ```, limpiar
  const limpio = respuestaTexto
    .replace(/```json\s*/g, '')
    .replace(/```\s*$/g, '')
    .trim();
  parsed = JSON.parse(limpio);
}

return [{
  json: {
    ...parsed,
    correo_original: $('Gmail Trigger').first().json
  }
}];
```

> 📌 **Nota**
>
> `$('Gmail Trigger')` te permite acceder a datos de cualquier nodo previo del workflow por nombre. Si renombraste el nodo Gmail Trigger, ajusta el nombre.

## Nodo 4 · Switch (ramificar por urgencia)

Ahora separamos el flujo según la clasificación.

### Agregar el nodo

1. Conecta el output del Code a un nuevo nodo **Switch**.
2. Configura las rutas:

#### Modo: Rules

| Output | Condición |
|--------|-----------|
| 0 (URGENTE) | `{{ $json.clasificacion }}` equals `URGENTE` |
| 1 (NORMAL)  | `{{ $json.clasificacion }}` equals `NORMAL` |
| 2 (SPAM)    | `{{ $json.clasificacion }}` equals `SPAM` |

Cada salida del Switch (`0`, `1`, `2`) ahora tiene su propio camino. Vamos a llenar solo la salida `0` (URGENTE) para crear la tarea y notificar.

## Nodo 5 · Notion (crear tarea)

### Agregar el nodo

Conecta la **salida 0 (URGENTE)** del Switch a un nuevo nodo **Notion**.

### Configurar

- **Credential**: la de Notion que creaste
- **Resource**: `Database Page`
- **Operation**: `Create`
- **Database**: selecciona tu base de datos (la que compartiste con la integración)
- **Properties**: mapea los campos de tu DB. Mínimamente:

| Campo Notion | Valor |
|--------------|-------|
| `Name` (o `Title`) | `{{ $json.titulo_tarea }}` |
| `Status` | `Pendiente` (o el valor que uses) |
| `Description` | `{{ $json.resumen }}` |
| `Source` | `Email - {{ $json.correo_original.from.value[0].address }}` |

> 💡 **Tip**
>
> La estructura exacta de tu DB de Notion puede variar. Ajusta los nombres de los campos a los que tienes en tu DB.

## Nodo 6 · Slack (notificar) — opcional

### Agregar el nodo

Conecta el output del nodo Notion a un nuevo nodo **Slack**.

### Configurar

- **Credential**: la de Slack que creaste
- **Resource**: `Message`
- **Operation**: `Send`
- **Channel**: `#notifications` o el canal que prefieras
- **Text**:

```
🔴 Nueva tarea urgente creada

*{{ $('Code').first().json.titulo_tarea }}*

📧 De: {{ $('Code').first().json.correo_original.from.value[0].address }}
📝 {{ $('Code').first().json.resumen }}

Razón: {{ $('Code').first().json.razon }}
```

## Probar el workflow completo

### Ejecución manual

1. En la parte superior derecha del editor, busca el botón **Test workflow** o **Execute workflow**.
2. Si tienes correos no leídos en tu inbox, vas a ver el flujo ejecutarse paso a paso.
3. Cada nodo se pone verde si tuvo éxito, rojo si falló.

### Verificar resultados

- Ve a tu Notion: ¿se creó la tarea?
- Ve a tu Slack: ¿llegó el mensaje?
- Si algo falla, **haz clic en el nodo rojo** para ver el error específico.

## Activar el workflow

Hasta ahora has estado en modo "edit / test". Para que el workflow corra automáticamente cada 15 minutos:

1. En la parte superior del editor, hay un toggle **Inactive / Active**.
2. Cámbialo a **Active**.

A partir de ahora, n8n revisa tu inbox cada 15 minutos y procesa los correos no leídos automáticamente.

> ⚠️ **Cuidado**
>
> Antes de activar, **revisa con un correo real** que el flujo hace lo correcto. Si por error el AI Agent clasifica todo como URGENTE, vas a llenar tu Notion de tareas falsas en pocas horas.

## Estructura final

Tu workflow ahora se ve así:

```
[Gmail Trigger] → [AI Agent] → [Code (parse JSON)] → [Switch by clasificacion]
                                                            ├─ URGENTE → [Notion] → [Slack]
                                                            ├─ NORMAL  → (vacío)
                                                            └─ SPAM    → (vacío)
```

Cinco nodos. Un agente real funcionando. Cero líneas de código (excepto el helper de parsing).

---

## Lo que tienes ahora

- ✅ Workflow completo de clasificación de correos
- ✅ AI Agent decidiendo en runtime qué hacer
- ✅ Notion recibiendo tareas urgentes
- ✅ Slack notificándote (opcionalmente)
- ✅ Workflow activo corriendo cada 15 minutos

> 🎯 **Reto**
>
> Mejora el agente: agrega una salida adicional al Switch para una nueva categoría como `FACTURA` (correos con facturas pendientes). Hazlo crear una tarea en una base de datos distinta de Notion.

**Siguiente: [Troubleshooting →](04-troubleshooting.md)**
