# PLAN DE TRABAJO DEFINITIVO - Workflow Podium V7

## Estado: LISTO PARA IMPLEMENTAR
## Fecha: 2026-02-03

---

## RESUMEN DE CAMBIOS

El workflow actual ("Podium Enterprise V6 - 9 Categorias FINAL") tiene 60 nodos.
Se realizaran 12 pasos de correccion sobre el JSON existente.

**Principio rector**: NO se reconstruye. Se corrige el workflow existente.

---

## ARQUITECTURA ACTUAL vs NUEVA

### ACTUAL (INCORRECTA):
```
Gmail Trigger
  -> Datos (extractor)
  -> Basic LLM Chain (RESIDENTE/EXTERNO?)
  -> OpenAI Chat Model (sub-nodo del LLM)
  -> Restore Context (recupera binary)
  -> If (es RESIDENTE?)
      SI -> Code Resolver Apto -> IF#1 (tiene apto?)
              SI -> Code Prep Agent -> AI Agent -> Parser -> Router
              NO -> Gmail Pedir Apto -> Etiqueta Incompleto -> Log Rechazo
      NO -> Code Prep Agent -> AI Agent -> Parser -> Router
```

**Problema**: Pide apartamento ANTES de clasificar. Un proveedor recibiria "envie su numero de apartamento".

### NUEVA (CORRECTA):
```
Gmail Trigger
  -> Datos (extractor)
  -> Code Resolver Apto (intenta detectar apto, sin bloquear)
  -> IF#0 (es loop malo?)
      SI -> Etiquetar Manual -> Alerta Admin -> Log Manual -> FIN
      NO -> Code Prep Agent -> AI Agent -> Parser -> Router (equals)
            |
            Router envia a 9 ramas:
            |
            0: LEGAL        -> Prep Forward -> ACK -> Forward -> Label -> Log
            1: CCTV         -> Prep Forward -> ACK -> Forward -> Label -> Log
            2: CONVIVENCIA  -> IF Apto? -> ACK -> Forward -> Label -> Log
            3: CONTABILIDAD -> Prep Forward -> ACK -> Cache/Daytona -> Forward -> Label -> Log
            4: FACTURAS     -> Prep Forward -> ACK -> Forward -> Label -> Log
            5: MANTENIMIENTO-> IF Apto? -> ACK -> Forward -> Label -> Log
            6: TRASTEOS     -> IF Apto? -> ACK -> Forward -> Label -> Log
            7: SALON SOCIAL -> IF Apto? -> ACK -> Forward -> Label -> Log
            8: INFO GENERAL -> IF confianza >= 0.85 -> Send IA Reply / Telegram -> Label -> Log
```

---

## NODOS A ELIMINAR (8 nodos)

| # | Nombre Exacto en JSON | ID | Razon |
|---|---|---|---|
| 1 | `Basic LLM Chain` | (ver JSON) | Clasificacion RESIDENTE/EXTERNO innecesaria |
| 2 | `OpenAI Chat Model` | (sub-nodo del LLM) | Dependencia del nodo anterior |
| 3 | `Restore Context (after LLM)` | 67fbeaf2-... | Code Prep Agent ya recupera binary de Datos |
| 4 | `If` (RESIDENTE/EXTERNO) | 313c0574-... | Se elimina la bifurcacion RESIDENTE/EXTERNO |
| 5 | `IF #1 - Tiene Apto Valido?` | 0b63bcd0-... | Validacion de apto se mueve DESPUES del Router |
| 6 | `Gmail - Pedir Apto` | 8bccb18c-... | Se reemplaza por validaciones post-Router |
| 7 | `Gmail - Etiqueta Incompleto` | ab59ce41-... | Se reemplaza por validaciones post-Router |
| 8 | `Supabase - Log Rechazo` | a2e897db-... | Se reemplaza por validaciones post-Router |

---

## NODOS QUE SE CONSERVAN (sin cambios)

