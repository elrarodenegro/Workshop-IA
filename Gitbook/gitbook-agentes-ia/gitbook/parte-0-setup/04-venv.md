# 0.4 · Entornos virtuales (venv)

## El problema que resuelven

Imagina que tienes dos proyectos:

- **Proyecto A** usa la versión 1.0 de la librería `anthropic`
- **Proyecto B** necesita la versión 2.0 de la misma librería

Si instalas todo globalmente en tu sistema, instalar la 2.0 rompe el Proyecto A. Si vuelves a la 1.0, rompes el B. Es un infierno.

**Los entornos virtuales** (venv) resuelven esto creando una "burbuja" aislada de Python por cada proyecto. Cada burbuja tiene sus propias librerías sin interferir con las demás.

> 📌 **Nota**
>
> Esto es la práctica estándar en Python. **Nunca instales librerías globalmente con `pip install` directo.** Siempre dentro de un venv.

## Cómo funciona, en una analogía

Piensa en cada venv como un cuarto separado en tu casa:

- Cada cuarto tiene su propio escritorio (Python)
- Su propia caja de herramientas (las librerías que instalas)
- Lo que hagas en un cuarto no afecta los demás
- Cuando entras al cuarto, "activas" sus herramientas
- Cuando sales, vuelves a la herramienta general de la casa

## Crear tu primer venv

Vamos a crear el entorno virtual para el workshop. Asegúrate de estar en la carpeta `agentes-workshop` (verifica con `pwd`).

```bash
$ python -m venv venv
```

Desglose de este comando:

- `python -m venv` — ejecuta el módulo `venv` de Python
- `venv` (al final) — el nombre de la carpeta donde se creará el entorno. Por convención se llama `venv` (también podrías llamarla `.venv`, `env`, etc.)

Después de unos segundos verás una nueva carpeta `venv/` en tu workspace. **No la abras** — está llena de archivos del sistema. Solo confía en que está ahí.

> ⚠️ **Cuidado**
>
> Si te dice algo como "the virtual environment was not created successfully because ensurepip is not available", instala el paquete que falta:
>
> - **Ubuntu/Debian**: `sudo apt install python3-venv`
> - **macOS**: ya viene incluido. Si falla, reinstala Python con Homebrew.

## Activar el venv

Crear el venv no es lo mismo que estar dentro de él. Hay que **activarlo** en cada sesión nueva de terminal:

### macOS / Linux

```bash
$ source venv/bin/activate
```

### Windows (PowerShell)

```bash
$ venv\Scripts\Activate.ps1
```

### Windows (CMD)

```bash
$ venv\Scripts\activate.bat
```

> ⚠️ **Cuidado en Windows**
>
> Si PowerShell te dice algo como "running scripts is disabled on this system", ejecuta esto **una sola vez** (en PowerShell como administrador):
>
> ```bash
> $ Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```
>
> Es una restricción de seguridad por defecto de Windows.

## Cómo saber si está activado

Cuando un venv está activo, tu prompt de terminal cambia y muestra el nombre del entorno entre paréntesis al inicio:

```bash
(venv) $
```

Si ves `(venv)` al inicio de tu línea, **estás dentro del entorno virtual**. Cualquier `pip install` que hagas se queda aislado en esa carpeta.

Para verificar más explícitamente qué Python está activo:

```bash
(venv) $ which python
/Users/cristian/agentes-workshop/venv/bin/python
```

(En Windows: `where python`)

Debería apuntar a la carpeta `venv` de tu proyecto, no a la instalación global del sistema.

## Instalar tu primer paquete

Con el venv activo, vamos a instalar la librería de Anthropic como prueba:

```bash
(venv) $ pip install anthropic
```

Vas a ver un montón de líneas pasando — está descargando e instalando el paquete y sus dependencias. Al final:

```
Successfully installed anthropic-0.x.x ...
```

Verifica que se instaló:

```bash
(venv) $ pip list
```

Debería aparecer `anthropic` en la lista. **Esa instalación solo existe dentro del venv.** Si desactivas el venv, `pip list` no la mostrará.

## Desactivar el venv

Cuando termines de trabajar y quieras "salir del cuarto":

```bash
(venv) $ deactivate
$
```

El `(venv)` desaparece. Ya no estás en el entorno virtual.

## El archivo `requirements.txt`

Para que tu proyecto sea reproducible (otra persona pueda instalar lo mismo), Python usa un archivo de texto que lista todas las dependencias.

### Generar requirements.txt

Después de instalar todo lo que necesitas, dentro del venv:

```bash
(venv) $ pip freeze > requirements.txt
```

El comando `pip freeze` lista todos los paquetes instalados con su versión exacta. El `>` lo redirige a un archivo.

### Instalar desde requirements.txt

Cuando alguien (o tú mismo, en otra máquina) clona tu proyecto:

```bash
(venv) $ pip install -r requirements.txt
```

Le instala todas las dependencias de un solo golpe.

## Tu requirements.txt para el workshop

Para los Casos 2 y 3 vas a necesitar tres librerías. Vamos a crear el archivo ya.

En VS Code, crea un archivo nuevo en la raíz del workspace llamado `requirements.txt` con este contenido:

```text
anthropic
openai
google-genai
python-dotenv>=1.0.0
```

Después, con el venv activado, instala todo:

```bash
(venv) $ pip install -r requirements.txt
```

Esto va a tardar uno o dos minutos. Cuando termine, tendrás listas las tres APIs y la utilidad para manejar variables de entorno (`python-dotenv`).

## Buenas prácticas

### ✅ Hazlo

- **Un venv por proyecto.** No reuses el mismo venv para varios proyectos.
- **Agrega `venv/` a `.gitignore`** si vas a usar Git. El venv se reconstruye con `pip install -r requirements.txt`, no necesita versionarse.
- **Activa el venv siempre** antes de instalar o ejecutar código del proyecto.

### ❌ No lo hagas

- No copies la carpeta `venv/` entre máquinas. Las rutas son absolutas y se rompe.
- No instales con `sudo pip install` (ni siquiera fuera del venv).
- No te olvides de activar el venv y luego te preguntes por qué "el paquete no se encuentra".

## Resolver problemas

### "ModuleNotFoundError: No module named 'anthropic'"

Casi siempre significa que **el venv no está activado**. Mira tu prompt: ¿ves `(venv)` al inicio? Si no, actívalo.

### "El paquete se instaló pero VS Code no lo reconoce"

VS Code usa un Python específico. Asegúrate de que esté apuntando al del venv:

1. `Ctrl+Shift+P` (Windows/Linux) o `Cmd+Shift+P` (macOS)
2. Escribe `Python: Select Interpreter`
3. Selecciona el que esté en `./venv/bin/python` o `.\venv\Scripts\python.exe`

Después reinicia la terminal de VS Code.

### El venv se rompió

Si algo salió mal y quieres empezar de cero:

```bash
$ deactivate         # por si está activo
$ rm -rf venv        # macOS/Linux
$ python -m venv venv  # crearlo de nuevo
```

En Windows, borra la carpeta `venv` desde el explorador de archivos en lugar de `rm -rf`.

---

## Lo que tienes ahora

- ✅ Un entorno virtual `venv` creado en tu workspace
- ✅ Las librerías de Anthropic, OpenAI, Google y python-dotenv instaladas
- ✅ Un `requirements.txt` que documenta tus dependencias
- ✅ Sabes activar y desactivar el venv

**Siguiente: [0.5 · Crear tus API keys →](05-api-keys.md)**
