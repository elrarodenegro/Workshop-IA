# 2.2 · Nivel 2 · RAG + memoria

## El problema que resuelve

El Nivel 1 es genial, pero tiene un techo: **el agente solo sabe lo que puso en el system prompt o lo que devuelven sus tools**.

Si quieres que tu agente:
- Conozca toda la documentación de tu producto
- Recuerde conversaciones de hace meses
- Aprenda de casos resueltos previamente

...el Nivel 1 ya no alcanza. El system prompt no puede crecer infinitamente — está limitado por el contexto del modelo.

**Solución**: agregamos memoria persistente y un mecanismo de búsqueda inteligente. Bienvenido al Nivel 2.

## Qué es RAG

**RAG** = **Retrieval-Augmented Generation** (Generación Aumentada por Recuperación).

La idea: en vez de meter toda la información en el prompt, **recuperas solo lo relevante en cada llamada**.

Funciona así:

1. Tienes una base de conocimiento (documentos, FAQs, conversaciones previas, manuales) almacenada en una **vector database**
2. Cuando llega una pregunta del usuario, **buscas en la vector DB los fragmentos más relevantes**
3. Inyectas esos fragmentos en el contexto del LLM
4. El LLM responde usando esa información reciente y específica

## El diagrama

```
┌──────────┐         ┌─────────────┐         ┌──────────┐
│ USUARIO  │────────▶│   LLM       │◀────────│ VECTOR   │
└──────────┘ pregunta│  AGENT      │ relevant│ DB       │
                     │             │ context │ (Pinecone│
                     │ + retrieval │         │  Chroma) │
                     │ + tools     │         └──────────┘
                     └─────────────┘                │
                            │                       │
                            ▼                       ▼
                       respuesta            ┌──────────────┐
                                            │ KNOWLEDGE    │
                                            │ BASE         │
                                            │ (docs, FAQs) │
                                            └──────────────┘
```

## Caso típico

> *"Agente de soporte que conoce toda la documentación interna de tu producto y aprende de cada caso resuelto."*

Antes de responder, el agente:

1. Busca en la documentación los temas relacionados con la pregunta
2. Busca en casos previamente resueltos cómo se manejaron problemas similares
3. Combina ambos contextos
4. Responde con esa información específica

**El usuario no escribe "consulta la documentación".** El agente lo hace automáticamente, en cada turno.

## ¿Qué es una vector database?

Una vector DB almacena texto convertido a **embeddings** — vectores numéricos que representan el significado del texto.

**La magia**: textos con significados similares quedan **cerca** en el espacio vectorial, aunque usen palabras distintas.

Ejemplo:
- "¿Cómo reseteo mi contraseña?"
- "Olvidé mi clave"
- "No puedo iniciar sesión, mi password no funciona"

Las tres frases son **distintas** sintácticamente pero **cercanas** semánticamente. Una vector DB las puede encontrar todas con la misma búsqueda.

### Vector DBs populares

| DB | Característica | Cuándo usar |
|----|----------------|-------------|
| **Pinecone** | Hosted, fácil de usar | MVPs, equipos sin DevOps |
| **Chroma** | Open source, embebida | Proyectos locales, prototipos |
| **Weaviate** | Open source, hosted o self-hosted | Producción flexible |
| **pgvector** | Extensión de PostgreSQL | Si ya tienes Postgres |
| **Qdrant** | Open source, performante | Producción grande |

## Cómo se ve en código (sketch conceptual)

```python
# Pseudocódigo de un agente Nivel 2
import os
from anthropic import Anthropic
import chromadb

client = Anthropic()
vector_db = chromadb.Client()
collection = vector_db.get_collection("documentacion")

def responder(pregunta):
    # 1 · Recuperar contexto relevante
    resultados = collection.query(
        query_texts=[pregunta],
        n_results=5  # los 5 fragmentos más relevantes
    )
    contexto_relevante = "\n\n".join(resultados['documents'][0])
    
    # 2 · Construir el system prompt con el contexto
    system_prompt = f"""
    Eres un asistente de soporte. Usa esta información para responder:
    
    {contexto_relevante}
    
    Si la información no cubre la pregunta, dilo honestamente.
    """
    
    # 3 · Llamar al LLM (con o sin tools adicionales)
    response = client.messages.create(
        model="claude-opus-4-5",
        system=system_prompt,
        messages=[{"role": "user", "content": pregunta}],
        max_tokens=1024
    )
    
    return response.content[0].text
```

