# 0.5 · Crear tus API keys

## ¿Qué es una API key?

Una **API key** es como una contraseña que te identifica cuando le hablas a un servicio en la nube. Cuando llamas a la API de Claude para que responda, la key es tu pase de entrada — el servicio sabe quién eres, qué plan tienes, y cuánto cobrarte.

> ⚠️ **REGLA DE ORO**
>
> **Una API key es una contraseña.** Si alguien la consigue, puede gastar tu dinero o usar tus créditos. **Nunca la pongas en código que vayas a publicar en GitHub o compartir.** En este capítulo te enseño la forma correcta de manejarlas.

## Las tres APIs que vamos a usar

Para el workshop completo necesitas keys de:

| Proveedor | API | Lo usamos en |
|-----------|-----|--------------|
| **Anthropic** | Claude | Caso 2 y Caso 3 |
| **OpenAI** | GPT-4 | Caso 3 |
| **Google AI Studio** | Gemini | Caso 3 |

Si solo vas a hacer el Caso 2, la única que necesitas es **Anthropic**.

## Anthropic (Claude)

### Crear cuenta y obtener la key

1. Ve a [console.anthropic.com](https://console.anthropic.com).
2. Crea una cuenta (o inicia sesión si ya tienes).
3. En la barra lateral izquierda, ve a **API Keys**.
4. Haz clic en **Create Key**.
5. Dale un nombre (ej. `workshop-agentes`).
6. Copia la key inmediatamente — la API key empieza con `sk-ant-...`

> ⚠️ **Cuidado**
>
> La key solo se muestra **una vez**. Si la pierdes, debes crear una nueva. Pégala temporalmente en un lugar seguro (un gestor de contraseñas, no un archivo de texto en tu escritorio).

### Configurar límites de gasto

Antes de seguir, configura un límite de gasto:

1. Ve a **Settings** → **Limits**.
2. Establece un límite mensual bajo para empezar (ej. $5 USD).
3. Activa alertas por email cuando se acerque al límite.

Esto te protege de cualquier error o uso accidental excesivo.

### Créditos iniciales

Anthropic suele dar **$5 USD de crédito gratis** al verificar tu cuenta. Más que suficiente para todo este workshop.

## OpenAI (GPT-4)

### Crear cuenta y obtener la key

1. Ve a [platform.openai.com](https://platform.openai.com).
2. Crea una cuenta o inicia sesión.
3. En la esquina superior derecha, haz clic en tu perfil y ve a **View API keys** o entra a **API Keys** en la barra lateral.
4. Haz clic en **Create new secret key**.
5. Dale un nombre y copia la key (empieza con `sk-...`).

### Configurar límites

1. Ve a **Settings** → **Limits**.
2. Configura un **Hard limit** (límite duro) bajo para empezar.

### Créditos

OpenAI suele dar créditos iniciales pero **expiran** en pocos meses. Verifica tu balance en **Usage**.

## Google AI Studio (Gemini)

### Crear cuenta y obtener la key

1. Ve a [aistudio.google.com](https://aistudio.google.com).
2. Inicia sesión con tu cuenta de Google.
3. En la barra lateral, busca **Get API key** o **API keys**.
4. Haz clic en **Create API key**.
5. Selecciona o crea un proyecto de Google Cloud (puedes usar el predeterminado).
6. Copia la key generada.

### Tier gratuito

Gemini tiene un **tier gratuito generoso** que cubre completamente cualquier uso de aprendizaje del workshop. No necesitas configurar facturación si solo vas a estar experimentando.

> 📌 **Nota**
>
> Las URLs y nombres exactos de los menús pueden cambiar — los proveedores actualizan sus consolas frecuentemente. Si no encuentras algo donde digo, busca "API key" en la barra lateral de cada plataforma. Está siempre en algún lugar visible.

## Guardar tus keys de forma segura

Aquí viene la parte importante. Vamos a guardar las keys en un archivo `.env` y **excluirlo de cualquier subida a Git**.

### Paso 1 · Crear el archivo .env

En la raíz de tu workspace `agentes-workshop`, crea un archivo llamado **`.env`** (sí, empieza con punto). Pega esto adentro y reemplaza con tus keys reales:

```bash
# archivo: .env
ANTHROPIC_API_KEY=sk-ant-tu-key-completa-aqui
OPENAI_API_KEY=sk-tu-key-completa-aqui
GEMINI_API_KEY=tu-key-de-gemini-aqui
```

> 💡 **Tip**
>
> No pongas comillas alrededor de las keys. No pongas espacios alrededor del `=`. El formato es estricto: `NOMBRE=valor`.

### Paso 2 · Crear .gitignore

Si vas a usar Git para versionar tu código (recomendado), debes asegurarte de **nunca subir el .env**. Crea un archivo llamado `.gitignore` en la raíz del workspace:

```text
# archivo: .gitignore
venv/
.env
__pycache__/
*.pyc
.DS_Store
```

> ⚠️ **Cuidado**
>
> Si por error subes una key a un repo público, **considera esa key comprometida inmediatamente**. Ve al panel del proveedor y revócala. Crea una nueva. Hay bots automatizados escaneando GitHub buscando keys filtradas y las usan en minutos.

### Paso 3 · Cargar el .env en Python

Ya instalaste `python-dotenv` en el capítulo anterior. Esta librería lee el archivo `.env` y carga sus valores como variables de entorno en tu programa.

Vamos a probarlo. Crea un archivo `verificar_keys.py` en la raíz:

```python
# archivo: verificar_keys.py
import os
from dotenv import load_dotenv

# Cargar las variables del archivo .env
load_dotenv()

# Verificar que las keys estén disponibles
keys = {
    "Anthropic": os.getenv("ANTHROPIC_API_KEY"),
    "OpenAI": os.getenv("OPENAI_API_KEY"),
    "Gemini": os.getenv("GEMINI_API_KEY"),
}

print("Estado de tus API keys:")
print("=" * 40)
for nombre, key in keys.items():
    if key:
        # Mostrar solo los primeros y últimos 4 caracteres por seguridad
        masked = f"{key[:6]}...{key[-4:]}"
        print(f"✅ {nombre}: {masked}")
    else:
        print(f"❌ {nombre}: NO CONFIGURADA")
```

Ejecuta:

```bash
(venv) $ python verificar_keys.py
```

Si todo está bien, vas a ver:

```
Estado de tus API keys:
========================================
✅ Anthropic: sk-ant...xxxx
✅ OpenAI: sk-pro...xxxx
✅ Gemini: AIzaSy...xxxx
```

> 🎯 **Reto**
>
> ¿Por qué crees que el script muestra solo los primeros y últimos caracteres en vez de la key completa? Pista: piensa en lo que pasa si compartes una captura de pantalla.

## Probar una llamada real

Ya tienes las keys cargadas. Hagamos la prueba mínima posible — pedirle a Claude que diga "hola":

```python
# archivo: prueba_claude.py
import os
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()

client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=100,
    messages=[
        {"role": "user", "content": "Saluda en una sola frase."}
    ]
)

print(response.content[0].text)
```

Ejecuta:

```bash
(venv) $ python prueba_claude.py
```

Si todo funciona, **Claude te responde directamente desde tu terminal**. Felicidades — acabas de hacer tu primera llamada a una API de IA.

> 📌 **Nota**
>
> El nombre exacto del modelo (`claude-opus-4-5`) puede cambiar con el tiempo cuando salen nuevas versiones. Si te da error de "model not found", revisa la documentación oficial de Anthropic para el modelo más reciente.

## Resolver problemas

### "AuthenticationError: invalid x-api-key"

La key está mal. Verifica:

1. Que copiaste la key completa (sin espacios al inicio o al final).
2. Que el `.env` no tiene comillas alrededor del valor.
3. Que el nombre de la variable en `.env` coincide con el que usas en `os.getenv()`.

### "El .env no se está cargando"

Verifica que estás ejecutando el script desde **la misma carpeta** donde está el `.env`. `load_dotenv()` busca el archivo en el directorio actual.

### "No hay créditos en mi cuenta"

Algunas plataformas requieren verificación adicional o agregar un método de pago para activar créditos. Revisa la sección de Billing/Usage de cada plataforma.

### "Mi región no está soportada"

Anthropic, OpenAI y Google tienen restricciones por país. Si vives en una región no soportada, considera:

1. Usar un proveedor que sí lo soporte
2. Empezar con Gemini (suele tener cobertura más amplia)
3. Para el Caso 1 del workshop no necesitas ninguna key — todo se hace en n8n

---

## Lo que tienes ahora

- ✅ API keys de Anthropic, OpenAI y Gemini (o al menos una de ellas)
- ✅ Archivo `.env` con tus keys de forma segura
- ✅ Archivo `.gitignore` configurado
- ✅ Capacidad de cargar las keys desde Python
- ✅ Tu primera llamada a una API de IA exitosa

**🎉 Has terminado el setup. Ahora viene la parte interesante.**

**Siguiente: [1.1 · LLM vs Chatbot vs Agente →](../parte-1-fundamentos/11-llm-chatbot-agente.md)**
