# Troubleshooting

Tu agente no funciona como esperabas. Bienvenido al 90% del trabajo real con agentes.

Esta es una guía de los problemas más comunes y cómo resolverlos.

## Errores en cada nodo

### Gmail Trigger no devuelve correos

**Síntoma**: ejecutas el trigger y el output es vacío.

**Causas posibles y soluciones**:

1. **No tienes correos no leídos**
   - Solución: marca como no leído algún correo de prueba y vuelve a ejecutar
   
2. **El filtro está demasiado restrictivo**
   - Revisa los filtros configurados (From, Label, etc.)
   - Empieza sin filtros adicionales para confirmar que el trigger funciona, luego refina

3. **OAuth de Gmail caducó**
   - Ve a Credentials → Gmail → Reconnect
   - Vuelve a hacer el flujo OAuth completo

4. **Estás en un trigger configurado pero no lo activaste**
   - Si el workflow está inactivo, los triggers no corren automáticamente
   - Pero **Test step** sí debería traerte correos
   - Verifica que estés haciendo `Test step` en el nodo Gmail Trigger, no `Test workflow`

### AI Agent devuelve error o respuesta inesperada

**Síntoma**: el nodo AI Agent se pone rojo.

**Causas comunes**:

1. **"AuthenticationError" / "Invalid API key"**
   - Tu credencial de Anthropic está mal
   - Verifica que la key sea válida en [console.anthropic.com](https://console.anthropic.com)
   - Recrea la credencial en n8n con la key correcta

2. **"Model not found"**
   - El nombre del modelo es incorrecto
   - Usa el nombre exacto que aparece en la documentación de Anthropic
   - Si `claude-opus-4-5` no funciona, prueba con `claude-3-5-sonnet-20241022` o el modelo más reciente

3. **"Rate limit exceeded"**
   - Estás haciendo muchas llamadas en poco tiempo
   - Espera unos minutos y vuelve a intentar
   - Si pasa frecuentemente, sube el plan de Anthropic o reduce la frecuencia del trigger

4. **El output es texto narrativo, no JSON**
   - El LLM ignoró la instrucción de responder solo en JSON
   - Refuerza el prompt: agrega "**RESPONDE ÚNICAMENTE el objeto JSON. NO incluyas ```json ni ningún texto antes o después.**"
   - Considera bajar la temperatura a `0.1` para mayor consistencia

### El nodo Code falla al parsear JSON

**Síntoma**: error tipo `SyntaxError: Unexpected token`.

**Causa**: el LLM devolvió algo que no es JSON válido. Casi siempre porque agregó texto adicional o markdown.

**Solución**: el código que te di ya intenta limpiar markdown ```json. Si aún falla, agrega más limpieza:

```javascript
const respuestaTexto = $input.first().json.output || $input.first().json.text;

let parsed;
try {
  parsed = JSON.parse(respuestaTexto);
} catch (e) {
  // Limpieza más agresiva
  let limpio = respuestaTexto;
  
  // Quitar markdown
  limpio = limpio.replace(/```json/gi, '').replace(/```/g, '');
  
  // Buscar el primer { y el último } para extraer solo el objeto
  const inicio = limpio.indexOf('{');
  const fin = limpio.lastIndexOf('}');
  
  if (inicio === -1 || fin === -1) {
    throw new Error("No se encontró JSON válido en la respuesta del LLM: " + respuestaTexto);
  }
  
  limpio = limpio.substring(inicio, fin + 1);
  parsed = JSON.parse(limpio);
}

return [{
  json: {
    ...parsed,
    correo_original: $('Gmail Trigger').first().json
  }
}];
```

### Switch no rutea bien

**Síntoma**: todos los correos van por la misma ruta, o ninguno sale del Switch.

**Causa**: la condición no coincide con el valor real.

**Solución**:

1. Ejecuta hasta el nodo Code y mira el output exacto
2. Verifica que `clasificacion` venga con las strings exactas (`URGENTE`, no `Urgente` ni `urgent`)
3. Si el LLM varía mayúsculas/minúsculas, normaliza en el nodo Code antes:

```javascript
parsed.clasificacion = parsed.clasificacion.toUpperCase().trim();
```

### Notion no crea la tarea

**Síntoma**: el nodo Notion se pone rojo, error "could not find database" o "object_not_found".

**Causas y soluciones**:

1. **No compartiste la DB con la integración**
   - El error más común
   - Ve a tu DB en Notion → `...` → Connections → agrega tu integración
   
2. **Estás referenciando un campo que no existe**
   - Los nombres de propiedades en Notion son **case-sensitive**
   - Si tu campo se llama "Title" pero pones "title", falla
   
3. **El tipo del campo no coincide**
   - Si "Status" en tu Notion es un Select, debes pasar el nombre exacto de una opción que ya exista
   - Si pasas "Pendiente" pero solo tienes opciones "To Do", "Doing", "Done" → falla
   - Crea las opciones primero en Notion, o usa una que ya exista

### Slack no manda mensaje

**Síntoma**: nodo Slack rojo.

**Causas**:

1. **El bot no está en el canal**
   - Solución: en Slack, abre el canal → integraciones → agrega la app de n8n
   - O envía a un canal donde el bot ya esté (ej. `#general` si ya está ahí)

2. **Permisos insuficientes**
   - Pide a tu admin de Slack que apruebe los scopes que n8n necesita
   - Mínimo: `chat:write`

3. **Canal no existe o nombre mal escrito**
   - Verifica el nombre del canal con # incluido (`#mi-canal`)
   - O usa el ID del canal directamente

## El AI Agent clasifica mal

**Síntoma**: clasifica spam como urgente, o newsletters como normal.

**Soluciones**:

### 1 · Mejora el system prompt con ejemplos

Agrega ejemplos few-shot al prompt:

```
EJEMPLOS:

Correo: De jefe@empresa.com, asunto "Necesito tu reporte HOY", cuerpo "Por favor, manda el reporte antes de las 5pm"
Clasificación: URGENTE
Razón: Solicitud específica con fecha límite del jefe directo

Correo: De newsletter@medium.com, asunto "Top 10 articles this week"
Clasificación: SPAM
Razón: Newsletter automatizado sin acción requerida

Correo: De colega@empresa.com, asunto "Reunión cambiada de las 10 a las 11"
Clasificación: NORMAL
Razón: Comunicación informativa sin acción urgente
```

Tres o cuatro ejemplos suelen mejorar dramáticamente la precisión.

### 2 · Baja la temperatura

A `0.1` (en el nodo Anthropic Chat Model). Esto hace al LLM más consistente.

### 3 · Pide razonamiento explícito

Modifica el JSON para incluir `"razonamiento_paso_a_paso"` antes de la clasificación. Esto fuerza al LLM a pensar antes de etiquetar.

```json
{
  "razonamiento_paso_a_paso": "string con tu análisis",
  "clasificacion": "...",
  ...
}
```

### 4 · Usa un modelo más capaz

Si estás usando un modelo barato (Haiku, GPT-3.5), prueba con uno más capaz (Sonnet, Opus, GPT-4). La diferencia en clasificación de matices puede ser grande.

## El workflow corre pero gasta mucho dinero

**Síntoma**: tu factura de Anthropic crece más rápido de lo esperado.

**Causas comunes**:

1. **Trigger demasiado frecuente**
   - Cada 15 min puede ser excesivo
   - Considera cada 30 min, 1h, o solo en horas laborales (puedes agregar un nodo `IF` que compruebe la hora)

2. **El LLM procesa correos largos completos**
   - El cuerpo de un email puede ser miles de tokens
   - Solución: usa `snippet` en lugar de `body` (snippet es ~150 caracteres)
   - O trunca: `{{ $json.body.substring(0, 1000) }}`

3. **El AI Agent itera muchas veces**
   - Por defecto el AI Agent puede entrar en loop con tools
   - Para clasificación simple **no necesitas** el loop con tools — usa solo Chat Model directo, sin AI Agent

4. **Hay un loop infinito accidental**
   - Revisa **Executions**: ¿hay ejecuciones que nunca terminan?
   - Configura `Max iterations` bajo en el AI Agent (5 o 10)

## Cómo debuggear cualquier problema

### El método universal

1. **Ejecuta paso a paso**: cada nodo tiene **Test step**. Verifica el output de cada uno antes de pasar al siguiente.
2. **Lee el error completo**: n8n muestra el error en el panel inferior. Copia el mensaje exacto.
3. **Revisa Executions**: ve al historial de ejecuciones. Cada ejecución muestra qué nodo falló y por qué.
4. **Pregúntale a un LLM**: literal — pega el error y el contexto a Claude o ChatGPT. El 80% de las veces te explica qué pasa.
5. **Revisa los logs del proveedor**: si el error es de Anthropic/Slack/Notion, ve al panel de cada uno y revisa si registran el intento.

### Cuando estás muy perdido

A veces lo mejor es **empezar de nuevo con un workflow más simple**:

1. Crea un workflow nuevo
2. Solo Gmail Trigger → AI Agent (sin tools, solo clasificar)
3. Confirma que ESO funciona
4. Agrega un nodo a la vez

Esto te ayuda a aislar dónde está el problema.

## Lo que aprendiste

- ✅ A leer errores y rastrear su causa
- ✅ Patrones comunes de falla en cada nodo
- ✅ Cómo iterar el prompt del LLM cuando clasifica mal
- ✅ Cómo controlar costos
- ✅ El método de debugging universal

> 💡 **Tip final**
>
> En agentes reales en producción, **algo va a fallar** todas las semanas. No es señal de que hayas fallado — es señal de que estás trabajando con sistemas estocásticos. Aprender a debuggear es una de las habilidades más valiosas que te llevas del workshop.

---

**Has terminado el Caso 1.** Tu primer agente real está funcionando.

**Siguiente: [Caso 2 · Python + Claude Tool Use →](../caso-2-claude/README.md)**
