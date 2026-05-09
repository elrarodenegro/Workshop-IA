# 1.5 · Cuándo NO usar agentes

> *"La credibilidad técnica empieza por saber cuándo no necesitas la herramienta de moda."*

Este es probablemente el capítulo más importante del libro. Si solo te tomas en serio uno, que sea este.

## La regla de oro

Antes de empezar:

> **Si puedes resolverlo con un `if/else`, usa un `if/else`.**

Esa es la regla. Todo lo que sigue es elaboración de esa idea.

## Las seis señales de "NO USES AGENTE"

### 1 · La tarea es 100% determinista

Si para un input X siempre quieres un output Y, sin matices, sin excepciones, **un agente es un peor `if/else`**.

**Ejemplo que NO necesita agente**: convertir todos los archivos PDF de una carpeta a Word.
- Input siempre: PDF
- Output siempre: el mismo PDF en formato Word
- Cero ambigüedad

```python
# Esto es 100x más rápido y 1000x más barato
for archivo in carpeta:
    if archivo.endswith(".pdf"):
        convertir_a_word(archivo)
```

### 2 · Los pasos siempre son los mismos

Si el flujo es:
1. Recibir X
2. Hacer Y
3. Hacer Z
4. Devolver W

Y nunca cambia el orden ni se saltan pasos — **es un workflow tradicional**, no un agente.

**Ejemplo que NO necesita agente**: cuando llega un correo de cierto remitente, archivarlo y notificar en Slack.

Eso es Zapier. Eso es n8n sin nodo de IA. Eso es 5 líneas de código.

### 3 · El input está bien estructurado

Si los datos llegan en JSON, en filas de base de datos, en webhooks con esquema claro — **no necesitas un LLM para "entender" el input**. Ya lo entiendes.

**Ejemplo que NO necesita agente**: procesar transacciones de una API de pagos.
- Cada transacción tiene `id`, `monto`, `cliente_id`, `fecha`, `estado`
- Saber qué hacer con cada campo es un `if/else` de manual

### 4 · Necesitas latencia ultra-baja

Una llamada a un LLM tarda **segundos**, no milisegundos. Si tu proceso es:

- Una validación en tiempo real durante el checkout
- Una respuesta a un usuario en una UI conversacional síncrona
- Un endpoint que necesita responder bajo 100ms

**El LLM no es la herramienta correcta**. La latencia mata la experiencia.

> 📌 **Nota**
>
> Hay alternativas: usar el LLM para *pre-computar* el resultado (cachéalo) y solo servirlo en runtime. Pero eso ya no es "un agente en runtime" — es texto generado anticipadamente.

### 5 · El costo por ejecución importa mucho

Cada llamada al LLM tiene un costo. Si tu proceso se ejecuta:

- Millones de veces al día
- Por usuarios que pagan poco o nada
- En un margen de ganancia delgado

**Un agente puede destruir tu economía.** Calcula el costo unitario antes de comprometer arquitectura.

**Cálculo rápido**:
```
Costo por agente ≈ (tokens_input + tokens_output) × precio_por_token × iteraciones_promedio
```

Para Claude Opus, una iteración promedio puede costar entre $0.05 y $0.50 USD. Si tienes 100,000 ejecuciones al día, son entre $5,000 y $50,000 diarios. **Súmale ese costo a tu pricing antes de prometer features con agentes.**

### 6 · No toleras ningún margen de error

Los LLMs son **probabilísticos**. La misma pregunta puede dar respuestas distintas. Pueden alucinar (inventar datos). Pueden malinterpretar.

Si tu proceso es:

- Cálculos financieros (ej. impuestos, balances)
- Decisiones legales o regulatorias
- Operaciones médicas críticas
- Cualquier cosa donde un error tiene consecuencias graves

**Un agente sin validación humana es un riesgo serio.** Puedes usar LLMs como **sugerencia para un humano**, pero no como decisión autónoma.

## Las seis señales de "SÍ USA UN AGENTE"

Las inversas:

### 1 · El input es texto libre o ambiguo

Correos, mensajes de WhatsApp, descripciones de tickets, comentarios de redes sociales. Todo donde haya **lenguaje natural sin estructura**.

### 2 · Los pasos cambian según el contexto

Si la respuesta a "qué hacer ahora" depende de "qué pasó antes" o "qué dice el dato actual", el agente brilla.

### 3 · Hay que combinar múltiples herramientas

