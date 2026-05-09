# Comparación lado a lado

Ahora que implementaste el mismo agente con los tres SDKs, podemos comparar sin hablar de marketing.

## Tabla resumen

| Aspecto | Claude | GPT-4 | Gemini |
|---------|--------|-------|--------|
| **Estructura de tool** | Plana (`name, description, input_schema`) | Anidada (`type:"function", function:{...}`) | Protobuf (Tool > FunctionDeclaration) |
| **Argumentos** | Dict directo | String JSON (hay que parsearlo) | Protobuf (hay que convertir) |
| **System prompt** | Parámetro aparte (`system=`) | Mensaje en el array | Parámetro `system_instruction` |
| **stop_reason / finish_reason** | `end_turn`, `tool_use` | `stop`, `tool_calls` | implícito (sin function_call → terminó) |
| **Múltiples tool calls** | En un solo mensaje user | Mensajes "tool" separados | Lista de Parts en un send_message |
| **Manejo de historial** | Manual (array messages) | Manual (array messages) | Automático (`chat.send_message`) |
| **Identificador de la llamada** | `id` (en el block) | `tool_call_id` | (no aplica) |

## Líneas de código

Cuántas líneas necesitas para el agente equivalente (sin contar mocks ni system prompt, sin comentarios):

| SDK | Líneas (aprox.) | Notas |
|-----|------------------|-------|
| Anthropic | ~50 | Más limpio gracias a estructura plana |
| OpenAI | ~60 | Necesita parsear JSON args y manejar tool_calls como objetos |
| Gemini | ~70 | Más verbose con protobuf (hay shortcut con funciones Python) |

Si usas el shortcut de Gemini con funciones Python, baja a ~40 líneas — pero pierdes algo de control sobre el schema.

## Calidad del tool calling

**Disclaimer importante**: esto es subjetivo y depende del caso. No es un benchmark formal. Pero tras hacer este ejercicio múltiples veces:

| Aspecto | Mejor performer (típicamente) |
|---------|------------------------------|
| Decidir cuándo NO usar una tool | Claude (rara vez sobreactúa) |
| Combinar múltiples tools en una llamada | Claude / GPT-4 (paralelo nativo) |
| Manejar tools complejas con muchos parámetros | Claude / GPT-4 |
| Razonamiento explícito antes de llamar | Claude (más natural el text + tool_use mezclado) |
| Tareas multimodales | Gemini (video/audio nativo) |
| Procesar documentos muy largos | Gemini (2M tokens) |
| Costo bajo manteniendo calidad razonable | Gemini Flash o GPT-4o-mini |

> 📌 **Nota**
>
> Estas son tendencias, no leyes. **Prueba con TU caso específico** antes de comprometerte. Lo que funciona bien en ejemplos genéricos puede no aplicar a tu dominio.

## Ergonomía de desarrollo

### El SDK más fácil para empezar

🥇 **Anthropic**: estructura plana, conceptos claros, documentación coherente
🥈 **OpenAI**: ecosystem maduro, mucha documentación pero estructura más anidada
🥉 **Google**: dos formas (verbose vs simplificada), curva inicial más pronunciada

### El SDK con mejor manejo de errores

🥇 **OpenAI**: errores muy descriptivos, status codes consistentes
🥈 **Anthropic**: errores claros pero a veces genéricos
🥉 **Google**: a veces los errores vienen como protobuf y son menos legibles

### El SDK más estable a través de versiones

🥇 **OpenAI**: API estable desde hace años con minor changes
🥈 **Anthropic**: estable, evoluciones documentadas
🥉 **Google**: cambios más frecuentes entre versiones del SDK

## Costos (referencia inicios de 2025)

Para 1 millón de tokens:

| Modelo | Input | Output | Notas |
|--------|-------|--------|-------|
| Claude Opus | $15.00 | $75.00 | Top tier, calidad máxima |
| Claude Sonnet 3.5 | $3.00 | $15.00 | Balance calidad/precio |
| Claude Haiku | $0.80 | $4.00 | Más rápido, más barato |
| GPT-4o | $2.50 | $10.00 | Top de OpenAI |
| GPT-4o-mini | $0.15 | $0.60 | Económico |
| Gemini 1.5 Pro | $1.25 | $5.00 | Muy competitivo |
| Gemini 1.5 Flash | $0.075 | $0.30 | El más barato del trío |

