# Apéndice D: FAQ y errores frecuentes

Esta sección recopila los errores que más vas a encontrar mientras trabajas con agentes. Está organizada por tipo de error: copia el mensaje exacto que te aparece en consola y busca aquí.

---

## Errores de Python y entorno

### `ModuleNotFoundError: No module named 'anthropic'`

**Causa**: instalaste el paquete fuera del entorno virtual, o no activaste el venv antes de correr el script.

**Solución**:

```bash
# 1. Activa el venv (verás (venv) al inicio del prompt)
$ source venv/bin/activate          # macOS/Linux
$ venv\Scripts\activate             # Windows

# 2. Reinstala dentro del venv
$ pip install anthropic openai google-generativeai python-dotenv
```

**Cómo evitarlo**: siempre verifica que veas `(venv)` al inicio de tu prompt antes de instalar nada.

---

### `command not found: python` o `python: command not found`

**Causa**: en macOS y Linux modernos, el binario se llama `python3`, no `python`.

**Solución**:

```bash
$ python3 --version
$ python3 -m venv venv
```

Si quieres usar `python` a secas, crea un alias en tu `~/.zshrc` o `~/.bashrc`:

```bash
alias python=python3
alias pip=pip3
```

---

### `pip: command not found`

**Causa**: igual que el anterior. Usa `pip3` o instala dentro de un venv (donde `pip` siempre funciona).

**Solución dentro de un venv** (recomendado):

```bash
$ source venv/bin/activate
$ pip install anthropic     # ahora sí funciona
```

---

### `externally-managed-environment` (Ubuntu 23.04+, Debian 12+)

**Mensaje**:

```
error: externally-managed-environment
× This environment is externally managed
```

**Causa**: Python del sistema no permite instalar paquetes globales para no romper el SO.

**Solución**: usa un venv. **Siempre.** No uses `--break-system-packages` salvo que sepas exactamente lo que haces.

```bash
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install anthropic
```

---

## Errores de API (Claude, OpenAI, Gemini)

### `AuthenticationError: invalid x-api-key`

**Causa**: la API key no se cargó, está mal copiada, o expiró.

**Diagnóstico paso a paso**:

```python
import os
from dotenv import load_dotenv

load_dotenv()
key = os.getenv("ANTHROPIC_API_KEY")

print(f"Key cargada: {key is not None}")
print(f"Empieza con: {key[:10] if key else 'NADA'}...")
print(f"Longitud: {len(key) if key else 0}")
```

**Checklist**:

- ¿Tu archivo se llama exactamente `.env` (con el punto)?
- ¿Está en el mismo directorio que tu script?
- ¿La línea es `ANTHROPIC_API_KEY=sk-ant-...` sin espacios alrededor del `=`?
- ¿La key empieza con `sk-ant-` (Claude), `sk-` (OpenAI), o es una key de Gemini?
- ¿Tu archivo `.gitignore` incluye `.env`? Si subiste la key a GitHub por error, **revócala inmediatamente** en el dashboard del proveedor.

---

### `RateLimitError: 429 Too Many Requests`

**Causa**: superaste el límite de requests por minuto, o tu cuenta no tiene crédito.

**Solución inmediata**: espera 60 segundos y reintenta.

**Solución a mediano plazo**: agrega reintentos con espera exponencial.

```python
import time
from anthropic import Anthropic, RateLimitError

client = Anthropic()

def llamar_con_reintento(messages, max_intentos=5):
    for intento in range(max_intentos):
        try:
            return client.messages.create(
                model="claude-opus-4-5",
                max_tokens=1024,
                messages=messages
            )
        except RateLimitError:
            espera = 2 ** intento  # 1, 2, 4, 8, 16 segundos
            print(f"Rate limit. Esperando {espera}s...")
            time.sleep(espera)
    raise Exception("Demasiados reintentos")
```

**Solución a largo plazo**: revisa tu plan en el dashboard del proveedor y aumenta los límites.

---

### `Context too long` o `prompt is too long`

**Mensaje típico**:

```
BadRequestError: prompt is too long: 220000 tokens > 200000 maximum
```

**Causa**: tu agente acumuló demasiado historial. Cada turno guarda mensajes + tool_use + tool_result, y rápido llega al límite.

**Soluciones**:

1. **Limita el loop**: máximo 10 iteraciones (como hicimos en el Caso 2).
2. **Resume en lugar de acumular**: cada N turnos, pide al modelo un resumen y reemplaza los mensajes viejos.
3. **Filtra tool_results gigantes**: si una tool devuelve 50 KB de JSON, recórtalo antes de pasarlo al modelo.

```python
def resumir_historial(messages, client):
    """Si el historial es muy largo, lo resume."""
    if len(messages) < 20:
        return messages

    resumen = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"Resume esta conversación en 3 párrafos:\n{messages}"
        }]
    )

    return [{
        "role": "user",
        "content": f"[Resumen de contexto previo]: {resumen.content[0].text}"
    }]
```

