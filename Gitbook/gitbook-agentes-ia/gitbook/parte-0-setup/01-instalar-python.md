# 0.1 · Instalar Python

## ¿Por qué Python?

Para los Casos 2 y 3 del workshop vamos a escribir código. Python es el lenguaje estándar para trabajar con APIs de IA porque:

- Es legible (parece pseudocódigo en inglés)
- Todas las APIs tienen librerías oficiales en Python
- La comunidad de IA escribe casi todo en Python primero

> 📌 **Nota**
>
> Si vas a hacer **solo el Caso 1** (n8n, sin código), puedes saltarte esta parte. Pero si quieres el workshop completo, Python es obligatorio.

## Verificar si ya lo tienes

Antes de instalar nada, verifica si Python ya está en tu sistema:

```bash
$ python --version
```

o también:

```bash
$ python3 --version
```

Si te responde `Python 3.10.x` o superior (3.11, 3.12, 3.13), **ya estás listo, salta al siguiente capítulo.**

Si te dice "comando no encontrado", "command not found" o te da una versión menor a 3.10, sigue las instrucciones de tu sistema operativo abajo.

---

## Windows

### Paso 1 · Descargar el instalador

Ve a [python.org/downloads](https://www.python.org/downloads/) y descarga la última versión estable (Python 3.12+).

### Paso 2 · Ejecutar el instalador

Al abrirlo, **lo más importante**: marca la casilla **"Add Python to PATH"** abajo del instalador antes de hacer clic en "Install Now".

> ⚠️ **Cuidado**
>
> Si olvidas marcar "Add Python to PATH", Windows no va a encontrar `python` cuando lo escribas en la terminal. Si ya instalaste sin marcarla, desinstala y vuelve a empezar — es más fácil que arreglarlo manualmente.

Después haz clic en **"Install Now"** y espera a que termine.

### Paso 3 · Verificar

Abre una **nueva** ventana de Command Prompt (CMD) o PowerShell y escribe:

```bash
$ python --version
```

Deberías ver `Python 3.12.x` o similar.

---

## macOS

### Opción A · Con Homebrew (recomendada)

Si no tienes Homebrew, instálalo primero abriendo Terminal y pegando:

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Luego instala Python:

```bash
$ brew install python@3.12
```

### Opción B · Con instalador

Ve a [python.org/downloads](https://www.python.org/downloads/), descarga el `.pkg` de la última versión y ejecútalo.

### Verificar

Abre Terminal y escribe:

```bash
$ python3 --version
```

> 📌 **Nota**
>
> En macOS suele haber un `python` antiguo (versión 2.7) que viene con el sistema. Por eso usamos `python3` para asegurarnos de invocar la versión nueva. En Windows con la instalación nueva, `python` ya apunta directamente a Python 3.

---

## Linux (Ubuntu/Debian)

Casi todas las distribuciones modernas vienen con Python 3 ya instalado. Verifica:

```bash
$ python3 --version
```

Si la versión es menor a 3.10, actualízala:

```bash
$ sudo apt update
$ sudo apt install python3.12 python3.12-venv python3-pip
```

Para Fedora:

```bash
$ sudo dnf install python3.12 python3-pip
```

---

## Verificar `pip`

`pip` es el gestor de paquetes de Python. Lo vamos a usar para instalar las librerías de Anthropic, OpenAI y Google. Verifica que esté funcionando:

```bash
$ pip --version
```

o:

```bash
$ pip3 --version
```

Deberías ver algo como `pip 24.0 from /usr/...`.

> 💡 **Tip**
>
> Si `pip` no funciona pero `python` sí, prueba con:
>
> ```bash
> $ python -m pip --version
> ```

---

## Resolver problemas comunes

### "python: command not found" después de instalar (Windows)

- Cierra y vuelve a abrir la terminal — el PATH se carga al iniciar la sesión.
- Si aún no funciona, reinstala marcando "Add Python to PATH".

### "permission denied" al instalar paquetes (macOS/Linux)

- **No uses `sudo pip install`.** Eso instala globalmente y puede romper tu sistema.
- En vez de eso, vamos a usar entornos virtuales (capítulo **0.4**) que evitan completamente este problema.

### Tengo Python 3.8 pero no quiero borrarlo

Mantén el viejo, instala el nuevo en paralelo. Cuando ejecutes los comandos del workshop, usa explícitamente `python3.12` o `python3.11` en vez de `python3` para forzar la versión nueva.

---

## Lo que tienes ahora

- ✅ Python 3.10+ instalado
- ✅ `pip` funcionando

**Siguiente: [0.2 · Editor de código (VS Code) →](02-editor-vscode.md)**