> ⚠️ **Cuidado**
>
> **Los precios cambian.** Verifica antes de comprometerte. Las URLs oficiales:
>
> - [anthropic.com/pricing](https://www.anthropic.com/pricing)
> - [openai.com/pricing](https://openai.com/pricing)
> - [ai.google.dev/pricing](https://ai.google.dev/pricing)

## Ecosistema y comunidad

| Aspecto | Claude | GPT-4 | Gemini |
|---------|--------|-------|--------|
| **Frameworks que lo soportan** | LangChain, LlamaIndex, MCP | Todos (estándar de facto) | LangChain, otros |
| **Stack Overflow / GitHub** | Creciendo rápido | Enorme | Mediano |
| **Documentación oficial** | Excelente | Buena | Inconsistente |
| **Cookbook con ejemplos** | Sí, muy bueno | Sí, exhaustivo | Sí |
| **Plataformas que lo embeben** | Cursor, Zed, Continue | Casi todas | Gradualmente |

## Mi recomendación (sesgada y subjetiva)

### Para aprender / prototipar
**Empieza con Claude.** La estructura del Tool Use es la más clara y educativa. Si entiendes Claude, GPT y Gemini te van a parecer fáciles.

### Para producción simple
**GPT-4o-mini o Gemini Flash.** Costo bajo, suficiente para clasificación, extracción y workflows simples.

### Para tareas críticas que requieren razonamiento
**Claude Sonnet/Opus o GPT-4o.** Más caros pero menos errores en decisiones complejas.

### Para procesar documentos enormes
**Gemini 1.5 Pro.** El contexto de 2M tokens es genuinamente útil cuando tienes documentos largos.

### Para apps multimodales (video/audio)
**Gemini.** Líder claro en este momento.

### Si no sabes y necesitas decidir hoy
**Claude Sonnet.** El balance calidad/precio/ergonomía es lo más balanceado para casos generales.

## Cuándo usar más de uno

A veces tiene sentido **combinar** modelos en un mismo sistema:

- **Modelo barato para clasificación** (Haiku, Mini, Flash)
- **Modelo capaz para decisiones complejas** (Opus, GPT-4o, Pro)
- **Modelo especializado** para tareas específicas (Gemini para video, etc.)

Esto se llama **routing** o **cascada de modelos**: usas el más barato primero, y solo escalas al caro cuando el barato no es suficiente.

## El antipatrón: vendor lock-in temprano

❌ **No te enganches con un proveedor antes de validar tu caso.**

Construye tu agente con la **abstracción mínima necesaria**:
- Define tus tools una vez
- Tu loop puede vivir detrás de una interfaz común
- El "cuál LLM" debería ser configurable

Si haces eso, cambiar de proveedor cuando sea necesario es cuestión de horas, no semanas.

## Lo que aprendiste

- ✅ Las tres APIs hacen lo mismo conceptualmente, con estructuras distintas
- ✅ Cada una tiene fortalezas específicas (contexto, costo, ergonomía, multimodalidad)
- ✅ La elección depende del caso, no del marketing
- ✅ Combinar modelos es una opción válida y a menudo óptima
- ✅ Sabes cómo evaluar técnicamente cuál usar

## Recapitulando los 3 casos del workshop

| Caso | Stack | Lo que enseña |
|------|-------|---------------|
| **Caso 1** | n8n (low-code) | Que los conceptos no requieren código. La intuición visual. |
| **Caso 2** | Python + Claude | Cómo funciona Tool Use en código real. El loop completo. |
| **Caso 3** | Python + Claude + GPT + Gemini | Diferencias entre proveedores. Criterio de elección. |

Si terminaste los tres, tienes una base sólida para construir agentes en cualquier contexto.

---

**Has terminado el workshop.** Pero el libro tiene más. Sigue con los apéndices para profundizar en glosarios, recursos y prompts.

**Siguiente: [A · Glosario →](../../apendices/a-glosario.md)**