---

### `Invalid tool schema` o `tools[0].input_schema: Field required`

**Causa**: tu definición de tool tiene un error de formato.

**Schema correcto para Claude**:

```python
{
    "name": "leer_correos",                           # obligatorio
    "description": "Lee correos del inbox",           # obligatorio
    "input_schema": {                                  # obligatorio (no "parameters")
        "type": "object",                             # obligatorio
        "properties": {                               # obligatorio (puede estar vacío)
            "limite": {
                "type": "integer",
                "description": "Número máximo de correos"
            }
        },
        "required": []                                # opcional
    }
}
```

**Errores comunes**:

- Usar `"parameters"` en lugar de `"input_schema"` (eso es OpenAI, no Claude).
- Olvidar `"type": "object"` dentro del schema.
- Definir un parámetro requerido que no está en `properties`.

---

### Diferencias entre proveedores en el schema de tools

Esta es la fuente del 80% de los errores cuando portas código entre modelos.

**Claude** (`anthropic`):

```python
tools = [{
    "name": "...",
    "description": "...",
    "input_schema": { "type": "object", "properties": {...} }
}]
```

**OpenAI** (`openai`):

```python
tools = [{
    "type": "function",
    "function": {
        "name": "...",
        "description": "...",
        "parameters": { "type": "object", "properties": {...} }
    }
}]
```

**Gemini** (`google-generativeai`):

```python
tools = [{
    "function_declarations": [{
        "name": "...",
        "description": "...",
        "parameters": { "type": "object", "properties": {...} }
    }]
}]
```

Si copias y pegas entre uno y otro, **siempre falla**. Adapta el schema o usa la versión del Caso 3 que ya tiene los tres formatos.

---

### El modelo no llama a la tool, solo responde texto

**Causa**: el system prompt no le dice claramente cuándo usar la herramienta, o la descripción de la tool es ambigua.

**Mal**:

```python
{
    "name": "buscar_cve",
    "description": "Busca info"   # ← demasiado vago
}
```

**Bien**:

```python
{
    "name": "buscar_cve",
    "description": "Busca información de severidad CVSS y descripción de una vulnerabilidad CVE en la base de datos NVD. Úsala SIEMPRE que el usuario mencione un identificador CVE como 'CVE-2024-12345'."
}
```

Y en el system prompt:

```python
system = """Eres un analista de vulnerabilidades.
Cuando el usuario mencione un CVE, DEBES usar la tool buscar_cve antes de responder.
NO inventes datos de severidad. Si no sabes, llama a la tool."""
```

---

### El loop nunca termina (`stop_reason` siempre es `tool_use`)

**Causa**: olvidaste agregar el `tool_result` al historial, o el modelo está atrapado pidiendo la misma tool una y otra vez.

**Checklist del loop**:

```python
while True:
    response = client.messages.create(...)

    # 1. Agrega la respuesta del modelo SIEMPRE
    messages.append({"role": "assistant", "content": response.content})

    # 2. Si terminó, sal del loop
    if response.stop_reason == "end_turn":
        break

    # 3. Si pidió tool, ejecuta y agrega resultado
    if response.stop_reason == "tool_use":
        for block in response.content:
            if block.type == "tool_use":
                resultado = ejecutar_tool(block.name, block.input)
                messages.append({
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": block.id,    # ← clave: el ID debe coincidir
                        "content": resultado
                    }]
                })
```

**Si aún se cicla**: pon un contador de seguridad.

```python
for _ in range(10):  # máximo 10 iteraciones
    response = client.messages.create(...)
    ...
```

---

## Errores de n8n (Caso 1)

### "Connection refused" o "ECONNREFUSED" cuando abro localhost:5678

**Causa**: n8n no está corriendo, o está en otro puerto.

**Solución**:

```bash
# Verifica que el contenedor esté arriba
$ docker ps | grep n8n

# Si no aparece, arráncalo
$ docker start n8n

# Si nunca lo creaste:
$ docker run -d --name n8n -p 5678:5678 \
    -v ~/.n8n:/home/node/.n8n \
    n8nio/n8n
```

---

### "OAuth token expired" en el nodo Gmail / Notion / Slack

**Causa**: el token de acceso caducó (suele pasar a las pocas horas).

**Solución**:

1. En n8n, abre el workflow.
2. Click en el nodo que falla → pestaña "Credentials".
3. Click en "Reconnect" o "Reauthorize".
4. Sigue el flujo OAuth en el navegador.

**Cómo evitarlo**: usa credenciales con `refresh_token`, que n8n renueva automáticamente. La mayoría de proveedores ya lo hacen por default.

---

### El AI Agent node responde "I don't have access to that information"

**Causa**: no le pasaste el contexto, o el system prompt es muy genérico.

