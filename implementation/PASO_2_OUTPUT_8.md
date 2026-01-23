# 🔧 PASO 2: MODIFICAR OUTPUT 8 (INFORMACIÓN GENERAL)

## 🎯 OBJETIVO:

Corregir el flujo de Output 8 para que:
1. ✅ Email incluya **ACK + Respuesta IA** en el mismo mensaje
2. ✅ Telegram use **botones de aprobación** (no texto "SI/NO")
3. ✅ Workflow **espere respuesta real** de Telegram

---

## 📋 CAMBIOS A REALIZAR:

### Cambio 2.1: Modificar "Send IA Reply"
### Cambio 2.2: Modificar "Telegram Ask"  
### Cambio 2.3: Agregar nodo "IF Telegram Approved"

---

## 🔧 CAMBIO 2.1: MODIFICAR "SEND IA REPLY"

### PASO 2.1.1: Localizar el nodo

1. En el workflow, busca el nodo: **"📧 Send IA Reply"**
2. Está en la rama de Output 8 (INFORMACION_GENERAL)
3. **Click en el nodo**

---

### PASO 2.1.2: Modificar el Message

1. En el panel, busca el campo **"Message"**
2. **Borra** el contenido actual
3. **Copia y pega** este código:

```javascript
={{
  '<p>Hola,</p>' +
  '<p><strong>Gracias por escribirnos. Confirmamos que tu mensaje fue recibido y registrado correctamente.</strong></p>' +
  '<hr>' +
  '<p>' + ($json.ai_analysis?.borrador_respuesta || 'Estamos procesando tu solicitud.') + '</p>' +
  '<hr>' +
  '<p>Cordial saludo,<br>Administración Podium P.H.</p>'
}}
```

**Explicación:**
- Primera parte: **ACK** (confirmación de recepción)
- Segunda parte: **Respuesta IA** (del análisis)
- Tercera parte: **Firma**

---

### PASO 2.1.3: Verificar Subject

1. Busca el campo **"Subject"**
2. Debe tener:
   ```javascript
   ={{ 'Re: ' + $json.subject }}
   ```
3. Si no está, agrégalo

---

### PASO 2.1.4: Guardar

1. **Cierra el panel**
2. **Click en "Save"** (arriba a la derecha)

---

## 📱 CAMBIO 2.2: MODIFICAR "TELEGRAM ASK"

### PASO 2.2.1: Localizar el nodo

1. Busca el nodo: **"📱 Telegram Ask"**
2. Está en la rama de Output 8, después del IF Auto-Approve (FALSE)
3. **Click en el nodo**

---

### PASO 2.2.2: Cambiar Operation

1. En el panel, busca **"Operation"**
2. **Selecciona:** `Send and Wait for Response`

---

### PASO 2.2.3: Configurar Message

1. Busca el campo **"Message"**
2. **Reemplaza** con:

```javascript
={{ 
  '📧 *Aprobar respuesta automática?*\n\n' +
  '*De:* ' + $json.from_email + ' (' + $json.nombre_residente + ')\n' +
  '*Asunto:* ' + $json.subject + '\n' +
  '*Confianza:* ' + ($json.ai_analysis.confianza * 100).toFixed(1) + '%\n\n' +
  '*Borrador de Respuesta:*\n' + $json.ai_analysis.borrador_respuesta
}}
```

---

### PASO 2.2.4: Configurar Response Type

1. Busca **"Additional Fields"** o **"Options"**
2. **Click en "Add Field"**
3. **Selecciona:** `Response Configuration`
4. Dentro de Response Configuration:
   - **Response Type:** `Approval`
   - **Approve Button Text:** `✅ Aprobar`
   - **Reject Button Text:** `❌ Rechazar`

---

### PASO 2.2.5: Configurar Timeout

1. En **"Additional Fields"** o **"Options"**
2. **Busca:** `Limit Wait Time`
3. **Selecciona:** `1 hour` (o 3600 segundos)

---

### PASO 2.2.6: Configurar Parse Mode

1. En **"Additional Fields"**
2. **Busca:** `Parse Mode`
3. **Selecciona:** `Markdown` (Legacy)

---

### PASO 2.2.7: Guardar

1. **Cierra el panel**
2. **Click en "Save"**

---

## ⚖️ CAMBIO 2.3: AGREGAR NODO "IF TELEGRAM APPROVED"

### PASO 2.3.1: Agregar nuevo nodo IF

1. **Click en el "+"** después del nodo "Telegram Ask"
2. **Busca:** `IF`
3. **Selecciona:** `IF` node
4. **Nombre:** `IF Telegram Approved`

---

### PASO 2.3.2: Configurar condición

1. En el panel del IF:
   - **Condition:** `{{ $json.approved === true }}`

O si prefieres la UI:
   - **Value 1:** `{{ $json.approved }}`
   - **Operation:** `Equal`
   - **Value 2:** `true`

---

### PASO 2.3.3: Conectar nodos

**Conexiones:**

```
Telegram Ask
  ↓
IF Telegram Approved
  ├─ TRUE → Send IA Reply
  └─ FALSE → Label IA (o Supabase Log)
```

**Cómo conectar:**

1. **Arrastra** desde el punto de salida de "Telegram Ask"
2. **Conecta** a "IF Telegram Approved"
3. **Arrastra** desde "TRUE" del IF
4. **Conecta** a "Send IA Reply"
5. **Arrastra** desde "FALSE" del IF
6. **Conecta** a "Label IA" (o directamente a Supabase si prefieres)

---

### PASO 2.3.4: Guardar

1. **Click en "Save"**

---

## ✅ VERIFICACIÓN

### Flujo final Output 8 debe ser:

```
Router [Output 8]
  ↓
IF Auto-Approve (confianza >= 85%)
  ├─ TRUE → Send IA Reply (ACK + Respuesta)
  │           ↓
  │         Label IA
  │           ↓
  │         Supabase Log
  │
  └─ FALSE → Telegram Ask (Send and Wait)
              ↓
            IF Telegram Approved
              ├─ TRUE → Send IA Reply (ACK + Respuesta)
              │           ↓
              │         Label IA
              │           ↓
              │         Supabase Log
              │
              └─ FALSE → Label IA (rechazado)
                          ↓
                        Supabase Log
```

---

## 📸 MÁNDAME CAPTURA:

Por favor envía captura mostrando:
1. El nodo "Send IA Reply" con el nuevo Message
2. El nodo "Telegram Ask" con Response Type: Approval
3. El nuevo nodo "IF Telegram Approved" conectado

---

## 🎯 SIGUIENTE PASO:

Una vez verificado Output 8, continuaremos con:
- **PASO 3:** Modificar Outputs 0-7 (Departamentos)
  - Agregar nodos ACK
  - Modificar nodos Forward

**¿Listo para continuar después de verificar este paso?**
