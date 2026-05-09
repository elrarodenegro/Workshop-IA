# 2.4 · Nivel 4 · Agente autónomo en producción

## El salto a producción

Tener un agente que funciona en tu máquina local es 10% del trabajo. Llevarlo a producción — donde lo usan usuarios reales, 24/7, con dinero o data sensible — es el 90% restante.

Este capítulo es un **mapa**, no un tutorial. Cada capa que veremos puede ser un libro completo. La idea es que sepas **qué necesitas considerar** antes de poner un agente al mundo real.

## Las 6 capas de un agente productivo

```
┌──────────────────────────────────────────┐
│ 1. OBSERVABILIDAD · logs, traces, métricas│
├──────────────────────────────────────────┤
│ 2. GUARDRAILS · validación, límites, auth│
├──────────────────────────────────────────┤
│ 3. HUMAN-IN-THE-LOOP · aprobaciones      │
├──────────────────────────────────────────┤
│ 4. COLA DE TAREAS · async workers        │
├──────────────────────────────────────────┤
│ 5. PERSISTENCIA · DB + vector store      │
├──────────────────────────────────────────┤
│ 6. INFRAESTRUCTURA · Docker, K8s, cloud  │
└──────────────────────────────────────────┘
```

## 1 · Observabilidad

**El problema**: si tu agente toma 1,000 decisiones al día, ¿cómo sabes qué está pasando?

Necesitas:

### Logs estructurados

No `print("usuario hizo algo")`. Estructura cada evento:

```python
logger.info({
    "event": "tool_executed",
    "tool_name": "send_email",
    "agent_id": "agent_abc",
    "user_id": "user_123",
    "duration_ms": 245,
    "tokens_used": 1547
})
```

### Traces

Cada interacción con un agente puede involucrar 5+ llamadas al LLM y 10+ ejecuciones de tools. Un **trace** une todos esos eventos en una línea de tiempo navegable.

**Herramientas**: LangSmith, Helicone, Langfuse, Datadog.

### Métricas

Lo mínimo que debes monitorear:

- **Latencia promedio y p95** de las respuestas
- **Costo por interacción** (tokens × precio)
- **Tasa de éxito** (¿cuántos agentes terminaron su tarea?)
- **Tasa de error** por tipo (timeout, rate limit, tool failure)
- **Alertas** cuando algo se sale de rangos normales

## 2 · Guardrails (límites de seguridad)

Lo que protege a tus usuarios y a tu negocio.

### Validación de inputs

Antes de pasar datos al LLM, **valida**:

- ¿El input tiene el formato esperado?
- ¿El usuario tiene permiso para esta acción?
- ¿No hay intentos de prompt injection?

### Validación de outputs

Antes de **ejecutar** lo que el LLM dice:

- ¿El tool_call tiene parámetros válidos?
- ¿Los parámetros están dentro de rangos seguros?
- ¿El output contiene información sensible que no debería?

### Límites de uso

- **Rate limiting** por usuario (no más de N llamadas/hora)
- **Spending limits** (no más de $X por usuario/día)
- **Iteration limits** (no más de N vueltas del loop)

### Sanitización

Si tu agente publica contenido o lo muestra a otros usuarios, **filtra**:
- Contenido tóxico
- Datos personales (PII) que se filtraron
- Código que podría ser inyección de scripts

> 💡 **Tip**
>
> Un buen guardrail es uno que **rechaza por default y permite por excepción**. Más fácil de auditar y razonar sobre seguridad.

## 3 · Human-in-the-loop (HITL)

Para acciones críticas, **un humano valida antes de ejecutar**.

### Cuándo aplicarlo

- Acciones que mueven dinero
- Cambios irreversibles (eliminar datos, enviar comunicaciones masivas)
- Decisiones legales o regulatorias
- Cualquier cosa donde un error es costoso

### Cómo se ve

```python
# Pseudocódigo
decision = agente.proponer_accion(input)

if decision.es_critica:
    # Pausa, espera aprobación
    aprobacion = enviar_a_humano_para_aprobar(decision)
    if aprobacion.aprobada:
        ejecutar(decision)
    else:
        agente.notificar_rechazo(aprobacion.razon)
else:
    # Acción no crítica, sigue automática
    ejecutar(decision)
```