- Gmail Trigger (bef2497c)
- Datos / Code Extractor
- Code Prep Agent (6ef1043d) - YA recupera binary de "Datos" directamente
- AI Agent + OpenAI Model (el del Agent, NO el del LLM Chain)
- Vector Store Supabase + Embeddings
- IF Auto-Approve
- IF Telegram Approved
- Telegram Ask
- Send IA Reply (Info General)
- Todos los Gmail Forward existentes (Legal, CCTV, etc.)
- Todos los Gmail Label existentes
- Todos los Supabase Log existentes
- Cache Contabilidad + API Daytona + Guardar Cache

---

## PASOS DE IMPLEMENTACION

### PASO 1: Eliminar 8 nodos y reconectar

**Que hacer en n8n:**
1. Seleccionar y eliminar estos 8 nodos:
   - `Basic LLM Chain`
   - `OpenAI Chat Model` (el que esta conectado al Basic LLM Chain)
   - `Restore Context (after LLM)`
   - `If` (el nodo que dice RESIDENTE/EXTERNO)
   - `IF #1 - Tiene Apto Valido?`
   - `Gmail - Pedir Apto`
   - `Gmail - Etiqueta Incompleto`
   - `Supabase - Log Rechazo`

2. Reconectar:
   ```
   Datos [salida] --> Code - Resolver Apto (FINAL) [entrada]
   Code - Resolver Apto (FINAL) [salida] --> IF #0 - Es Loop Malo? [entrada]
   IF #0 FALSE [salida] --> Code - Prep Agent [entrada]
   ```

3. IF#0 TRUE ya esta conectado a:
   - Gmail - Etiquetar Manual
   - (verificar que la cadena Manual -> Alerta -> Log sigue intacta)

**Verificacion**: Despues de este paso, el flujo debe ser:
```
Gmail Trigger -> Datos -> Code Resolver Apto -> IF#0
  TRUE -> Etiquetar Manual -> ...
  FALSE -> Code Prep Agent -> AI Agent -> Parser -> Router -> ...
```

---

### PASO 2: Cambiar Router de "contains" a "equals"

**Que hacer en n8n:**
1. Abrir el nodo `Router`
2. Para CADA una de las 9 condiciones:
   - Cambiar operacion de "contains" a "equals"
3. Las 9 condiciones deben quedar:
   - Output 0: `ai_analysis.categoria` equals `TEMA_LEGAL_PQR`
   - Output 1: `ai_analysis.categoria` equals `CCTV_SEGURIDAD`
   - Output 2: `ai_analysis.categoria` equals `CONVIVENCIA`
   - Output 3: `ai_analysis.categoria` equals `CONTABILIDAD`
   - Output 4: `ai_analysis.categoria` equals `FACTURAS_PROVEEDORES`
   - Output 5: `ai_analysis.categoria` equals `MANTENIMIENTO`
   - Output 6: `ai_analysis.categoria` equals `TRASTEOS`
   - Output 7: `ai_analysis.categoria` equals `SALON_SOCIAL`
   - Output 8: `ai_analysis.categoria` equals `INFORMACION_GENERAL`

**Por que**: "contains" causaria que "CONTABILIDAD" tambien haga match con "FACTURAS_PROVEEDORES" si el texto contiene una dentro de otra, o que categorias parciales hagan match incorrecto.

---

### PASO 3: Corregir Parser (proteccion null)

**Que hacer en n8n:**
1. Abrir nodo `Code - Parser`
2. Reemplazar la primera linea del try:
   ```javascript
   // ANTES:
   const output = items[0].json.output;

   // DESPUES:
   if (!items || !items.length || !items[0]?.json?.output) {
     return [{ json: {
       ...((items && items[0]?.json) || {}),
       ai_analysis: {
         categoria: "INFORMACION_GENERAL",
         confianza: 0.1,
         resumen: "Sin respuesta del agente IA",
         borrador_respuesta: ""
       }
     } }];
   }
   const output = items[0].json.output;
   ```

---

### PASO 4: Actualizar nodos con typeVersion obsoleta

