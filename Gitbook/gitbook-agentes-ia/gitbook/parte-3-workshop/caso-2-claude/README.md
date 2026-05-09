# Caso 2 · Python + Claude Tool Use

> *El mismo problema, ahora con código.*

## Qué vamos a construir

**El mismo agente del Caso 1**, pero implementado en Python desde cero usando la API de Anthropic con Tool Use.

¿Por qué hacer dos veces lo mismo? Porque construirlo en código te enseña lo que n8n te ocultaba detrás de su UI:

- **Cómo se define una tool** con JSON Schema
- **Cómo el LLM elige cuál usar** y con qué parámetros
- **Cómo funciona el loop** vuelta por vuelta
- **Qué pasa cuando algo falla** y cómo manejar errores

Si dominas esto, puedes construir cualquier agente Nivel 1 desde el primer principio — sin depender de plataformas low-code.

## Lo que aprenderás

Al terminar este caso podrás:

- ✅ Definir herramientas (tools) en formato Anthropic
- ✅ Implementar funciones reales que el LLM puede invocar
- ✅ Construir el loop completo de Tool Use
- ✅ Manejar `tool_use` y `tool_result` correctamente
- ✅ Identificar `stop_reason` para saber cuándo terminar
- ✅ Debuggear fallas comunes del loop

## Tiempo estimado

| Sección | Tiempo |
|---------|--------|
| Setup del proyecto | 5 min |
| Definir las herramientas | 10 min |
| Construir el loop | 15 min |
| Ejecutar y debug | 10 min |
| **Total** | **~40 min** |

## Stack

| Componente | Qué es |
|------------|--------|
| **Python 3.10+** | Lenguaje |
| **anthropic** | SDK oficial de Anthropic |
| **python-dotenv** | Cargar API keys desde `.env` |

Solo dos librerías. Mínimo viable real.

## Diferencia con el Caso 1

| Aspecto | Caso 1 (n8n) | Caso 2 (Python) |
|---------|--------------|-----------------|
| **Plataforma** | Visual, low-code | Código puro |
| **Loop** | Implícito en n8n | Tú lo escribes |
| **Tools** | Nodos pre-hechos | Funciones que tú defines |
| **Mocks** | Conexiones reales (Gmail, Notion) | Datos simulados (para enseñanza) |
| **Customización** | Limitada a lo que n8n soporta | Total |
| **Curva de aprendizaje** | Suave | Más empinada pero más profunda |

> 📌 **Nota sobre los mocks**
>
> En este caso vamos a **simular** las herramientas (`leer_correos`, `crear_tarea`) con datos en memoria. Conectarlas a Gmail y Notion reales agrega complejidad de OAuth y APIs que distrae del concepto central. Una vez entiendas el patrón, conectar a APIs reales es solo cuestión de cambiar las funciones internas.

## Estructura del código

Vamos a construir cinco partes:

```
agente.py
├── 1. Imports y setup
├── 2. Definición de tools (JSON Schema)
├── 3. Implementación de las tools (funciones Python)
├── 4. Loop principal del agente
└── 5. Entry point (main)
```

Total esperado: **~150 líneas de código bien comentadas**.

## Diagrama del loop

```
   ┌─────────────┐
   │ user_input  │
   └──────┬──────┘
          │
          ▼
   ┌─────────────────┐
   │ LLM razona      │ ◀──┐
   │ con tools=[...]│    │
   └──────┬──────────┘    │
          │               │
          ▼               │
    ¿stop_reason?         │
    │                     │
    ├─ "end_turn" ────────┼──→ FIN, devolver respuesta
    │                     │
    └─ "tool_use" ──┐     │
                    ▼     │
       ┌─────────────────┐│
       │ Ejecutar tool   ││
       │ (función Python)││
       └────────┬────────┘│
                │         │
                ▼         │
       ┌─────────────────┐│
       │ tool_result     ││
       │ → al LLM        ││
       └────────┬────────┘│
                │         │
                └─────────┘
```

## Estructura de este caso

1. **[Setup del proyecto](01-setup.md)** — preparar carpeta, venv, .env
2. **[Definir las herramientas](02-tools.md)** — JSON Schema y funciones mock
3. **[El loop de tool_use](03-loop.md)** — la lógica central, paso a paso
4. **[Código completo](04-codigo-completo.md)** — todo junto, comentado
5. **[Ejecutar y troubleshooting](05-ejecutar.md)** — correr y resolver problemas

## Prerrequisitos

Antes de seguir asegúrate de tener:

- ✅ Python 3.10+ instalado ([0.1](../../parte-0-setup/01-instalar-python.md))
- ✅ VS Code con extensión Python ([0.2](../../parte-0-setup/02-editor-vscode.md))
- ✅ venv activado en tu workspace ([0.4](../../parte-0-setup/04-venv.md))
- ✅ `.env` con `ANTHROPIC_API_KEY` ([0.5](../../parte-0-setup/05-api-keys.md))
- ✅ Librería `anthropic` instalada (`pip install anthropic`)

Si te falta alguno, vuelve atrás y resuélvelo. Sin estos, no puedes correr nada.

---

**Siguiente: [Setup del proyecto →](01-setup.md)**
