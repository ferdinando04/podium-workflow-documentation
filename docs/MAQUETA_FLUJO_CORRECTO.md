# 🎯 MAQUETA DEL FLUJO CORRECTO - Podium Workflow

## ⚠️ CORRECCIONES TÉCNICAS APLICADAS

### ✅ 1. Code Extractor - Return Array
```javascript
return [{  // ✅ Array de items
  json: {...}
}];
```

### ✅ 2. Email Limpio (sin "Nombre <email>")
```javascript
function extractEmail(raw) {
  // Extrae solo email@... de "Nombre <email@...>"
}
```

### ✅ 3. Parse Date Robusto
```javascript
function parseDate(d) {
  // Maneja milisegundos, strings, formatos raros
}
```

### ✅ 4. HTML Escapado en Body
```javascript
'<pre>' + escapeHtml($json.body) + '</pre>'
```

### ✅ 5. Telegram Approval Buttons
```
Response Type: Approval
Botones: ✅ Aprobar / ❌ Rechazar
```

### ✅ 6. Timeout Correcto
```
Limit Wait Time: 1 hour
```

---

## 📊 DIAGRAMA VISUAL COMPLETO

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECCIÓN 1: RECEPCIÓN                         │
└─────────────────────────────────────────────────────────────────┘

    📧 Gmail Trigger
    (Detecta email nuevo en crcpodiumph@gmail.com)
         │
         ↓
    🔍 Code - Extractor
    (Extrae: remitente, asunto, cuerpo, apartamento, etc.)
         │
         ↓
    📝 Code - Prep Agent
    (Prepara datos para IA)
         │
         ↓

┌─────────────────────────────────────────────────────────────────┐
│              SECCIÓN 2: ANÁLISIS CON IA                         │
└─────────────────────────────────────────────────────────────────┘

    🤖 AI Agent (RAG)
    (Clasifica email en 9 categorías + genera análisis)
         │
         ├─ Usa Vector Store (Manual Convivencia)
         ├─ Usa OpenAI Model
         └─ Genera:
             • Categoría (ej: CCTV_SEGURIDAD)
             • Confianza (ej: 95%)
             • Resumen
             • Borrador respuesta (solo para INFO_GENERAL)
         │
         ↓
    📋 Code - Parser
    (Procesa resultado de IA y prepara para Router)
         │
         ↓

┌─────────────────────────────────────────────────────────────────┐
│           SECCIÓN 3: ROUTER (CLASIFICACIÓN)                     │
└─────────────────────────────────────────────────────────────────┘

    🔀 Router
    (Decide qué hacer según categoría)
         │
         ├─ Output 0: TEMA_LEGAL_PQR
         ├─ Output 1: CCTV_SEGURIDAD
         ├─ Output 2: CONVIVENCIA
         ├─ Output 3: CONTABILIDAD
         ├─ Output 4: FACTURAS_PROVEEDORES
         ├─ Output 5: MANTENIMIENTO
         ├─ Output 6: TRASTEOS
         ├─ Output 7: SALON_SOCIAL
         └─ Output 8: INFORMACION_GENERAL (DEFAULT)

┌─────────────────────────────────────────────────────────────────┐
│    SECCIÓN 4A: FLUJO OUTPUTS 0-7 (DEPARTAMENTOS)                │
└─────────────────────────────────────────────────────────────────┘

Ejemplo: Output 1 - CCTV

    Router [Output 1]
         │
         ↓
    📧 Gmail ACK CCTV
    (Envía ACK al RESIDENTE)
         │
         ├─ To: juan.perez@gmail.com (el que envió)
         ├─ Subject: "Solicitud de CCTV recibida – Podium P.H."
         └─ Message: "Hemos recibido tu solicitud...
                      Tiempo de respuesta: 3 días hábiles..."
         │
         ↓
    📧 Gmail Forward CCTV
    (Envía email al DEPARTAMENTO)
         │
         ├─ To: operaciones@seguridadp3.com
         ├─ CC: jefe.operaciones@..., control@..., coordinador@...
         ├─ Subject: "Solicitud de verificación CCTV – Podium P.H."
         └─ Message:
             ┌──────────────────────────────────────┐
             │ DATOS DEL RESIDENTE:                 │
             │ - Nombre: Juan Pérez                 │
             │ - Email: juan.perez@gmail.com        │
             │ - Apartamento: 301                   │
             │ - Categoría IA: CCTV (95%)           │
             │                                      │
             │ --- EMAIL ORIGINAL ---               │
             │ De: juan.perez@gmail.com             │
             │ Asunto: Revisión cámaras             │
             │ Cuerpo: Solicito revisar...          │
             │                                      │
             │ Plazo: 3 días hábiles                │
             └──────────────────────────────────────┘
         │
         ↓
    🏷️ Label CCTV
    (Aplica etiqueta "CCTV_SEGURIDAD" en Gmail)
         │
         ↓
    🗄️ Supabase Log CCTV
    (Guarda registro en base de datos)
         │
         └─ FIN ✅