**Que hacer en n8n (via JSON):**

| Nodo | Version actual | Version correcta |
|---|---|---|
| `IF #0 - Es Loop Malo?` | typeVersion: 1 | typeVersion: 2.3 |
| `2. Existe Cache?` | typeVersion: 1 | typeVersion: 2.3 |
| `3. Consultar API Daytona` | typeVersion: 3 | typeVersion: 4.2 |

**IMPORTANTE**: Al actualizar IF de v1 a v2.3, la estructura de condiciones cambia.

**IF#0 v1 actual:**
```json
"conditions": {
  "boolean": [{
    "value1": "={{ /acci(o|ó)n requerida/i.test($json.subject...) }}",
    "value2": true
  }]
}
```

**IF#0 v2.3 nuevo:**
```json
"conditions": {
  "options": { "combinator": "and" },
  "conditions": [{
    "leftValue": "={{ /acci(o|ó)n requerida/i.test($json.subject || '') && !!$json.thread_id && !($json.apto_num && /\\d+/.test(String($json.apto_num))) }}",
    "rightValue": true,
    "operator": { "type": "boolean", "operation": "equals" }
  }]
}
```

**Nota**: Si la actualizacion de version causa problemas con las condiciones, se puede dejar en v1 por ahora y actualizar despues. La prioridad es la correccion arquitectural (pasos 1-3).

---

### PASO 5: Corregir referencia API Daytona

**Que hacer en n8n:**
1. Abrir nodo `3. Consultar API Daytona`
2. En el parametro Query:
   - **ANTES**: `={{ $('Code - Resolver Apto FINAL').first().json.aptonum }}`
   - **DESPUES**: `={{ $('Code - Resolver Apto (FINAL)').first().json.apto_num }}`

   Errores corregidos:
   - Nombre del nodo: faltaban los parentesis `(FINAL)` vs `FINAL`
   - Campo: `aptonum` vs `apto_num` (con guion bajo)

---

### PASO 6: Corregir expresiones =={{ a ={{

**Que hacer**: Buscar y reemplazar en el JSON todas las instancias de `=={{` por `={{`

Hay 104 instancias. En n8n, `={{` es la sintaxis correcta para expresiones. El prefijo `==` es invalido y puede causar comportamiento impredecible.

**Metodo**:
1. Exportar workflow como JSON
2. Abrir en editor de texto
3. Buscar y reemplazar `=={{` por `={{`
4. Verificar que no se rompio ninguna expresion
5. Re-importar

**EXCEPCION**: NO reemplazar si `==` es parte de una comparacion JavaScript dentro de la expresion. Ejemplo:
- `={{ $json.status == 'active' }}` - aqui el `==` es JavaScript, NO prefijo de expresion
- Verificar cada caso

---

### PASO 7: Agregar nodos "Code Prep Forward" antes de cada Gmail Forward

**Por que**: El AI Agent no pasa binary (adjuntos). Para que los Gmail Forward envien emails con los adjuntos originales, necesitamos un nodo Code que recupere el binary del nodo "Datos".

**Patron a repetir para cada rama (Legal, CCTV, Convivencia, etc.):**

```javascript
// Code Prep Forward - [NOMBRE_CATEGORIA]
// Recupera adjuntos originales del nodo "Datos" para el Gmail Forward

const datosNode = $('Datos').first();
const aiAnalysis = $json.ai_analysis || {};

return [{
  json: {
    from_email: datosNode.json.from_email || datosNode.json.fromemail || '',
    subject: datosNode.json.subject || datosNode.json.Subject || '',
    body: datosNode.json.body || '',
    nombre_residente: datosNode.json.nombre_residente || datosNode.json.nombreresidente || '',
    apartamento: $json.apto_num || $json.apartamento || datosNode.json.apartamento || '',
    apto_num: $json.apto_num || '',
    categoria: aiAnalysis.categoria || '',
    confianza: aiAnalysis.confianza || 0,
    resumen: aiAnalysis.resumen || '',
    email_id: datosNode.json.id || '',
    thread_id: datosNode.json.threadId || ''
  },
  binary: datosNode.binary || {}
}];
```

