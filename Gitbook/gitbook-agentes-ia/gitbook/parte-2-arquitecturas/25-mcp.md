# 2.5 · MCP · Model Context Protocol

## El problema que resuelve

Imagina que tienes 3 LLMs distintos (Claude, GPT, Gemini) y quieres conectarlos a 3 herramientas distintas (Slack, GitHub, Gmail).

**Sin estándar**, necesitas escribir **3 × 3 = 9 integraciones** custom. Cada una con su API, su formato, su autenticación, sus errores específicos.

```
LLMs        Tools
Claude  ─→  Slack   (integración custom 1)
Claude  ─→  GitHub  (integración custom 2)
Claude  ─→  Gmail   (integración custom 3)
GPT     ─→  Slack   (integración custom 4)
GPT     ─→  GitHub  (integración custom 5)
...
```

Eso es un problema **M × N**: cada nuevo LLM o cada nueva herramienta multiplica el trabajo.

## Qué es MCP

**MCP** = **Model Context Protocol** — protocolo abierto (creado por Anthropic, adoptado por la industria) que estandariza **cómo los LLMs hablan con herramientas**.

La idea: definir **una sola vez** la interfaz entre LLMs y tools. Cualquier LLM que hable MCP puede conectarse a cualquier tool que hable MCP.

```
LLMs                 MCP                Tools
Claude  ─┐                          ┌─→ Slack
GPT     ─┼──→  [protocolo único] ──┼─→ GitHub
Gemini  ─┘                          └─→ Gmail
```

Ahora el problema es **M + N**: cada nuevo LLM solo necesita "hablar MCP" (una integración), y cada nueva herramienta solo necesita "exponerse vía MCP" (una integración).

## La analogía

MCP es **el USB-C de los agentes de IA**.

Antes de USB-C, cada dispositivo tenía su propio cable. Ahora un solo cable conecta todo. MCP busca el mismo efecto en el mundo de la IA.

## Cómo funciona

### Servidores MCP

Una herramienta que quiere ser usable por agentes **expone un servidor MCP**. Este servidor:

- Lista qué herramientas/recursos ofrece
- Define los esquemas de inputs/outputs
- Maneja la autenticación
- Ejecuta las llamadas y devuelve resultados

### Clientes MCP

Un LLM (o el código que orquesta al LLM) actúa como **cliente MCP**:

- Se conecta a uno o varios servidores MCP
- Pide la lista de tools disponibles
- Las pasa al LLM como herramientas
- Cuando el LLM decide invocar una, el cliente la ejecuta vía MCP
- Devuelve el resultado al LLM

### Tipos de servidores MCP

| Tipo | Cómo funciona | Cuándo usar |
|------|---------------|-------------|
| **Local (stdio)** | Servidor corre en tu máquina, cliente habla por stdin/stdout | Tools que acceden a tu sistema (filesystem, terminal) |
| **Remoto (HTTP/SSE)** | Servidor corre en la nube, cliente se conecta vía URL | Tools que viven en servicios web (Slack, GitHub, etc.) |

## Ejemplos reales

### Servidores MCP públicos

Hay servidores MCP ya construidos para muchos servicios populares:

- **GitHub** — leer issues, crear PRs, comentar
- **Slack** — enviar mensajes, leer canales
- **Notion** — crear/editar páginas
- **Filesystem** — leer/escribir archivos en tu disco
- **Postgres** — consultar bases de datos

Puedes conectar tu agente a cualquiera de estos sin escribir el código de integración.

### Crear tu propio servidor MCP

Si tu empresa tiene una API interna, puedes envolverla en un servidor MCP. Una vez listo, **cualquier agente compatible** (Claude Desktop, Cursor, tu agente custom) puede usarla sin más trabajo.

## Cómo se ve usar MCP

Sin MCP, conectar tu agente a Slack requeriría:

```python
# Sin MCP - integración custom
import slack_sdk
client = slack_sdk.WebClient(token=SLACK_TOKEN)

def enviar_mensaje_slack(canal, texto):
    return client.chat_postMessage(channel=canal, text=texto)

# Defines la tool al estilo Anthropic
tool = {
    "name": "enviar_mensaje_slack",
    "description": "Envía un mensaje a un canal de Slack",
    "input_schema": {...}
}

# Implementas el handler
def handle_tool(name, input):
    if name == "enviar_mensaje_slack":
        return enviar_mensaje_slack(input['canal'], input['texto'])
    # ... otros casos
```

Con MCP, conceptualmente:

```python
# Con MCP - solo te conectas al servidor
from mcp_client import MCPClient

mcp = MCPClient()
mcp.connect_server("slack-mcp-server", url="...")

# Las tools de Slack ya están disponibles automáticamente
# El cliente MCP las pasa al LLM en formato compatible
tools_disponibles = mcp.list_tools()  # incluye todas las de Slack

# Cuando el LLM las invoca, MCP las ejecuta
```

> 📌 **Nota**
>
> El ecosistema MCP está evolucionando rápido. Las APIs específicas de las librerías cliente cambian. Los conceptos del protocolo son estables — pero los detalles de implementación los aprenderás cuando lo uses, no antes.

## Por qué importa

### Para desarrolladores

- **Menos código de integración custom**
- **Reusas servidores MCP** que otros ya hicieron
- **Un solo agente sirve a múltiples sistemas** sin re-escritura

### Para empresas

- **Estandarización** — equipos diferentes hablan el mismo lenguaje técnico
- **Vendor lock-in reducido** — si cambias de LLM, las tools siguen funcionando
- **Seguridad centralizada** — autenticación y permisos en el servidor MCP, no en cada integración

### Para el ecosistema

- **Marketplace de tools** — empresas publican servidores MCP, otras los consumen
- **Composabilidad** — tu agente combina tools de orígenes muy distintos sin esfuerzo

## Estado actual

A finales de 2024 / inicios de 2025, MCP es **adopción temprana**:

- **Anthropic** lo lanzó y lo soporta nativamente en Claude Desktop
- **Cursor**, **Continue**, **Zed** lo soportan en sus IDEs
- Muchos servidores MCP públicos en GitHub
- Aún no es soportado nativamente por OpenAI o Google (puedes hacer puentes)

Es razonable esperar que **MCP se vuelva el estándar de facto** en los próximos 1-2 años, similar a como REST se volvió estándar para APIs web.

## Cuándo te impacta hoy

✅ **MCP te interesa hoy si:**

- Quieres conectar Claude Desktop a herramientas (es trivial con MCP)
- Construyes agentes que necesitan muchas integraciones
- Quieres exponer tus APIs internas a múltiples LLMs sin re-escribir

❌ **MCP no es prioritario si:**

- Estás haciendo tu primer agente con 1-2 tools simples
- Tu integración con servicios externos es muy específica
- Estás aprendiendo conceptos básicos (como en este workshop)

## En el workshop

**No usamos MCP en el workshop principal.** Para los Casos 2 y 3 implementamos las tools directamente en código — es más educativo para entender el loop fundamental.

Una vez termines el workshop y tengas el patrón claro, MCP es la siguiente capa que vale la pena explorar.

## Recursos para profundizar

- [Especificación oficial · modelcontextprotocol.io](https://modelcontextprotocol.io)
- [Repositorio de servidores MCP públicos · github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)
- Apéndice B de este libro tiene más recursos

## Recapitulando

- **MCP = protocolo estándar** para conectar LLMs con tools
- Resuelve el problema **M × N** de integraciones custom (lo convierte en M + N)
- Analogía útil: el **USB-C de los agentes**
- Adopción temprana pero creciendo rápido
- **No es necesario** para empezar — pero vale la pena saber que existe

---

**Has terminado la Parte 2.** Ahora viene la mejor parte: **construir agentes reales con tus manos**.

**Siguiente: [Caso 1 · Agente personal en n8n →](../parte-3-workshop/caso-1-n8n/README.md)**
