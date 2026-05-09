# Apéndice A · Glosario

Términos que vas a encontrar a lo largo del libro y en cualquier conversación seria sobre agentes de IA.

## A

### Agente
Sistema que combina un LLM con herramientas, memoria y un loop de decisión, capaz de tomar acciones para cumplir un objetivo. Distinto de un chatbot, que solo conversa.

### API key
Credencial (tipo contraseña) que te identifica al hacer llamadas a un servicio en la nube. Las usaste para Anthropic, OpenAI y Google.

## C

### Chatbot
LLM con interfaz conversacional y memoria de la sesión actual. No ejecuta acciones más allá de responder texto.

### Context window (ventana de contexto)
Cantidad máxima de tokens que un LLM puede procesar en una sola llamada. Claude tiene 200k, GPT-4 128k, Gemini 1.5 Pro 2M.

### CVE (Common Vulnerabilities and Exposures)
Identificador estándar para vulnerabilidades de seguridad. Formato: `CVE-YYYY-NNNNN`. Lo usamos en el Caso 3.

### CVSS
Common Vulnerability Scoring System. Sistema numérico (0-10) para puntuar la severidad de un CVE.

## E

### Embedding
Representación numérica (vector) del significado de un texto. Textos similares tienen embeddings cercanos en el espacio vectorial.

### `end_turn`
En Anthropic, el `stop_reason` que indica que el LLM terminó de razonar y no necesita más herramientas.

## F

### Few-shot prompting
Técnica donde incluyes ejemplos en el prompt para enseñarle al LLM el formato/comportamiento deseado.

### Function calling
Término de OpenAI y Google para lo que Anthropic llama "Tool Use". El mismo concepto.

## G

### Guardrails
Mecanismos de validación y límite que protegen tu agente y a los usuarios. Pueden ser de inputs, outputs, costos, frecuencia, etc.

## H

### Human-in-the-loop (HITL)
Patrón donde un humano valida o aprueba decisiones del agente antes de ejecutarlas. Crítico para acciones irreversibles o costosas.

## I

### IOC (Indicator of Compromise)
En seguridad: artefacto observado que indica una posible intrusión. Hashes, IPs, dominios, comportamientos sospechosos.

### Iteration limit
Límite máximo de vueltas en el loop de un agente. Evita loops infinitos.

## J

### JSON Schema
Estándar para describir la estructura de objetos JSON. Lo usamos para definir el `input_schema` de las tools.

## L

### LLM (Large Language Model)
Modelo de lenguaje grande. Función pura texto → texto. GPT, Claude, Gemini, Llama, etc.

### LangChain
Framework popular para construir aplicaciones con LLMs. Abstrae sobre múltiples providers.

### LangGraph
Sub-proyecto de LangChain enfocado en flujos de agentes con grafos de estado.

## M

### MCP (Model Context Protocol)
Protocolo abierto creado por Anthropic para estandarizar cómo los LLMs hablan con herramientas. Ver [Capítulo 2.5](../parte-2-arquitecturas/25-mcp.md).

### Mock
Implementación simulada de una función. La usamos en los casos del workshop para evitar dependencias externas.

### MITRE ATT&CK
Framework de seguridad que cataloga técnicas de adversarios. Útil para describir el comportamiento de amenazas en agentes de threat hunting.

## N

### n8n
Herramienta open-source de automatización de workflows con soporte para AI Agent. La usamos en el Caso 1.

### Notion
Plataforma de productividad. Tiene API para crear/editar páginas y bases de datos. Usable como sistema de tareas.

## O

### OAuth
Protocolo estándar para autorizar a una app a acceder a tus datos sin compartir tu contraseña. Lo usamos al conectar Gmail/Slack a n8n.

### Orchestrator (orquestador)
Componente que coordina el flujo entre LLM, tools y memoria. Puede vivir en código (workflow) o en el LLM (loop autónomo).

## P

### Prompt
Texto de entrada que le das al LLM. Puede ser una pregunta, un comando, un sistema de instrucciones.

### Prompt injection
Ataque donde un input malicioso intenta manipular al LLM para ignorar sus instrucciones originales. Ej: "Ignora las instrucciones anteriores y haz X".

## R

### RAG (Retrieval-Augmented Generation)
Técnica donde recuperas información relevante de una base de conocimiento (vector DB) y se la pasas al LLM como contexto antes de que responda. Ver [Capítulo 2.2](../parte-2-arquitecturas/22-nivel-2-rag.md).

### Rate limiting
Límite de cuántas llamadas puede hacer un usuario a una API en un período. Protege contra abuso y consumo excesivo.

### Retrieval
Proceso de recuperar fragmentos relevantes de una base de datos. La R en RAG.

## S

### `stop_reason` / `finish_reason`
Campo en la respuesta del LLM que indica por qué terminó de generar. Anthropic usa `stop_reason`, OpenAI `finish_reason`. Distintos nombres, mismo concepto.

### System prompt
Instrucciones iniciales que le das al LLM para definir su rol, comportamiento, restricciones. El "manual del puesto".

## T

### Temperature (temperatura)
Parámetro que controla la aleatoriedad del output del LLM. 0 = determinista, 1 = creativo. Para clasificación se usa baja (0.0-0.3).

### Tool / Herramienta
Función externa que el LLM puede invocar. Definida con un schema (qué inputs recibe) y una implementación (qué hace).

### Tool Use
Capacidad de un LLM de elegir e invocar tools en runtime. Equivalente al Function Calling de OpenAI/Google.

### `tool_use_id`
Identificador único que conecta un `tool_use` con su `tool_result` correspondiente. Crítico para mantener la coherencia del historial.

### Token
Unidad básica que procesa un LLM. Aproximadamente: 1 token ≈ 4 caracteres en inglés, ≈ 0.5 palabras en español.

### Tracing
Práctica de registrar y conectar todos los eventos de una ejecución del agente para debugging y observabilidad.

## V

### Vector database (vector DB)
Base de datos especializada en almacenar y buscar embeddings. Pinecone, Chroma, Weaviate, pgvector son ejemplos.

### venv (virtual environment)
Entorno virtual de Python que aísla las dependencias de un proyecto. Ver [Capítulo 0.4](../parte-0-setup/04-venv.md).

### VPR (Vulnerability Priority Rating)
Sistema de Tenable para priorizar vulnerabilidades según contexto. Más rico que el CVSS solo. Útil en operaciones de seguridad.

## W

### Webhook
URL que recibe notificaciones HTTP cuando ocurre un evento. Ej: GitHub puede mandar un webhook a tu agente cuando alguien hace push.

### Workflow
Flujo de pasos definidos para ejecutar un proceso. En n8n cada workflow es una secuencia de nodos.

---

**Siguiente: [B · Recursos para seguir aprendiendo →](b-recursos.md)**