**Donde insertar (entre Router y Gmail Forward):**

| Rama | Insertar entre | Y |
|---|---|---|
| LEGAL | Router Output 0 | Gmail ACK Legal |
| CCTV | Router Output 1 | Gmail ACK CCTV |
| CONVIVENCIA | Router Output 2 | Gmail Convivencia |
| CONTABILIDAD | Router Output 3 | 1. Verificar Cache |
| FACTURAS | Router Output 4 | Gmail Facturas Prov |
| MANTENIMIENTO | Router Output 5 | Gmail Mantenimiento |
| TRASTEOS | Router Output 6 | Gmail Trasteos |
| SALON SOCIAL | Router Output 7 | Gmail Salon Social |

**Total**: 8 nodos "Code Prep Forward" nuevos.

**Configuracion Gmail Forward (adjuntos):**
En cada nodo Gmail Forward, agregar:
- Options -> Add Option -> Attachments -> Add Attachment
- Attachment Field Name: `{{ Object.keys($binary).join(',') }}`

Esto envia TODOS los adjuntos que tenia el email original.

---

### PASO 8: Agregar validacion de apartamento POST-Router (4 categorias)

**Categorias que REQUIEREN apartamento:**
- CONVIVENCIA (Output 2)
- MANTENIMIENTO (Output 5)
- TRASTEOS (Output 6)
- SALON SOCIAL (Output 7)

**Categorias que NO requieren apartamento:**
- LEGAL (Output 0) - puede venir de abogado externo
- CCTV (Output 1) - puede venir de admin/seguridad
- CONTABILIDAD (Output 3) - tiene su propia validacion via Daytona
- FACTURAS (Output 4) - viene de proveedores
- INFO GENERAL (Output 8) - consulta abierta

**Para cada una de las 4 categorias, agregar DESPUES del Code Prep Forward:**

```
Code Prep Forward [Cat] -> IF Tiene Apto [Cat]?
  SI -> Gmail ACK [Cat] -> Gmail Forward [Cat] -> Label -> Log
  NO -> Gmail Pedir Apto [Cat] -> Label DATOS_INCOMPLETOS -> Log
```

**Nodo IF Tiene Apto (v2.3):**
- Condicion: `{{ !!$json.apto_num && /\d+/.test(String($json.apto_num)) }}`
- Tipo: Boolean equals true

**Nodo Gmail Pedir Apto (generico, reutilizable):**
- To: `={{ $json.from_email }}`
- Subject: `Accion Requerida: Datos faltantes en tu solicitud - Podium P.H.`
- Message: (usar la misma plantilla del nodo eliminado "Gmail - Pedir Apto", linea 1622 del JSON original)

**IMPORTANTE**: Mantener "Accion Requerida" en el subject para que IF#0 (loop detection) siga funcionando cuando el usuario responda.

---

### PASO 9: Agregar nodos ACK faltantes

**ACKs que ya existen:**
- Gmail ACK Legal (ya existe)
- Gmail ACK CCTV (ya existe)

**ACKs que faltan (6 nuevos):**

| # | Nombre | To | Subject |
|---|---|---|---|
| 1 | Gmail ACK Convivencia | `={{ $json.from_email }}` | Reporte de convivencia recibido - Podium P.H. |
| 2 | Gmail ACK Contabilidad | `={{ $json.from_email }}` | Solicitud contable recibida - Podium P.H. |
| 3 | Gmail ACK Facturas | `={{ $json.from_email }}` | Factura recibida - Podium P.H. |
| 4 | Gmail ACK Mantenimiento | `={{ $json.from_email }}` | Solicitud de mantenimiento recibida - Podium P.H. |
| 5 | Gmail ACK Trasteos | `={{ $json.from_email }}` | Solicitud de trasteo recibida - Podium P.H. |
| 6 | Gmail ACK Salon | `={{ $json.from_email }}` | Solicitud de salon social recibida - Podium P.H. |