## Memoria de corto plazo vs largo plazo

| Tipo | Qué guarda | Dónde vive |
|------|------------|------------|
| **Corto plazo** | La conversación actual | En el array `messages` que pasas al LLM |
| **Largo plazo** | Todo el conocimiento histórico | En la vector DB |

Un agente Nivel 2 maduro usa **ambos**:

- Pregunta del usuario
- Recupera de **largo plazo** lo relevante (vector DB)
- Mantiene la **conversación actual** en memoria de corto plazo
- Combina ambos en cada llamada al LLM

## El paso de "indexar"

Para que la vector DB sea útil, primero hay que **alimentarla**. Este paso se llama **indexación** o **ingesta**:

```python
# Ingestar documentos a la vector DB
documentos = [
    "El producto X soporta integración con Slack via webhooks...",
    "Para resetear contraseña, ve a Settings → Account → Reset Password...",
    "Errores 502 generalmente indican problemas con el load balancer..."
]

# Esto convierte cada doc a embedding y lo guarda
collection.add(
    documents=documentos,
    ids=["doc_001", "doc_002", "doc_003"]
)
```

**Es un proceso batch**, generalmente offline. Lo haces cuando publicas documentación nueva o periódicamente reindexar todo.

## Ventajas del Nivel 2

✅ **El agente conoce información específica de tu negocio** sin meterla toda en el prompt
✅ **Aprende con el tiempo** (puedes ingestar nuevos documentos)
✅ **Reduce alucinaciones** — el LLM tiene info real para citar
✅ **Más barato a largo plazo** — no pagas tokens por mantener todo en el prompt

## Desventajas

❌ **Más complejidad** — ya no es solo el LLM, es LLM + vector DB + pipeline de ingesta
❌ **Calidad depende de la ingesta** — si tus documentos están mal organizados, las recuperaciones serán malas
❌ **Requiere mantenimiento** — la vector DB necesita actualizarse cuando cambia la documentación
❌ **No es magia** — si el documento relevante no existe en la DB, el agente no lo va a encontrar

## El error común: "RAG mal hecho"

Mucha gente arma un RAG así:
- Vector DB con cualquier documento que encontraron
- Recuperación de 20 fragmentos por pregunta
- Sin estructura ni metadata
- Sin actualización

Y luego se quejan de que "no funciona". El RAG es un **sistema de información**: la calidad de salida depende de la calidad de entrada.

> 💡 **Tip**
>
> Antes de armar RAG, pregúntate: **¿qué documentos voy a ingestar y por qué? ¿están limpios? ¿están bien etiquetados? ¿hay duplicados?** Resolver eso antes te ahorra meses de iterar sobre un sistema que nunca da buenos resultados.

## Cuándo subir al Nivel 2

✅ **Considera Nivel 2 cuando:**

- Tu agente necesita conocer documentación específica que no cabe en el prompt
- Quieres que recuerde interacciones de usuarios entre sesiones
- Estás construyendo un asistente de soporte, ventas o docs

❌ **Quédate en Nivel 1 si:**

- La info que necesita el agente cabe en su system prompt (~5,000 palabras)
- No necesitas memoria entre conversaciones
- Estás haciendo un prototipo o demo

## En el workshop

**No hacemos un agente Nivel 2 en el workshop principal** — agregaría 1-2 horas y no es necesario para entender los conceptos. Pero ahora sabes qué es y cuándo lo necesitarías.

Si quieres profundizar después del workshop, los recursos en el [Apéndice B](../apendices/b-recursos.md) tienen tutoriales específicos.

## Recapitulando

- Nivel 2 = Nivel 1 **+ vector DB + RAG + memoria de largo plazo**
- Resuelve el problema de "no cabe todo en el prompt"
- Vive en aplicaciones reales: soporte, knowledge bases, asistentes especializados
- Más complejo de mantener, pero mucho más capaz

---

**Siguiente: [2.3 · Nivel 3 · Multi-agente →](23-nivel-3-multi-agente.md)**
