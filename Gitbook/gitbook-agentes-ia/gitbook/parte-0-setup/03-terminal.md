# 0.3 · Tu primera terminal

## ¿Qué es una terminal?

Es una ventana donde le hablas al computador con texto en vez de clicks. Suena anticuado pero es **mucho más rápido y preciso** que una interfaz gráfica para muchas tareas — especialmente para programar.

Cada vez que veas un bloque como este en el libro:

```bash
$ python --version
```

Significa: "abre tu terminal y escribe lo que viene después del `$`".

> 📌 **Nota**
>
> El `$` es solo una convención visual para indicar "esto es un comando". **No lo escribas.** Es como el sangrado en un libro — está ahí para mostrarte estructura, no para que lo copies.

## Las terminales en cada sistema

| Sistema | Terminal por defecto | Alternativas |
|---------|----------------------|--------------|
| **Windows** | PowerShell, Command Prompt (CMD) | Git Bash, Windows Terminal, Terminal de VS Code |
| **macOS** | Terminal.app | iTerm2, Terminal de VS Code |
| **Linux** | depende de la distro | gnome-terminal, kitty, alacritty, Terminal de VS Code |

Para este libro, **vamos a usar la terminal integrada de VS Code** que ya abriste en el capítulo anterior. Te ahorra cambiar de ventana.

## Los comandos imprescindibles

Estos siete comandos cubren el 95% de lo que necesitas en este workshop. Apréndelos y adquirirás superpoderes.

### `pwd` · saber dónde estás

```bash
$ pwd
/Users/cristian/agentes-workshop
```

`pwd` significa "print working directory". Te dice en qué carpeta está la terminal apuntando ahora mismo. Es el primer comando que escribes cuando algo no funciona — para confirmar que estás donde crees que estás.

> 📌 **Nota**
>
> En Windows con PowerShell, `pwd` también funciona. En CMD usa `cd` sin argumentos.

### `ls` · listar contenido

```bash
$ ls
hola.py    README.md    notas.txt
```

`ls` te muestra los archivos y carpetas en la ubicación actual.

> ⚠️ **Cuidado**
>
> En **CMD de Windows**, el comando es `dir`. En **PowerShell**, ambos `ls` y `dir` funcionan. En la terminal de VS Code en Windows generalmente es PowerShell, así que `ls` te va a funcionar.

### `cd` · cambiar de carpeta

```bash
$ cd parte-0-setup
$ pwd
/Users/cristian/agentes-workshop/parte-0-setup
```

`cd` significa "change directory". Para subir un nivel:

```bash
$ cd ..
```

Para ir al directorio home (tu carpeta de usuario):

```bash
$ cd ~
```

Para volver al último directorio donde estabas:

```bash
$ cd -
```

### `mkdir` · crear carpeta

```bash
$ mkdir caso-1
$ ls
caso-1    hola.py
```

### `touch` · crear archivo vacío

```bash
$ touch agente.py
```

> ⚠️ **Cuidado**
>
> En **Windows** (CMD/PowerShell) `touch` no existe. Usa esto en su lugar:
>
> ```bash
> $ New-Item agente.py
> ```
>
> O simplemente créalo desde VS Code: clic derecho en el explorador → "New File".

### `cat` · ver contenido de un archivo

```bash
$ cat hola.py
print("Hola, agente")
```

Útil cuando quieres ver rápido qué hay en un archivo sin abrirlo en el editor.

> 📌 **Nota**
>
> En CMD el equivalente es `type hola.py`. En PowerShell ambos funcionan.

### `python` · ejecutar un script Python

```bash
$ python hola.py
Hola, agente
```

El que ya conoces.

## Los atajos que te van a salvar la vida

### Tab · autocompletar

Empieza a escribir un nombre de archivo o carpeta y presiona `Tab`. La terminal lo completa.

```bash
$ python ho<TAB>
$ python hola.py
```

Si hay varias opciones que empiezan igual, presiona `Tab` dos veces para ver todas.

### Flecha arriba · comando anterior

Presiona la flecha hacia arriba para repetir el último comando. Sigue presionando para ir más atrás en el historial. Esencial para repetir `python script.py` sin volver a escribirlo cada vez.

### Ctrl+C · cancelar

Si un comando se queda colgado o ejecutaste algo que no querías, `Ctrl+C` lo aborta.

> 💡 **Tip**
>
> En la terminal, `Ctrl+C` **no copia** — eso lo hace `Ctrl+Shift+C` (Linux) o lo seleccionas con mouse en macOS/Windows.

### Ctrl+L (o `clear`) · limpiar pantalla

Cuando la terminal está llena de output viejo y quieres empezar limpio:

```bash
$ clear
```

O presiona `Ctrl+L` directamente.

## Tu primer ejercicio práctico

Vamos a navegar y crear estructura para el workshop. Abre la terminal de VS Code y haz lo siguiente paso a paso:

```bash
$ pwd
$ ls
$ mkdir caso-2-claude
$ cd caso-2-claude
$ touch agente.py
$ ls
$ cd ..
$ ls
```

Si todo salió bien, tienes:

- Una nueva carpeta `caso-2-claude` dentro de tu workspace
- Un archivo vacío `agente.py` dentro de ella
- Volviste al directorio raíz

> 🎯 **Reto**
>
> Crea otra carpeta llamada `caso-3-comparacion` con tres archivos vacíos: `claude.py`, `gpt.py` y `gemini.py`. Hazlo todo desde la terminal sin tocar el editor.

## Resolver problemas

### "command not found" en cualquier comando

Si dice esto en `python`, `pip`, etc., probablemente el sistema no tiene el comando en el PATH. Para Python ya cubrimos esto en el capítulo **0.1**. Para otros comandos, suele significar que falta instalar algo.

### Estoy perdido y no sé dónde estoy

```bash
$ pwd
$ cd ~
$ pwd
```

Esto te lleva a tu carpeta de usuario, un punto de partida confiable.

### Borré algo por error

Los comandos como `rm` (delete file) NO mandan a la papelera — borran permanentemente. **Por eso este libro evita enseñarte `rm`.** Si necesitas borrar algo, hazlo desde el explorador de VS Code (clic derecho → Delete) — eso sí va a la papelera.

---

## Lo que tienes ahora

- ✅ Sabes navegar entre carpetas con `cd`, `pwd`, `ls`
- ✅ Sabes crear archivos y carpetas
- ✅ Conoces los atajos esenciales (Tab, flecha arriba, Ctrl+C)
- ✅ Tu primer mini-proyecto creado desde terminal

**Siguiente: [0.4 · Entornos virtuales (venv) →](04-venv.md)**