**Plantilla de mensaje ACK (adaptar por categoria):**
```html
<p>Hola,</p>
<p>Confirmamos la recepcion de su solicitud.</p>
<p><strong>Categoria:</strong> [NOMBRE_CATEGORIA]</p>
<p><strong>Resumen:</strong> {{ $json.resumen }}</p>
<p>Su solicitud ha sido enviada al area correspondiente. Sera atendida en el plazo establecido.</p>
<p>Cordialmente,<br><strong>Administracion Podium P.H.</strong></p>
```

**Posicion en el flujo:**
```
Code Prep Forward -> [IF Apto si aplica] -> Gmail ACK -> Gmail Forward -> Label -> Log
```

---

### PASO 10: Agregar validacion de datos CCTV y Mantenimiento

**Segun el documento operativo de Don Jorge:**

**CCTV requiere**: nombre, apartamento, fecha del evento, hora aproximada, zona/camara, motivo
**Mantenimiento requiere**: ubicacion exacta, descripcion clara, disponibilidad de ingreso

Estos nodos de validacion van ANTES del ACK en sus respectivas ramas:

```
Router Output 1 (CCTV) -> Code Prep Forward -> Code Validar Datos CCTV -> IF Datos Completos?
  SI -> Gmail ACK CCTV -> Gmail Forward CCTV -> Label -> Log
  NO -> Gmail Solicitar Datos CCTV -> Label CCTV_INCOMPLETO -> Log

Router Output 5 (Mant) -> Code Prep Forward -> [IF Apto?] -> Code Validar Datos Mant -> IF Datos Completos?
  SI -> Gmail ACK Mant -> Gmail Forward Mant -> Label -> Log
  NO -> Gmail Solicitar Datos Mant -> Label MANT_INCOMPLETO -> Log
```

**Codigo de validacion**: Ver archivo PLAN_COMPLETO_IMPLEMENTACION.md, seccion "Code - Validar Datos CCTV" y "Code - Validar Datos Mantenimiento".

---

### PASO 11: Verificar credenciales Gmail

**Hallazgo**: Hay 2 credenciales Gmail:
- `ZxAjG4YwQTaf4LJu` ("Gmail - Podium") - usada por 24 nodos
- `A9cSnQDIdWvgLcRL` - usada SOLO por Gmail Forward Legal y Gmail Forward CCTV

**Accion**: Verificar con Don Jorge si la segunda credencial es correcta o si todos los nodos deben usar "Gmail - Podium". Si es la misma cuenta, unificar.

---

### PASO 12: Pruebas

**Casos de prueba minimos:**

| # | Caso | Email simulado | Resultado esperado |
|---|---|---|---|
| 1 | Factura proveedor | "Adjunto factura de aseo febrero" | Clasifica FACTURAS, forward a contabilidad, NO pide apto |
| 2 | Residente con apto | "Soy del apto 501, hay una fuga" | Clasifica MANTENIMIENTO, detecta apto 501, ACK + forward |
| 3 | Residente sin apto | "Hay mucho ruido en el piso de arriba" | Clasifica CONVIVENCIA, no detecta apto, pide apto |
| 4 | Tutela/legal | "Por medio de la presente notifico tutela..." | Clasifica LEGAL, forward urgente, NO pide apto |
| 5 | Info general | "Cuales son los horarios de la piscina?" | Clasifica INFO_GENERAL, respuesta IA con RAG |
| 6 | CCTV con datos completos | "Necesito revision camaras parqueadero 20/01 3pm apto 301" | Clasifica CCTV, datos completos, forward |
| 7 | CCTV sin datos | "Necesito las camaras" | Clasifica CCTV, datos incompletos, solicita datos |
| 8 | Loop (respuesta sin apto) | Subject "Re: Accion Requerida..." sin apto | IF#0 detecta loop, etiqueta manual |
| 9 | Loop (respuesta CON apto) | Subject "Re: Accion Requerida..." con "apto 301" | IF#0 NO detecta loop, procesa normalmente |