Cuando hay que decidir, según el caso, si llamar a A, a B, a A+B en cierto orden, o a ninguna — eso es razonamiento, no flujo.

### 4 · Hay que interpretar o resumir

Resumir un documento, extraer la idea principal de un email, traducir, reformular — todo eso requiere comprensión semántica que solo los LLMs hacen bien hoy.

### 5 · Hay que tomar decisiones con criterio

Aprobar/rechazar una solicitud según múltiples factores blandos. Recomendar el siguiente paso para un usuario. Priorizar tickets según urgencia subjetiva.

### 6 · El proceso requiere razonamiento

Tareas que un humano resolvería pensando: "primero verifico esto, luego aquello, y según el resultado decido el siguiente paso".

## Casos límite (zona gris)

Algunas situaciones se sienten ambiguas. Aquí va orientación práctica:

### "Tengo reglas pero hay excepciones"

→ **Combinación**: código tradicional para las reglas, agente solo para las excepciones.

### "El input es estructurado pero hay un campo de texto libre"

→ **Combinación**: código tradicional para los campos estructurados, LLM solo para procesar el campo de texto libre.

### "El volumen es alto pero ocasionalmente hay casos raros"

→ **Combinación + caché**: pre-procesa los casos comunes con código clásico (rapidez y barato). Manda solo el 5-10% raro al agente.

### "Quiero aprovecharme del LLM pero no quiero que decida"

→ **No es un agente, es un chatbot integrado.** Usa el LLM para tareas puntuales (clasificar, generar texto, resumir) dentro de un flujo definido por ti. Mucho más barato y predecible.

## El antipatrón clásico: "el agente que llama a una API"

Mucha gente construye esto:

```
Usuario pregunta → LLM → LLM llama a una sola API → LLM formatea respuesta → Usuario
```

Y le llaman "agente". **Eso no es un agente — es un wrapper caro de una API.**

Si en cualquier ejecución, el LLM siempre llama a la misma herramienta, en el mismo orden, con la misma lógica → estás pagando por el LLM cuando un script clásico haría lo mismo.

> 💡 **Tip**
>
> El test: si miras 100 ejecuciones de tu "agente" y todas siguen el mismo camino, **no es un agente, es overhead**. Refactoriza.

## El error opuesto: "todo es un script"

Hay personas que se vacunan tanto contra la moda de la IA que rechazan agentes incluso donde son la herramienta correcta. Si te encuentras gastando semanas escribiendo `if/else` para casos cada vez más raros que el usuario describe en lenguaje natural — **probablemente sí necesitas un agente**.

La señal: **tu lógica tradicional empieza a sentirse como una imitación pobre de razonamiento humano**. Cuando ese es el caso, mejor pagar el costo del LLM y ahorrarte el código.

## Resumen visual

```
INPUT TIENE ESTRUCTURA?
    SÍ ──┐
         ├── PASOS SON FIJOS?
         │      SÍ → Código tradicional ✅
         │      NO → Combinación
         │
    NO ──┴── REQUIERE INTERPRETACIÓN?
                SÍ → Agente ✅
                NO → Revisa otra vez (probablemente código)

LATENCIA CRÍTICA?
    SÍ → No uses agente síncrono. Considera batch + caché.

VOLUMEN ALTO + MARGEN AJUSTADO?
    SÍ → Calcula costo unitario antes de comprometer arquitectura.

ERRORES INACEPTABLES?
    SÍ → Agente con validación humana, NUNCA autónomo.
```

## Recapitulando

- La regla de oro: **si puedes resolverlo con `if/else`, usa `if/else`**
- 6 señales de NO usar agente: tarea determinista, pasos fijos, input estructurado, latencia crítica, costo crítico, cero tolerancia al error
- 6 señales de SÍ usar: input ambiguo, pasos contextuales, múltiples tools, interpretación, criterio, razonamiento
- **El patrón ganador casi siempre es combinar** ambos enfoques
- "Agente que siempre hace lo mismo" no es agente — es overhead

> 🎯 **Reto**
>
> Toma el primer "vamos a hacerlo con IA" que escuches en tu trabajo en los próximos 7 días y aplícale los 6 tests. Anota cuál fue el resultado y por qué.

---

**Has terminado la Parte 1.** Ahora viene la sección sobre **arquitecturas** — los 4 niveles de complejidad de agentes que vas a encontrar en producción.

**Siguiente: [2.1 · Nivel 1 · Tool Use simple →](../parte-2-arquitecturas/21-nivel-1-tool-use.md)**
