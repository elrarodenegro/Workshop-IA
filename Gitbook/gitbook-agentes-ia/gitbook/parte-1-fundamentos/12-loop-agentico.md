# 1.2 · El loop agéntico

## El corazón de cualquier agente

Si solo te llevas una idea de este libro, que sea esta: **todo agente, sin excepción, funciona en un loop de cuatro pasos**:

```
Percibir → Razonar → Actuar → Observar → (volver a Percibir)
```

Sin importar si es un agente low-code en n8n, un script de Python en 50 líneas, o un sistema multi-agente de producción con FastAPI y Celery — la lógica subyacente es la misma.

## El loop, paso a paso

### 1 · Percibir (Perceive)

El agente recibe input. Puede ser:

- Un mensaje del usuario ("agéndame una reunión con Pedro mañana")
- Un evento (un correo nuevo llegó)
- Un dato de un sistema (un ticket fue creado)
- El resultado de una herramienta que ejecutó antes

Esta entrada se suma al **contexto** que el agente tiene disponible.

### 2 · Razonar (Reason)

El LLM procesa el contexto y decide:

- ¿Tengo toda la información que necesito?
- ¿Cuál es el siguiente paso lógico?
- ¿Necesito invocar alguna herramienta?
- ¿Cuál herramienta y con qué parámetros?

Este paso es **donde vive la "inteligencia"** del agente. El LLM, al ver el contexto, propone una acción.

### 3 · Actuar (Act)

El agente ejecuta la decisión:

- Llama a una API
- Lee/escribe en una base de datos
- Envía un correo
- Crea un archivo
- Hace una búsqueda web

**Aquí es donde el código tradicional toma el relevo.** El LLM dice "llama a `enviar_correo` con estos parámetros", y un trozo de código Python (o n8n, o lo que sea) realmente hace la llamada.

### 4 · Observar (Observe)

El resultado de la acción vuelve al agente:

- Éxito o error
- Datos devueltos
- Cambios en el sistema

Ese resultado **se agrega al contexto**. Y vuelve al paso 1.

El loop continúa hasta que el LLM decide que el objetivo está cumplido (o se alcanza un límite predefinido de iteraciones).

## El loop en pseudocódigo

Para que veas que no es magia, así se vería el loop más básico posible:

```python
# Pseudocódigo del loop agéntico
contexto = [mensaje_inicial_del_usuario]

while True:
    # 1 · Razonar
    decision = llm.decidir_siguiente_paso(contexto)
    
    if decision.es_respuesta_final:
        return decision.respuesta
    
    # 2 · Actuar
    resultado = ejecutar_herramienta(decision.tool, decision.parametros)
    
    # 3 · Observar (agregar al contexto)
    contexto.append(decision)
    contexto.append(resultado)
    
    # 4 · Volver a percibir → siguiente iteración
```

Eso es todo. Cualquier agente que veas, por más complejo que parezca, tiene este loop como núcleo.

## Un ejemplo concreto

Imagina que le dices a un agente: *"Mira el clima de mañana y agéndame una reunión con Pedro a las 9am si no llueve."*

El loop se ejecuta así:

| Iteración | Percibir | Razonar | Actuar | Observar |
|-----------|----------|---------|--------|----------|
| **1** | Pregunta del usuario | Necesito el clima | `obtener_clima(fecha="mañana")` | "Soleado, 0% lluvia" |
| **2** | Resultado del clima | No llueve, agendar reunión | `crear_evento(con="Pedro", hora="9am")` | "Evento creado" |
| **3** | Confirmación | Tarea completada | (ninguna) | — |
| **Final** | El agente responde al usuario: "Listo, agendé tu reunión con Pedro a las 9am, mañana no se espera lluvia." | | | |

Tres iteraciones. Dos herramientas usadas. El LLM no programó cuándo usar cada una — **lo decidió en cada paso según el contexto disponible**.

## ¿Quién controla el loop?

Hay dos formas de implementar el loop:

### Opción A · Loop dirigido por código

Tu código (Python, n8n, etc.) lleva el control. Llama al LLM, lee la decisión, ejecuta la herramienta, y vuelve a llamar al LLM. El LLM solo "responde preguntas" sobre qué hacer.

**Ventajas**: predecible, fácil de debuggear, caro de manera controlada.

**Es lo que vamos a hacer en el Caso 2 del workshop.**

### Opción B · Loop dirigido por el agente

El LLM mismo decide cuándo terminar y devuelve directamente la respuesta final. Frameworks como LangGraph, CrewAI o AutoGen abstraen esto.

**Ventajas**: menos código, más flexible.

**Desventajas**: más impredecible, puede entrar en loops infinitos si no le pones límites.

> 📌 **Nota**
>
> En el workshop usamos siempre **Opción A** porque es más educativa: ves exactamente qué pasa en cada iteración. En producción, frecuentemente acabas mezclando ambas.

## Los límites del loop

Sin condiciones de salida, un agente puede iterar para siempre. Por eso siempre debes definir **límites de seguridad**:

- **Máximo de iteraciones** (ej. 10 vueltas)
- **Timeout** (ej. 60 segundos máximo)
- **Presupuesto de tokens** (ej. máximo 10,000 tokens consumidos)
- **Lista de herramientas permitidas** (no le des acceso a más de lo necesario)

Si un agente llega a uno de estos límites, debe detenerse y reportar lo que alcanzó a hacer — incluso si no completó el objetivo.

> ⚠️ **Cuidado**
>
> En tu primer agente, **siempre define un límite bajo de iteraciones** (5 o 10). Un loop accidentalmente infinito puede consumir tus créditos rápido. Más vale que el agente diga "no pude" que descubrir una factura sorpresa.

## Por qué los chatbots NO tienen este loop

Un chatbot puro, sin tools, no necesita el loop:

```
Usuario pregunta → LLM responde → Fin
```

Un solo paso. No hay nada que ejecutar, observar, ni decidir adicional. Por eso un chatbot es más simple y predecible.

El loop nace cuando das al LLM **la capacidad de actuar**, y por lo tanto la posibilidad de que su acción **revele nueva información** que requiera otra acción.

## Recapitulando

- Todo agente funciona en un **loop de 4 pasos**: Percibir → Razonar → Actuar → Observar
- El **LLM razona y decide**; el **código ejecuta** (la mayoría de las veces)
- El loop continúa hasta que el LLM declare la tarea completa o se alcance un límite
- Sin tools, no hay loop — solo hay chatbot

> 🎯 **Reto**
>
> Toma una tarea cotidiana de tu trabajo (responder un correo, crear un ticket, validar un documento) e identifica:
> - ¿Qué información necesitas para empezar (Percibir)?
> - ¿Cuáles son las decisiones que tomas (Razonar)?
> - ¿Qué acciones ejecutas (Actuar)?
> - ¿Cómo sabes si fueron exitosas (Observar)?
>
> Esa tarea, en su esencia, ya tiene la forma de un agente.

---

**Siguiente: [1.3 · Anatomía de un agente →](13-anatomia.md)**
