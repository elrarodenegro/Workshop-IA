# 0.2 · Editor de código (VS Code)

## ¿Por qué VS Code?

Necesitas un editor de código para escribir Python. Podrías usar el Bloc de notas, pero VS Code te da:

- Resaltado de sintaxis (los colores que diferencian variables, funciones, comentarios)
- Autocompletado inteligente
- Detección de errores en vivo
- Terminal integrada (no tienes que cambiar entre ventanas)
- Extensiones específicas para Python

Y es gratis, multiplataforma y la mayoría de la comunidad lo usa — así que cuando busques ayuda online, los ejemplos asumirán VS Code.

> 📌 **Nota**
>
> Si ya estás cómodo con otro editor (PyCharm, Sublime, Vim, Cursor, Zed), úsalo. No necesitas cambiarte. Lo importante es tener UN editor donde te sientas productivo.

## Instalación

Ve a [code.visualstudio.com](https://code.visualstudio.com) y descarga la versión para tu sistema operativo. El instalador es estándar:

- **Windows**: ejecuta el `.exe` y dale "Siguiente" hasta el final
- **macOS**: arrastra la app a la carpeta Aplicaciones
- **Linux**: usa el `.deb`, `.rpm` o el comando de tu gestor de paquetes

Al abrirlo por primera vez verás una pantalla de bienvenida.

## Las extensiones que necesitas

Vas a instalar dos extensiones esenciales para Python.

### Cómo instalar extensiones

En VS Code:

1. Haz clic en el ícono de **Extensiones** en la barra lateral izquierda (parece cuatro cuadrados, uno separado), o presiona `Ctrl+Shift+X` (Windows/Linux) / `Cmd+Shift+X` (macOS).
2. Busca por nombre.
3. Haz clic en **Install**.

### Extensión 1 · Python (de Microsoft)

Busca **"Python"**. La que necesitas es la oficial de Microsoft (tiene un check azul de verificada y millones de descargas). Te da:

- Resaltado de sintaxis
- Autocompletado con Pylance
- Linting (te marca errores antes de ejecutar)
- Integración con entornos virtuales

### Extensión 2 · Pylance

Generalmente se instala automáticamente cuando instalas la extensión de Python. Si no, búscala e instálala manualmente. Es el motor de análisis de código que hace que el autocompletado sea inteligente.

### Bonus opcionales (recomendadas)

- **GitHub Copilot** — autocompletado con IA. Tiene tier gratuito limitado. Útil pero no esencial.
- **Better Comments** — colorea diferentes tipos de comentarios (`TODO`, `FIXME`, etc.).
- **indent-rainbow** — colorea la indentación, útil para Python (donde los espacios importan).

## Configuración inicial

### Tu primera carpeta de proyecto

Vamos a crear la carpeta donde vivirá todo tu trabajo del workshop:

1. Crea en tu computador una carpeta llamada `agentes-workshop` (donde quieras: Documentos, Escritorio, etc.).
2. Abre VS Code.
3. Ve a `Archivo` → `Abrir carpeta...` (en macOS: `File` → `Open Folder...`).
4. Selecciona la carpeta `agentes-workshop`.

VS Code va a abrirse con esa carpeta como tu workspace. Verás la estructura en la barra lateral izquierda (vacía por ahora).

### La terminal integrada

Esta es una de las features más útiles de VS Code. Para abrirla:

- **Atajo**: `Ctrl + ñ` (Windows/Linux) o `Ctrl + ` `` ` `` (acento grave) en algunos teclados
- **Menú**: `Terminal` → `New Terminal`

Aparecerá en la parte inferior de la ventana. Esta terminal está abierta automáticamente en la carpeta de tu proyecto, lo cual es una bendición — no tienes que navegar manualmente con `cd`.

## Tu primer archivo Python

Vamos a verificar que todo funcione con el "hola mundo" más aburrido del universo:

1. En VS Code, haz clic derecho en la barra lateral (donde está vacío) y selecciona **New File**.
2. Nómbralo `hola.py`.
3. Escribe esto adentro:

```python
print("Hola, agente")
```

4. Guarda con `Ctrl+S` (Windows/Linux) o `Cmd+S` (macOS).
5. Abre la terminal integrada (`Ctrl+ñ`) y escribe:

```bash
$ python hola.py
```

Si ves `Hola, agente` en la terminal, **todo está funcionando perfectamente**.

> 💡 **Tip**
>
> Si `python` no funciona pero `python3` sí, recuerda usar `python3 hola.py`. En el resto del libro pondré `python` para abreviar — usa el que te funcione.

## Resolver problemas

### VS Code no encuentra Python

Cuando abres un archivo `.py` por primera vez, VS Code suele preguntar qué intérprete de Python usar (aparece una notificación abajo a la derecha). Si no aparece:

1. Abre la paleta de comandos: `Ctrl+Shift+P` (Windows/Linux) / `Cmd+Shift+P` (macOS)
2. Escribe `Python: Select Interpreter`
3. Selecciona la versión que instalaste

### El terminal usa otro Python distinto al de VS Code

Esto es común y se resuelve con entornos virtuales — exactamente lo que vamos a ver en el capítulo **0.4**.

### Estoy en Windows y los comandos de bash no funcionan

VS Code en Windows abre PowerShell por defecto. La mayoría de comandos básicos funcionan igual (`python`, `pip`, `cd`, `ls` con un alias). Si quieres bash real, instala **Git for Windows** (te da Git Bash) y configúralo como terminal por defecto en VS Code.

---

## Lo que tienes ahora

- ✅ VS Code instalado
- ✅ Extensión Python + Pylance instaladas
- ✅ Una carpeta `agentes-workshop` abierta como workspace
- ✅ Tu primer `hola.py` ejecutándose

**Siguiente: [0.3 · Tu primera terminal →](03-terminal.md)**