┌─────────────────────────────────────────────────────────────────┐
│    SECCIÓN 4B: FLUJO OUTPUT 8 (INFORMACIÓN GENERAL)             │
└─────────────────────────────────────────────────────────────────┘

    Router [Output 8]
         │
         ↓
    ⚖️ IF Auto-Approve
    (¿Confianza >= 85%?)
         │
         ├─ TRUE (Confianza alta)
         │    │
         │    ↓
         │  📧 Send IA Reply
         │  (Envía respuesta automática al residente)
         │       │
         │       ├─ To: juan.perez@gmail.com
         │       ├─ Subject: "Re: [asunto original]"
         │       └─ Message:
         │           ┌──────────────────────────────────┐
         │           │ Hola,                            │
         │           │                                  │
         │           │ Gracias por escribirnos...       │
         │           │                                  │
         │           │ [RESPUESTA GENERADA POR IA]      │
         │           │ (Usa info del Manual RAG)        │
         │           │                                  │
         │           │ Cordialmente,                    │
         │           │ Administración Podium P.H.       │
         │           └──────────────────────────────────┘
         │
         └─ FALSE (Confianza baja < 85%)
              │
              ↓
            📱 Telegram Ask
            (Pide aprobación manual al admin)
                 │
                 ├─ Envía a grupo Telegram
                 ├─ Muestra borrador de respuesta
                 └─ Espera SI/NO del admin
                 │
                 ↓
            📧 Send IA Reply
            (Solo si admin aprueba)
         │
         ↓
    🏷️ Label IA
    (Aplica etiqueta "IA_RESPONDIDO")
         │
         ↓
    🗄️ Supabase Log Info
    (Guarda registro)
         │
         └─ FIN ✅