> 📌 **Nota**
>
> HITL bien diseñado es **invisible cuando todo está bien** y **crítico cuando algo está mal**. No conviertas tu agente en un formulario de aprobaciones constantes — solo escala lo realmente importante.

## 4 · Cola de tareas y workers asíncronos

**El problema**: una llamada a un agente puede tardar 30+ segundos. No puedes hacer al usuario esperar en una request HTTP síncrona.

### Patrón estándar

```
Usuario → API recibe request → Encola tarea → Devuelve "task_id"
                                    │
                                    ▼
                              Worker pesca de la cola
                                    │
                                    ▼
                              Ejecuta el agente (segundos/minutos)
                                    │
                                    ▼
                              Guarda resultado en DB
                                    │
                                    ▼
                  Usuario consulta task_id → recibe resultado
```

### Stack típico

- **Cola**: Redis, RabbitMQ, AWS SQS
- **Workers**: Celery (Python), BullMQ (Node), Sidekiq (Ruby)
- **Notificación de progreso**: WebSockets, Server-Sent Events, polling

> 📌 **Nota**
>
> Tu autor (Cristian) construyó el "Tenable AI Agent" usando exactamente este stack: **FastAPI + Celery + Redis + PostgreSQL**. Es la combinación canónica para agentes en producción Python.

## 5 · Persistencia

### Qué persistir

- **Conversaciones**: cada interacción con cada usuario
- **Estado de los agentes**: si un agente quedó a la mitad y debe continuar
- **Resultados de tools**: para no re-ejecutar lo que ya se ejecutó
- **Métricas de uso**: para análisis posterior
- **Embeddings**: la vector DB con tu base de conocimiento

### Stack típico

- **PostgreSQL** o MySQL para relacional (usuarios, tasks, logs estructurados)
- **Pinecone / Chroma / pgvector** para vectores
- **Redis** para caché y datos volátiles
- **S3 / Object storage** para archivos generados

## 6 · Infraestructura

Donde corre todo.

### Containerización

**Docker** — empaqueta tu agente en una imagen reproducible. La misma imagen corre en tu laptop y en producción.

```dockerfile
# Ejemplo simplificado de Dockerfile para un agente
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Orquestación

Cuando son varios contenedores (API + workers + Redis + Postgres):

- **Docker Compose** para desarrollo y deploys pequeños
- **Kubernetes** para producción seria con escalado automático

### Cloud

Donde lo hosteas:

- **AWS, GCP, Azure** — suite completa, complejos pero potentes
- **Render, Fly.io, Railway** — más simples, suficientes para MVPs y mediana escala
- **On-premise** — si tu data no puede salir de tu red

## La regla del 90/10

Si no has hecho deploy a producción antes:

> **El 10% del trabajo es construir el agente. El 90% es todo lo que rodea.**

No te asustes — es normal. Significa que lo que aprendas sobre Tool Use en este libro es valioso, pero no suficiente. La ingeniería de software clásica (CI/CD, testing, monitoring, seguridad) sigue siendo crucial.

## Mínimo viable de producción

Si vas a llevar tu primer agente a producción y necesitas un checklist mínimo:

- [ ] Logs estructurados (al menos `logger.info` con dict)
- [ ] Rate limiting por usuario
- [ ] Spending limit en las APIs (configurado en cada proveedor)
- [ ] Validación de inputs antes del LLM
- [ ] Iteration limit en el loop (máximo N vueltas)
- [ ] Cola de tareas (Celery + Redis para Python)
- [ ] DB persistente (Postgres para conversaciones)
- [ ] Container (Docker)
- [ ] Healthcheck endpoint en tu API
- [ ] Alertas básicas (email cuando algo se rompe)

Diez puntos. Si todos están, probablemente puedes operar tu agente con tranquilidad razonable.

## Recapitulando

- Producción ≠ "lo subo a Render y ya"
- 6 capas: observabilidad, guardrails, HITL, colas, persistencia, infra
- El **90/10**: construir el agente es solo el 10% del trabajo total
- Hay un mínimo viable de producción de 10 ítems

---

**Siguiente: [2.5 · MCP · Model Context Protocol →](25-mcp.md)**
