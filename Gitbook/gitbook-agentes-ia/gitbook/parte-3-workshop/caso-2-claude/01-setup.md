# Setup del proyecto

## Crear la estructura de carpetas

Vamos a tener una carpeta limpia para este caso, separada del resto.

Abre la terminal de VS Code (en tu workspace `agentes-workshop`):

```bash
$ mkdir caso-2-claude
$ cd caso-2-claude
$ touch agente.py
```

> 📌 **Nota Windows**
>
> Si `touch` no funciona, simplemente crea el archivo desde el explorador de VS Code: clic derecho en la carpeta `caso-2-claude` → New File → `agente.py`.

Ahora tu workspace luce así:

```
agentes-workshop/
├── venv/
├── .env
├── .gitignore
├── requirements.txt
├── caso-2-claude/
│   └── agente.py        ← aquí trabajamos
└── ...
```

## Activar el venv

Antes de cualquier comando de Python, **activa el venv**:

### macOS / Linux
```bash
$ source ../venv/bin/activate
```

### Windows
```bash
$ ..\venv\Scripts\activate
```

Verifica que aparezca `(venv)` al inicio de tu prompt.

> ⚠️ **Cuidado**
>
> El comando `source ../venv/bin/activate` usa `../` porque estás dentro de `caso-2-claude` y el venv está un nivel arriba. Si te confunde, vuelve al workspace raíz primero (`cd ..`), activa el venv allí, y luego entra a `caso-2-claude`.

## Verificar que las dependencias están

Con el venv activo:

```bash
(venv) $ pip list
```

Deberías ver al menos:

```
anthropic   0.40.x  (o más reciente)
python-dotenv  1.0.x
```

Si te falta alguna:

```bash
(venv) $ pip install anthropic python-dotenv
```

## Verificar que el .env carga

Ya tienes el `.env` en la raíz del workspace. Vamos a verificar que se puede leer desde la subcarpeta `caso-2-claude`.

Abre `agente.py` y pon esto temporalmente:

```python
# archivo: agente.py
import os
from dotenv import load_dotenv

# Cargar .env desde el directorio padre
load_dotenv("../.env")

api_key = os.getenv("ANTHROPIC_API_KEY")

if api_key:
    print(f"✅ Key cargada: {api_key[:8]}...{api_key[-4:]}")
else:
    print("❌ No se encontró ANTHROPIC_API_KEY")
```

Ejecuta:

```bash
(venv) $ python agente.py
```

Si todo va bien:

```
✅ Key cargada: sk-ant-x...xxxx
```

Si te dice "no se encontró", verifica:

1. Que el archivo `.env` esté en `agentes-workshop/.env` (no dentro de `caso-2-claude`)
2. Que el `.env` tenga la línea `ANTHROPIC_API_KEY=sk-ant-...` sin espacios ni comillas
3. Que estás ejecutando con el venv activo

## Hacer una llamada de prueba

Reemplaza el contenido de `agente.py` con esto:

```python
# archivo: agente.py
import os
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv("../.env")

client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=200,
    messages=[
        {"role": "user", "content": "Saluda en español como un agente de IA en una sola frase."}
    ]
)

print("Respuesta de Claude:")
print(response.content[0].text)
print(f"\nTokens usados: {response.usage.input_tokens} input, {response.usage.output_tokens} output")
```

Ejecuta:

```bash
(venv) $ python agente.py
```

Deberías ver una respuesta tipo:

```
Respuesta de Claude:
¡Hola! Soy un agente de IA listo para ayudarte con automatizaciones y tareas inteligentes.

Tokens usados: 19 input, 23 output
```

🎉 **Si llegaste aquí, todo el setup funciona.** Ya tienes Claude respondiendo desde tu propio código.

## Estructura para el resto del caso

A medida que avancemos, el archivo `agente.py` va a crecer. Vamos a estructurarlo en estas secciones:

```python
# 1. Imports
# 2. Setup del cliente
# 3. Definición de tools (JSON schemas)
# 4. Implementación de tools (funciones)
# 5. Loop principal
# 6. Entry point (main)
```

> 💡 **Tip**
>
> Para proyectos más grandes, separarías cada sección en archivos distintos (`tools.py`, `agent.py`, `main.py`). Para el workshop mantenemos todo en `agente.py` para que sea fácil de seguir.

## Errores comunes en este paso

### "ModuleNotFoundError: No module named 'anthropic'"

El venv no está activo. Mira tu prompt: ¿hay `(venv)` al inicio? Si no, actívalo de nuevo.

### "AuthenticationError"

La key de Anthropic está mal o no se cargó. Verifica el `.env`.

### El comando `python` ejecuta una versión vieja

Asegúrate de que el venv está activo. Cuando un venv está activo, `python` apunta al de la carpeta `venv/`, no al global.

Si en duda:

```bash
(venv) $ which python   # macOS/Linux
(venv) $ where python   # Windows
```

Debe apuntar a algo dentro de `agentes-workshop/venv/`.

---

## Lo que tienes ahora

- ✅ Carpeta `caso-2-claude/` creada
- ✅ `agente.py` con primera llamada exitosa a Claude
- ✅ venv activo y dependencias verificadas
- ✅ `.env` cargando correctamente desde la subcarpeta

**Siguiente: [Definir las herramientas →](02-tools.md)**
