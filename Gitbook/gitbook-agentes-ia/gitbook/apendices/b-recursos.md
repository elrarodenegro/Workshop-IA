# Apéndice B · Recursos para seguir aprendiendo

Lista curada de los mejores recursos para profundizar después del workshop. Organizada por categoría.

## Documentación oficial

### Anthropic (Claude)
- **Cookbook**: [github.com/anthropics/anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) — ejemplos en notebooks
- **Docs**: [docs.anthropic.com](https://docs.anthropic.com)
- **Tool Use guide**: [docs.anthropic.com/en/docs/build-with-claude/tool-use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- **Prompting guide**: [docs.anthropic.com/en/docs/build-with-claude/prompt-engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)

### OpenAI
- **Cookbook**: [cookbook.openai.com](https://cookbook.openai.com) — el más completo de la industria
- **Docs**: [platform.openai.com/docs](https://platform.openai.com/docs)
- **Function calling guide**: [platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling)

### Google AI
- **Docs**: [ai.google.dev/docs](https://ai.google.dev/docs)
- **Function calling guide**: [ai.google.dev/gemini-api/docs/function-calling](https://ai.google.dev/gemini-api/docs/function-calling)

## Frameworks de agentes

### LangChain & LangGraph
- **Sitio**: [langchain.com](https://www.langchain.com)
- **Docs**: [python.langchain.com](https://python.langchain.com)
- **LangGraph** (para multi-agente con grafos): [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/)
- **LangSmith** (observabilidad): [langsmith.com](https://www.langsmith.com)

### CrewAI
- **Sitio**: [crewai.com](https://www.crewai.com)
- **Docs**: [docs.crewai.com](https://docs.crewai.com)
- Pensado para equipos de agentes con roles claros

### AutoGen (Microsoft)
- **Repo**: [github.com/microsoft/autogen](https://github.com/microsoft/autogen)
- **Docs**: [microsoft.github.io/autogen](https://microsoft.github.io/autogen/)
- Conversaciones entre agentes, código-first

### LlamaIndex
- **Sitio**: [llamaindex.ai](https://www.llamaindex.ai)
- **Docs**: [docs.llamaindex.ai](https://docs.llamaindex.ai)
- Especialmente fuerte en RAG y manejo de datos

## MCP (Model Context Protocol)

- **Especificación oficial**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Servidores públicos**: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)
- **SDK para Python**: [github.com/modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk)
- **SDK para TypeScript**: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)

## Vector databases

- **Pinecone**: [pinecone.io](https://www.pinecone.io) — hosted, fácil de usar
- **Chroma**: [trychroma.com](https://www.trychroma.com) — open source, embebida
- **Weaviate**: [weaviate.io](https://weaviate.io) — open source, hosted o self-hosted
- **pgvector**: [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector) — extensión de Postgres
- **Qdrant**: [qdrant.tech](https://qdrant.tech) — open source, performante

## Low-code y plataformas visuales

- **n8n**: [n8n.io](https://n8n.io) — el del Caso 1
- **Make** (antes Integromat): [make.com](https://www.make.com)
- **Zapier**: [zapier.com](https://zapier.com) — más limitado en agentes pero útil
- **Pipedream**: [pipedream.com](https://pipedream.com)
- **Activepieces**: [activepieces.com](https://www.activepieces.com) — open source, alternativa a Zapier

## Papers seminales (lectura recomendada)

Si quieres entender los fundamentos académicos:

- **ReAct: Synergizing Reasoning and Acting in Language Models** (2022) — el paper que formalizó el patrón Razonar+Actuar en LLMs
  - [arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

- **Toolformer: Language Models Can Teach Themselves to Use Tools** (2023)
  - [arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

- **Tree of Thoughts: Deliberate Problem Solving with Large Language Models** (2023)
  - [arxiv.org/abs/2305.10601](https://arxiv.org/abs/2305.10601)

- **Reflexion: Language Agents with Verbal Reinforcement Learning** (2023)
  - [arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366)

## Blogs y newsletters

### Para mantenerte al día
- **Anthropic Engineering Blog**: [anthropic.com/news](https://www.anthropic.com/news)
- **OpenAI Blog**: [openai.com/blog](https://openai.com/blog)
- **Hugging Face Daily Papers**: [huggingface.co/papers](https://huggingface.co/papers)
- **The Batch (DeepLearning.AI)**: [deeplearning.ai/the-batch](https://www.deeplearning.ai/the-batch/) — newsletter semanal

### Voces individuales recomendadas
- **Simon Willison's blog** ([simonwillison.net](https://simonwillison.net/)) — agentes, prompt injection, herramientas
- **Lilian Weng's blog** ([lilianweng.github.io](https://lilianweng.github.io/)) — deep dives técnicos
- **Eugene Yan's blog** ([eugeneyan.com](https://eugeneyan.com)) — patterns para LLMs en producción

## YouTube y video

### Canales recomendados
- **Anthropic** (oficial) — workshops y demos técnicas
- **AI Engineering by Latent Space** — conversaciones técnicas profundas
- **Matthew Berman** — tutoriales accesibles sobre frameworks
- **DeepLearning.AI** — cursos cortos gratuitos sobre LLMs y agentes

## Comunidades

### Discord
- **Anthropic Discord** — soporte oficial
- **OpenAI Discord** — comunidad oficial
- **LangChain Discord** — preguntas sobre el framework

### Reddit
- **r/LocalLLaMA** — modelos open-source, self-hosting
- **r/LangChain** — discussions y problemas comunes
- **r/MachineLearning** — investigación

### Twitter/X
Sigue a (en orden alfabético): Andrej Karpathy, Anthropic, Cassidy Williams, Daniel Han, Dario Amodei, Eugene Yan, OpenAI, Sam Witteveen, Simon Willison, Swyx.

## Cursos estructurados

### Gratis
- **Building Systems with the ChatGPT API** (DeepLearning.AI + OpenAI)
- **AI Agents in LangGraph** (DeepLearning.AI + LangChain)
- **Functions, Tools and Agents with LangChain** (DeepLearning.AI + LangChain)
- **MCP: Build Rich-Context AI Apps with Anthropic** (DeepLearning.AI + Anthropic)

Todos en [learn.deeplearning.ai](https://learn.deeplearning.ai).

### De pago
- **AI Engineering** book by Chip Huyen (O'Reilly) — el libro de referencia para producción
- **Building LLM Powered Applications** book by Valentina Alto

## Para seguridad cibernética específicamente

Como tu autor (Cristian) viene de cybersecurity, recursos especialmente relevantes:

- **MITRE ATT&CK**: [attack.mitre.org](https://attack.mitre.org)
- **MITRE D3FEND**: [d3fend.mitre.org](https://d3fend.mitre.org)
- **NIST AI Risk Management Framework**: [nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework)
- **OWASP Top 10 for LLM Applications**: [genai.owasp.org](https://genai.owasp.org/)
- **Tenable Research Blog**: [tenable.com/blog](https://www.tenable.com/blog)
- **Diamond Model of Intrusion Analysis**: papers académicos del modelo

## Herramientas de observabilidad

- **LangSmith**: [smith.langchain.com](https://smith.langchain.com)
- **Helicone**: [helicone.ai](https://www.helicone.ai)
- **Langfuse**: [langfuse.com](https://langfuse.com) — open source
- **Arize Phoenix**: [phoenix.arize.com](https://phoenix.arize.com) — open source

## Para infraestructura

- **Modal**: [modal.com](https://modal.com) — corre tu código Python en la nube
- **Render**: [render.com](https://render.com) — deploy fácil
- **Fly.io**: [fly.io](https://fly.io)
- **Railway**: [railway.app](https://railway.app)
- **Replicate**: [replicate.com](https://replicate.com) — modelos hospedados

## Tu autor recomienda especialmente

Si tuvieras que elegir 5 recursos para profundizar después del workshop:

1. **Anthropic Cookbook** — para profundizar en Tool Use real
2. **OpenAI Cookbook** — para function calling avanzado
3. **LangChain docs** — para el framework más usado
4. **Simon Willison's blog** — para entender el estado del arte semanalmente
5. **El paper de ReAct** — para entender por qué el patrón funciona

Con estos cinco, tienes 10x más profundidad que con los miles de blogs genéricos.

---

**Siguiente: [C · Cheatsheet de prompts →](c-cheatsheet-prompts.md)**
