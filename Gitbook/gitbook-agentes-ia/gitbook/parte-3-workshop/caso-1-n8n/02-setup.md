# Setup de n8n

## Crear cuenta en n8n.cloud

### Paso 1 · Registro

1. Ve a [n8n.cloud](https://n8n.cloud)
2. Haz clic en **Get started for free** (o **Sign up**)
3. Crea cuenta con tu email o con Google/GitHub OAuth (más rápido)
4. Verifica tu email si te lo pide

### Paso 2 · Crear workspace

n8n te pide un nombre para tu workspace al inicio. Llámalo algo como `mi-agente` o `workshop`. Esto te lleva al editor visual de n8n.

> 📌 **Nota**
>
> n8n.cloud free tier incluye 5,000 ejecuciones de workflow al mes. Suficiente para todo el workshop. Si después quieres usarlo intensamente, considera self-hostear (gratis sin límite).

### Paso 3 · Familiarízate con la interfaz

Vas a ver tres secciones principales:

- **Workflows** — donde viven tus automatizaciones
- **Credentials** — donde guardas las conexiones a servicios externos
- **Executions** — el historial de ejecuciones (útil para debug)

Empezamos por crear las credenciales — sin ellas no puedes conectar Gmail, Slack ni Notion.

## Conectar tus servicios

### Conectar Gmail

1. En el menú lateral, ve a **Credentials**.
2. Haz clic en **Add credential**.
3. Busca **Gmail** y selecciona **Gmail OAuth2 API**.
4. Sigue el flujo OAuth: te redirige a Google, das permiso, regresa a n8n.
5. Cuando termine, dale un nombre como `Gmail - mi cuenta`.
6. **Save**.

> ⚠️ **Cuidado**
>
> El primer OAuth con Google puede pedir verificación adicional o mostrar un warning de "app no verificada" si usas una cuenta personal. **Es normal** — n8n.cloud está pidiendo permisos en tu nombre. Si confías en n8n (que es opensource y conocido), continúa.

### Conectar Slack

1. **Add credential** → busca **Slack** → **Slack OAuth2 API**
2. Sigue el flujo: te lleva a Slack, eliges el workspace, das permisos.
3. Si tu workspace de Slack requiere aprobación de admin, vas a tener que pedirle a tu admin que apruebe la app de n8n.
4. Nombra la credencial como `Slack - mi workspace` y guarda.

> 📌 **Nota**
>
> Si no tienes un Slack al que conectarte, **no te preocupes** — puedes seguir el workshop usando solo Gmail + Notion. El paso de Slack es opcional. Más abajo te muestro cómo saltarlo.

### Conectar Notion

1. **Add credential** → busca **Notion** → **Notion API**
2. Notion no usa OAuth como Google/Slack. Usa un **Internal Integration Token** que tú generas.
3. Te explico cómo:

#### Generar el Notion Internal Integration Token

a. Ve a [notion.so/my-integrations](https://notion.so/my-integrations)
b. Haz clic en **+ New integration**
c. Dale un nombre (ej. `n8n-workshop`)
d. Selecciona el workspace correspondiente
e. Haz clic en **Submit**
f. Copia el **Internal Integration Token** que aparece (empieza con `secret_...`)

#### Pegar el token en n8n

Vuelve a n8n y pega el token en el campo correspondiente. Guarda con un nombre como `Notion - mi workspace`.

#### Compartir tu base de datos con la integración

Esto es CLAVE y mucha gente lo olvida:

1. Ve a tu Notion y abre la base de datos donde quieres que se creen las tareas (o crea una nueva).
2. En la esquina superior derecha de la base de datos, haz clic en los **tres puntos** (`...`).
3. Busca **Connections** o **Add connections**.
4. Selecciona la integración que creaste (`n8n-workshop`).

Si no haces este paso, n8n no puede ver tus bases de datos aunque tengas el token correcto.

> 💡 **Tip**
>
> En lugar de Notion, puedes usar **Trello, ClickUp, Asana o cualquier otra herramienta de tareas** que n8n soporte. La estructura del workflow es la misma — solo cambia el nodo final.

### Conectar tu LLM (Anthropic)

n8n tiene un nodo **AI Agent** que puede usar diferentes modelos. Vamos a configurarlo con tu API key de Anthropic (la que creaste en el [capítulo 0.5](../../parte-0-setup/05-api-keys.md)).

1. **Add credential** → busca **Anthropic**
2. Pega tu API key (la que empieza con `sk-ant-...`)
3. Nombra como `Anthropic - workshop` y guarda.

> 📌 **Nota**
>
> Si prefieres usar OpenAI o Google Gemini, n8n los soporta también. Solo crea la credencial correspondiente y luego en el nodo AI Agent eliges qué modelo usar.

## Verificar que todo está conectado

Ve a **Credentials**. Deberías tener al menos:

- ✅ Gmail OAuth2 API
- ✅ Notion API
- ✅ Anthropic
- ⚪ Slack OAuth2 API (opcional)

Si te falta alguna y no es opcional, vuelve atrás y resuélvela antes de seguir.

## Crear tu primer workflow

1. En el menú lateral, ve a **Workflows**.
2. Haz clic en **+ Add workflow** (botón en la esquina superior derecha).
3. Te abre un canvas vacío.
4. Dale un nombre arriba: `Agente de correos urgentes`.

Ahora tienes una pizarra en blanco. En el siguiente capítulo vamos a llenarla con nodos, paso a paso.

## Resolver problemas

### "OAuth callback failed" al conectar Gmail/Slack

- Revisa que tu navegador no esté bloqueando popups
- Intenta en una ventana incógnito
- Si persiste, prueba con otro navegador

### "Notion no me muestra mis bases de datos"

99% de las veces es que **olvidaste compartir la base de datos con la integración** (paso del "Connections" en Notion). Vuelve a hacerlo.

### "Mi credencial de Anthropic no funciona"

- Verifica que copiaste la key completa
- Asegúrate que tienes créditos en tu cuenta de Anthropic
- Verifica que la key no esté revocada (ve al panel de Anthropic)

### No puedo conectar Slack

Si tu admin no aprueba la app, puedes:

1. **Saltar el paso de Slack** — el workflow funciona sin él, solo no recibes notificaciones
2. **Usar email como notificación alternativa** — agrega un nodo Gmail al final que te envíe un correo a ti mismo
3. **Usar webhook a Discord, Telegram, etc.** — n8n soporta muchos canales

---

## Lo que tienes ahora

- ✅ Cuenta en n8n.cloud activa
- ✅ Credenciales para Gmail, Notion, Anthropic
- ✅ Opcionalmente: Slack
- ✅ Un workflow vacío llamado `Agente de correos urgentes`

**Siguiente: [Construcción paso a paso →](03-paso-a-paso.md)**