```

---

## 📋 EXPLICACIÓN DETALLADA POR SECCIÓN

### 🔵 SECCIÓN 1: RECEPCIÓN

**¿Qué hace?**
- Detecta cuando llega un email nuevo
- Extrae todos los datos importantes (remitente, asunto, cuerpo, etc.)
- Prepara los datos para enviarlos a la IA

**Nodos:**
1. **Gmail Trigger** - Escucha emails nuevos en crcpodiumph@gmail.com
2. **Code - Extractor** - Extrae datos del email (nombre, apartamento, etc.)
3. **Code - Prep Agent** - Formatea datos para la IA

**Salida:** Datos estructurados listos para IA

---

### 🟢 SECCIÓN 2: ANÁLISIS CON IA

**¿Qué hace?**
- La IA lee el email
- Consulta el Manual de Convivencia (RAG)
- Clasifica en 1 de 9 categorías
- Genera análisis y resumen

**Nodos:**
1. **AI Agent (RAG)** - Modelo OpenAI + Vector Store
2. **Code - Parser** - Procesa respuesta de IA

**Salida:** 
```json
{
  "categoria": "CCTV_SEGURIDAD",
  "confianza": 0.95,
  "resumen": "Solicitud de revisión de cámaras",
  "borrador_respuesta": "..." (solo para INFO_GENERAL)
}
```

---

### 🟡 SECCIÓN 3: ROUTER

**¿Qué hace?**
- Lee la categoría que asignó la IA
- Decide a qué "Output" enviar el email
- Cada Output es un camino diferente

**Categorías:**
- Output 0: TEMA_LEGAL_PQR → Casos legales urgentes
- Output 1: CCTV_SEGURIDAD → Solicitudes de cámaras
- Output 2: CONVIVENCIA → Reportes de vecinos
- Output 3: CONTABILIDAD → Pagos y estados de cuenta
- Output 4: FACTURAS_PROVEEDORES → Facturas de empresas
- Output 5: MANTENIMIENTO → Reparaciones
- Output 6: TRASTEOS → Mudanzas
- Output 7: SALON_SOCIAL → Reservas de salón
- Output 8: INFORMACION_GENERAL → Preguntas generales

---

### 🔴 SECCIÓN 4A: OUTPUTS 0-7 (DEPARTAMENTOS)

**¿Qué hace?**
Para emails que necesitan atención de un departamento específico.

**Flujo (4 pasos):**

#### PASO 1: ACK al Residente
- **Quién lo recibe:** La persona que envió el email
- **Qué dice:** Confirmación de que recibimos su solicitud
- **Ejemplo (CCTV):**
  ```
  Hola,
  
  Hemos recibido tu solicitud de revisión de cámaras.
  La información fue enviada al operador de seguridad.
  
  Tiempo de respuesta: 3 días hábiles.
  
  Administración Podium P.H.
  ```

#### PASO 2: Forward al Departamento
- **Quién lo recibe:** El departamento encargado (ej: seguridad, legal, contabilidad)
- **Qué contiene:**
  1. Datos del residente (nombre, email, apartamento)
  2. Análisis de IA (categoría, confianza)
  3. **EMAIL ORIGINAL COMPLETO** del residente
  4. Instrucciones específicas para el departamento

- **Ejemplo (CCTV a seguridad):**
  ```
  Buen día,
  
  Se solicita verificación de cámaras.
  
  DATOS DEL RESIDENTE:
  - Nombre: Juan Pérez
  - Email: juan.perez@gmail.com
  - Apartamento: 301
  
  --- EMAIL ORIGINAL ---
  Asunto: Revisión cámaras
  Solicito revisar las cámaras del parqueadero...
  ---
  
  Plazo: 3 días hábiles.
  ```

#### PASO 3: Label
- Aplica etiqueta en Gmail para organizar

#### PASO 4: Supabase Log
- Guarda registro en base de datos para auditoría

---

### 🟣 SECCIÓN 4B: OUTPUT 8 (INFORMACIÓN GENERAL)

**¿Qué hace?**
Para preguntas que la IA puede responder directamente (horarios, normas, etc.)

**Flujo:**

#### PASO 1: Verificar Confianza
- **Si confianza >= 85%:** Enviar respuesta automática
- **Si confianza < 85%:** Pedir aprobación a admin por Telegram

#### PASO 2A: Respuesta Automática (confianza alta)
- Envía respuesta generada por IA directamente al residente
- Usa información del Manual de Convivencia (RAG)

#### PASO 2B: Aprobación Manual (confianza baja)
- Envía borrador a Telegram
- Admin revisa y aprueba/rechaza
- Si aprueba → envía respuesta

#### PASO 3: Label y Log
- Igual que outputs 0-7

---

## 🎯 DIFERENCIAS CLAVE

### Outputs 0-7 (Departamentos):
```
Email → IA clasifica → ACK al residente → Forward a departamento → Label → Log
```
**NO hay respuesta automática**, solo confirmación de recepción.

### Output 8 (Info General):
```
Email → IA clasifica → IA genera respuesta → Envía al residente → Label → Log
```
**SÍ hay respuesta automática** con información del Manual.

---

## 📝 EJEMPLO COMPLETO - CASO CCTV

### Email Original:
```
De: juan.perez@gmail.com
Para: crcpodiumph@gmail.com
Asunto: Revisión cámaras
Cuerpo: Solicito revisar las cámaras del parqueadero el 20/01 a las 3pm
```

### Paso 1: IA Clasifica
```
Categoría: CCTV_SEGURIDAD
Confianza: 95%
Resumen: Solicitud de revisión de cámaras de parqueadero
```

### Paso 2: Router → Output 1 (CCTV)

### Paso 3: ACK al Residente (juan.perez@gmail.com)
```
Asunto: Solicitud de CCTV recibida – Podium P.H.

Hola,

Hemos recibido tu solicitud de revisión de cámaras.
La información fue enviada al operador de seguridad.

Tiempo de respuesta: 3 días hábiles.

Administración Podium P.H.
```

### Paso 4: Forward a Seguridad (operaciones@seguridadp3.com)
```
Asunto: Solicitud de verificación CCTV – Podium P.H.

Buen día,

Se solicita verificación de cámaras.

DATOS DEL RESIDENTE:
- Nombre: Juan Pérez
- Email: juan.perez@gmail.com
- Apartamento: 301
- Categoría IA: CCTV_SEGURIDAD (95%)

--- EMAIL ORIGINAL DEL RESIDENTE ---
De: juan.perez@gmail.com
Asunto: Revisión cámaras
Fecha: 23-Ene-2026

Solicito revisar las cámaras del parqueadero el 20/01 a las 3pm

---

Plazo máximo: 3 días hábiles.

Administración Podium P.H.
```

### Paso 5: Label "CCTV_SEGURIDAD" en Gmail

### Paso 6: Log en Supabase

---

## ❓ PREGUNTAS FRECUENTES

### ¿Por qué hay 2 emails (ACK + Forward)?

**ACK:** Para que el residente sepa que recibimos su solicitud.
**Forward:** Para que el departamento tenga toda la información y pueda atender.

### ¿La IA responde en todos los casos?

**NO.** Solo en Output 8 (INFORMACION_GENERAL).
En Outputs 0-7, la IA solo **clasifica**, no responde.

### ¿Qué pasa si la IA se equivoca de categoría?

El admin puede ver en Supabase y reclasificar manualmente si es necesario.

---

**¿Te queda claro el flujo? ¿Procedo con la implementación?**