**Solución**: en el campo "System Message" del nodo AI Agent, sé específico:

```
Eres un clasificador de correos. Recibes el asunto y el cuerpo de un email.

Devuelve SOLO un JSON válido con esta estructura:
{
  "categoria": "urgente" | "normal" | "spam",
  "razon": "explicación corta",
  "accion": "crear_tarea" | "ignorar"
}

NO agregues texto antes ni después del JSON.
NO uses markdown.
```

Y en el campo "Prompt" del usuario:

```
Asunto: {{ $json.subject }}
Cuerpo: {{ $json.snippet }}
```

---

### El Switch node no enruta correctamente

**Causa**: estás comparando el campo equivocado, o el JSON viene anidado.

**Diagnóstico**: agrega un nodo "Edit Fields" justo antes del Switch para ver qué llega.

**Solución**: si el AI Agent devuelve:

```json
{ "output": { "categoria": "urgente" } }
```

En el Switch, compara contra `{{ $json.output.categoria }}`, no `{{ $json.categoria }}`.

---

## Errores de Git y GitHub

### `fatal: not a git repository`

**Causa**: estás corriendo `git` fuera de un directorio inicializado.

**Solución**:

```bash
$ cd /ruta/a/tu/proyecto
$ git init
```

---

### Subiste tu `.env` a GitHub por error

**Esto es serio**. Tu API key está expuesta.

**Pasos urgentes**:

1. **Revoca la key inmediatamente** en el dashboard de Anthropic / OpenAI / Google.
2. Genera una key nueva.
3. Borra el archivo del repo:

```bash
$ git rm --cached .env
$ echo ".env" >> .gitignore
$ git add .gitignore
$ git commit -m "Remove .env and ignore it"
$ git push
```

4. Si quieres borrar el archivo del historial completo, usa `git filter-repo` o `BFG Repo-Cleaner`. Asume que cualquier key que estuvo en GitHub público **ya fue scrapeada por bots**, aunque la borres del historial.

---

## Errores conceptuales (no son bugs, son malentendidos)

### "El modelo me da respuestas distintas cada vez"

**No es un bug.** Los LLMs son no-determinísticos por defecto.

Si necesitas reproducibilidad (para tests, por ejemplo), baja la temperatura:

```python
client.messages.create(
    model="claude-opus-4-5",
    temperature=0,         # 0 = lo más determinístico posible
    max_tokens=1024,
    messages=[...]
)
```

Aun con `temperature=0`, puede haber pequeñas variaciones porque el hardware no es determinístico al 100%, pero serán mínimas.

---

### "Mi agente alucina datos que no existen"

**Causa común**: le pediste algo que no puede saber sin consultar una fuente, y no le diste tools.

**Solución 1**: dale acceso a la fuente real (RAG, API, base de datos).

**Solución 2**: en el system prompt, prohíbe inventar.

```
Si no tienes información verificada, responde literalmente:
"No tengo datos para responder esto. Necesitaría consultar [fuente]."

NUNCA inventes números, fechas, nombres o citas.
```

**Solución 3**: pide que cite la fuente de cada afirmación. Si no puede citar, no afirma.

---

### "El agente es lento"

**Causa**: cada llamada al LLM toma 2–10 segundos. Si tu loop hace 5 llamadas, son 25 segundos.

**Soluciones**:

1. Usa un modelo más rápido para tareas simples (`claude-haiku-4-5`, `gpt-4-mini`, `gemini-flash`).
2. Paraleliza llamadas independientes con `asyncio` o `concurrent.futures`.
3. Cachea respuestas para inputs repetidos (Redis, o un dict en memoria).
4. Reduce el `max_tokens` si las respuestas son cortas.

---

### "¿Por qué no uso ChatGPT en lugar de programar esto?"

Excelente pregunta. **Si tu caso lo resuelve un chat manual, no necesitas un agente.**

Construye un agente solo cuando:

- El proceso se ejecuta muchas veces (no una vez).
- Necesita correr sin que estés presente (cron, webhook, evento).
- Necesita integrarse con sistemas (APIs, bases de datos, Slack, etc.).
- Necesita auditoría o trazabilidad de cada paso.

Si nada de eso aplica: usa el chat. Es más rápido y más barato.

---

## Recapitulando

Los errores se repiten. Los más comunes en orden de frecuencia son:

1. Olvidar activar el venv → `ModuleNotFoundError`
2. Mal configurar el `.env` → `AuthenticationError`
3. Schema de tools incorrecto entre proveedores
4. Loop sin condición de salida
5. Subir el `.env` a GitHub (¡revoca la key!)

Si te aparece un error que no está aquí: **copia el mensaje exacto en Google o pregúntaselo a Claude**. El 99% de las veces alguien ya lo tuvo y lo resolvió.

---

Siguiente: [Mensaje final →](../cierre/mensaje-final.md)
