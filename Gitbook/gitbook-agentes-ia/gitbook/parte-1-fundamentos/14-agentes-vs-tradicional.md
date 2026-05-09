# 1.4 · Agentes vs automatización tradicional

## La pregunta que nadie te hace pero deberías

Cuando alguien te dice "automatízame esto con IA", la primera pregunta interna debe ser:

> **¿Esto realmente necesita IA, o un script clásico es suficiente?**

Saber responder esta pregunta te ahorra meses de trabajo y miles de dólares en facturas de API.

## Las dos formas de automatizar

### Automatización tradicional

Scripts, RPA (Robotic Process Automation), workflows clásicos.

**Funciona así**: tú describes los pasos en código o reglas. El sistema los ejecuta exactamente como los definiste.

```python
# Ejemplo de automatización tradicional
if archivo.extension == ".pdf":
    convertir_a_word(archivo)
elif archivo.extension == ".jpg":
    redimensionar(archivo, 800, 600)
else:
    ignorar(archivo)
```

**Características**:
- Determinista: misma entrada → misma salida, siempre
- Predecible: sabes exactamente qué va a hacer
- Barato: no consume tokens, no llama a APIs caras
- Frágil ante cambios: si el formato del input cambia, se rompe

### Automatización agéntica

Agentes de IA que **deciden los pasos en el momento**.

**Funciona así**: tú das un objetivo y herramientas. El agente decide cómo combinarlas.

```python
# Ejemplo de automatización agéntica (pseudocódigo)
agente.objetivo = "Procesa este documento y dame un resumen ejecutivo"
agente.tools = [extraer_texto, analizar_imagenes, generar_resumen]
resultado = agente.ejecutar(documento)
# El agente decide qué herramientas usar y en qué orden
```

**Características**:
- Adaptativo: maneja casos no previstos
- Flexible: el mismo agente sirve para variantes del problema
- Costoso: cada decisión es una llamada al LLM
- Menos predecible: dos ejecuciones pueden diferir en el camino tomado

## Comparativa lado a lado

| Aspecto | Automatización tradicional | Agente de IA |
|---------|---------------------------|--------------|
| **¿Quién define los pasos?** | Tú, en código | El agente, en runtime |
| **¿Maneja ambigüedad?** | No | Sí |
| **¿Decide en ejecución?** | No, sigue el script | Sí, según contexto |
| **Costo por ejecución** | Bajo (centavos) | Medio-alto (decenas de centavos a dólares) |
| **Velocidad** | Rápido (ms) | Lento (segundos) |
| **Predictibilidad** | Alta | Media |
| **Debugging** | Fácil | Difícil |
| **Casos de uso ideal** | Procesos repetitivos y bien definidos | Procesos con juicio, variabilidad o contexto |

## Ejemplos comparados

### Ejemplo 1 · Procesar facturas

**Automatización tradicional sirve cuando:**
- Las facturas vienen en un formato fijo
- Los campos están en posiciones predecibles
- Las reglas de negocio son claras

**Agente sirve cuando:**
- Las facturas vienen en formatos variados (PDFs, imágenes, escaneos)
- Hay que interpretar texto manuscrito o ambiguo
- Hay que aplicar excepciones según el contexto

### Ejemplo 2 · Clasificar correos

**Automatización tradicional sirve cuando:**
- Las reglas son binarias y claras (asunto contiene "factura" → clasificar como financiero)
- Hay pocos remitentes conocidos

**Agente sirve cuando:**
- El contenido es libre y variado
- Las reglas requieren interpretar tono, urgencia o contexto
- Los remitentes son desconocidos y los temas variados

### Ejemplo 3 · Aprobar reembolsos

**Automatización tradicional sirve cuando:**
- Existe una política clara: monto < X, automático; monto > Y, escalar
- Los datos vienen en formato estructurado

**Agente sirve cuando:**
- Hay que interpretar la justificación del usuario
- Hay que considerar el historial completo del cliente
- Hay zona gris donde aplicar criterio

## El error caro: "vamos a hacer todo con IA"

El reflejo de hoy es: *"como tenemos LLMs, hagamos todo con LLMs"*. Es un error caro.

### Caso real (anecdótico pero típico)

Imagina una empresa que automatiza el procesamiento de tickets de soporte usando un agente con LLM en cada paso. Resultado:

- Costo por ticket: ~$0.40 USD
- 10,000 tickets/mes = **$4,000 USD mensuales**

Refactorización: solo el 15% de tickets requiere juicio del LLM. El 85% restante son patrones claros que un script clásico clasifica perfectamente.

Después de la refactorización:
- Solo el 15% pasa por el agente
- 1,500 × $0.40 = $600 USD
- 8,500 × $0.001 (script) = $8.50 USD
- **Total: ~$610 USD mensuales**

**Ahorro del 85%** sin pérdida de calidad.

> 💡 **Tip**
>
> El patrón ganador en producción se llama frecuentemente **"agentes en los bordes"**: usas IA solo en los puntos donde el código tradicional no puede manejar la ambigüedad. El resto sigue siendo código tradicional, barato y predecible.

## Cuándo se complementan

Lo más común es que la solución no sea "uno u otro" sino **una combinación**:

```
[Trigger del sistema]
    ↓
[Validación tradicional con if/else]
    ↓
[Si requiere juicio → Agente]
    ↓
[Resultado del agente]
    ↓
[Post-procesamiento tradicional]
    ↓
[Acción final]
```

El agente es **una pieza dentro de un flujo más grande**, no el flujo entero.

## El test de los 30 segundos

Antes de decidir si usar un agente, hazte estas tres preguntas:

1. **¿Puedo escribir las reglas exactas del proceso en una hoja de papel?**
   - Sí → Usa código tradicional
   - No, hay variaciones → Considera agente

2. **¿El input siempre tiene la misma estructura?**
   - Sí → Usa código tradicional
   - No, varía mucho → Considera agente

3. **¿El proceso requiere interpretar lenguaje natural, contexto, o tono?**
   - No → Usa código tradicional
   - Sí → Probablemente necesitas un agente

Si responde "sí, sí, no" → automatización tradicional. Si "no, no, sí" → agente. Lo intermedio es donde puedes combinar ambos.

## Recapitulando

- **Automatización tradicional**: tú defines los pasos, el sistema los ejecuta. Predecible, barata, frágil ante cambios.
- **Agente**: el sistema decide los pasos en runtime. Adaptable, flexible, más caro y menos predecible.
- **El patrón ganador**: combina ambos. Usa código clásico para lo predecible, agentes solo donde hace falta juicio.
- **El test rápido**: si puedes escribir el proceso como reglas, no necesitas agente.

> 🎯 **Reto**
>
> De los procesos que tu empresa (o tu vida) automatiza o quisiera automatizar, lista 5. Para cada uno, decide: ¿script tradicional, agente, o combinación? Justifica con los criterios de este capítulo.

---

**Siguiente: [1.5 · Cuándo NO usar agentes →](15-cuando-no-usar.md)**