---

## SITUACIONES LOGICAS IDENTIFICADAS

### 1. Loop infinito de "Pedir Apto" (RESUELTO)
- IF#0 detecta emails con subject "Accion Requerida" + sin apto = loop malo
- Si el usuario responde CON apto, IF#0 deja pasar y se procesa normal

### 2. Proveedor recibe "envie su apartamento" (RESUELTO)
- Se mueve validacion de apto DESPUES del Router
- Solo categorias de residentes (CONVIVENCIA, MANTENIMIENTO, TRASTEOS, SALON) piden apto

### 3. Adjuntos se pierden en AI Agent (RESUELTO)
- Code Prep Forward recupera binary de "Datos" via `$('Datos').first().binary`
- Gmail Forward usa `{{ Object.keys($binary).join(',') }}`

### 4. Router con "contains" causa matches incorrectos (RESUELTO)
- Cambio a "equals" en las 9 condiciones

### 5. Parser crash si AI Agent no responde (RESUELTO)
- Verificacion null antes de `items[0].json.output`

### 6. Expresiones =={{ invalidas (RESUELTO)
- 104 instancias a corregir con buscar/reemplazar

### 7. API Daytona con nombre de nodo incorrecto (RESUELTO)
- Corregir `Code - Resolver Apto FINAL` a `Code - Resolver Apto (FINAL)`
- Corregir `aptonum` a `apto_num`

### 8. Email propio del sistema re-procesado (YA RESUELTO)
- Gmail Trigger ya filtra: `-from:crcpodiumph@gmail.com`

### 9. Emails de Datos Incompletos re-procesados (YA RESUELTO)
- Gmail Trigger ya filtra: `-label:DATOS_INCOMPLETOS`

---

## ORDEN DE EJECUCION RECOMENDADO

```
Sesion 1 (Correccion critica):
  PASO 1: Eliminar 8 nodos y reconectar
  PASO 2: Router contains -> equals
  PASO 3: Parser null check
  PASO 5: API Daytona referencia
  PASO 6: Expresiones =={{ -> ={{
  --> PROBAR que el flujo basico funciona

Sesion 2 (Adjuntos y ACKs):
  PASO 7: Code Prep Forward (8 nodos)
  PASO 9: ACKs faltantes (6 nodos)
  --> PROBAR que adjuntos se envian correctamente

Sesion 3 (Validaciones post-Router):
  PASO 8: IF Apto post-Router (4 categorias)
  PASO 10: Validacion datos CCTV y Mantenimiento
  PASO 4: Actualizar typeVersions (si no causa problemas)
  PASO 11: Verificar credenciales
  PASO 12: Pruebas completas
```

---

## CONTEO FINAL DE NODOS

| Accion | Cantidad |
|---|---|
| Nodos eliminados | -8 |
| Nodos nuevos Code Prep Forward | +8 |
| Nodos nuevos ACK | +6 |
| Nodos nuevos IF Apto (post-Router) | +4 |
| Nodos nuevos Gmail Pedir Apto (post-Router) | +4 |
| Nodos nuevos Validacion CCTV | +2 (Code + IF) |
| Nodos nuevos Validacion Mantenimiento | +2 (Code + IF) |
| Nodos nuevos Gmail Solicitar Datos | +2 |
| **Neto** | **+20 nodos** |
| **Total estimado** | **~72 nodos** |

---

## NOTAS IMPORTANTES

1. **SIEMPRE hacer backup del JSON antes de cada sesion**
2. **Probar despues de cada paso**, no acumular cambios sin probar
3. **El nodo Code Prep Agent NO necesita cambios** - ya recupera binary de "Datos" y merge apto de "Code Resolver Apto"
4. **El AI Agent NO necesita adjuntos** - solo procesa texto
5. **IF#0 se CONSERVA** - sigue siendo util para detectar loops
6. **Las credenciales Gmail existentes se conservan** - solo verificar la segunda
<!--  -->