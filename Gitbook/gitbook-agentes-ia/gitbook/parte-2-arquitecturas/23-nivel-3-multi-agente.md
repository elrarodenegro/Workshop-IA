# 2.3 · Nivel 3 · Multi-agente / orquestación

## La idea de fondo

Llega un punto donde un solo agente con muchas tools se vuelve **menos eficiente** que varios agentes especializados que colaboran.

Es lo mismo que pasa con humanos: un equipo de 5 especialistas con un coordinador suele rendir mejor que una persona haciendo de todo.

## El diagrama

```
                    ┌─────────────┐
                    │ SUPERVISOR  │
                    │  (router)   │
                    └─────┬───────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ INVESTIGADOR │  │  REDACTOR    │  │  REVISOR     │
│ busca y      │  │  sintetiza   │  │  valida      │
│ recolecta    │  │  y escribe   │  │  y mejora    │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                 │
       └────────────[Resultado]────────────┘
```

Tres especialistas. Un supervisor que coordina.

## Patrones comunes

### Router (enrutador)

Un agente "supervisor" recibe la tarea y **decide a qué especialista mandarla**. Cada especialista tiene su propio set de tools y system prompt.

**Ejemplo**: un asistente legal recibe una consulta. El router decide: ¿es contratos, propiedad intelectual, o laboral? Manda al especialista correspondiente.

### Planner-Executor

Un agente **planifica** los pasos. Otro agente (o varios) los **ejecuta**. El planificador no tiene tools de acción — solo razona sobre la estrategia.

**Ejemplo**: planificador genera un plan de 5 pasos para escribir un reporte. El ejecutor pasa por cada paso usando sus herramientas.

### Swarm (enjambre)

Agentes pares colaboran sin jerarquía. Cada uno aporta desde su especialidad. Útil para creatividad o brainstorming.

**Ejemplo**: 3 agentes con personalidades distintas (técnico, marketing, financiero) discuten una propuesta de producto.

## Por qué hacerlo así

### Ventaja 1 · Especialización

Cada agente tiene un system prompt enfocado en su rol y solo las tools necesarias para él. Esto reduce errores y mejora calidad — el agente "investigador" no se distrae con cómo redactar.

### Ventaja 2 · Modularidad

Puedes mejorar/reemplazar un agente sin tocar los demás. Si el agente "redactor" no escribe bien, lo iteras a él solo.

### Ventaja 3 · Paralelización

Si las tareas son independientes, varios agentes pueden trabajar **en paralelo**. Reduce tiempo total.

### Ventaja 4 · Trazabilidad

Es más fácil debuggear qué falló cuando ves "el agente investigador devolvió mala data" vs. "algo en el agente único salió mal".

## Las desventajas (importantes)

### ❌ Costo se multiplica

Cada agente cuesta. Tres agentes haciendo lo que uno haría → hasta 3x el costo.

### ❌ Latencia se acumula

Si los agentes son secuenciales, sumas el tiempo de cada uno.

### ❌ Complejidad se dispara

Coordinar 3 agentes es más difícil que tener 1. Decidir qué información pasarle a cada uno es un problema en sí mismo.

### ❌ Errores en cadena

Si el investigador da datos mal, el redactor escribe basura, el revisor avala basura. **El error en uno se propaga.**

> ⚠️ **Cuidado**
>
> No subas a multi-agente "porque queda cool en una presentación". Solo si **un agente único, bien diseñado, no resuelve tu problema**.

## Frameworks populares

Construir sistemas multi-agente desde cero es complejo. Hay frameworks que te dan abstracciones útiles:

| Framework | Característica |
|-----------|----------------|
| **LangGraph** | Grafos de estado, control fino del flujo |
| **CrewAI** | Pensado en "tripulaciones" con roles definidos |
| **AutoGen** (Microsoft) | Conversaciones entre agentes, código primero |
| **OpenAI Swarm** | Ligero, oficial de OpenAI, simple |

> 📌 **Nota**
>
> Estos frameworks se mueven rápido — versiones nuevas cambian APIs cada pocos meses. Si los vas a usar, **fija una versión** en tu `requirements.txt` y prueba antes de actualizar.

## Cómo se ve, conceptualmente

```python
# Pseudocódigo - patrón "Router con especialistas"

def supervisor(tarea):
    # El supervisor solo decide qué agente usar
    decision = llm_supervisor.clasificar(tarea)
    
    if decision == "investigar":
        return agente_investigador.ejecutar(tarea)
    elif decision == "redactar":
        return agente_redactor.ejecutar(tarea)
    elif decision == "revisar":
        return agente_revisor.ejecutar(tarea)

class AgenteInvestigador:
    system_prompt = "Eres un investigador. Tu trabajo es..."
    tools = [busqueda_web, leer_documento, consultar_db]
    
    def ejecutar(self, tarea):
        # loop de Tool Use con sus tools específicas
        return resultado

class AgenteRedactor:
    system_prompt = "Eres un redactor. Tu trabajo es..."
    tools = [generar_borrador, revisar_estilo]
    
    def ejecutar(self, tarea):
        # loop de Tool Use con sus tools específicas
        return resultado

# ... etc
```

Cada agente es básicamente un Nivel 1 con su propia personalidad. La novedad es **la capa de coordinación encima**.

## Cuándo está justificado

✅ **Considera multi-agente cuando:**

- La tarea tiene fases naturalmente distintas con tools y conocimientos muy diferentes
- Hay valor en revisión cruzada (un agente que valide al otro)
- Necesitas paralelización real (varios agentes simultáneos)
- Tienes presupuesto para los costos extras

❌ **NO subas a multi-agente cuando:**

- Un agente único con buenas tools resuelve el caso
- Estás haciendo un prototipo o demo
- El presupuesto es ajustado
- No tienes claras las "fronteras" entre los roles de los agentes

## El antipatrón: "agentes por todos lados"

He visto equipos descomponer todo en multi-agente:
- Agente clasificador
- Agente extractor
- Agente validador
- Agente formateador
- Agente notificador

Cada uno con LLM. **Para una tarea que un solo Tool Use con 4 tools haría 5x más barato y rápido.**

> 💡 **Tip**
>
> Antes de agregar otro agente, pregúntate: **¿esto realmente requiere razonamiento independiente, o es solo otro paso del flujo que un solo agente con tool más puede hacer?**

## En el workshop

El workshop **no incluye un caso multi-agente** — añade complejidad sin enseñar conceptos nuevos sobre el loop fundamental. Pero ahora sabes:

- Cuándo lo justificarías
- Qué patrones existen
- Qué frameworks te ayudan

Cuando lo necesites en producción, vas con criterio.

## Recapitulando

- Nivel 3 = **varios agentes especializados + coordinación**
- Patrones: Router, Planner-Executor, Swarm
- Frameworks que ayudan: LangGraph, CrewAI, AutoGen, OpenAI Swarm
- **Justificado** cuando hay fases muy distintas con tools y prompts muy diferentes
- **Antipatrón común**: descomponer todo en agentes innecesariamente

---

**Siguiente: [2.4 · Nivel 4 · Producción →](24-nivel-4-produccion.md)**
