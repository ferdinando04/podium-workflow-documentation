# 🔧 PASO 1: CORREGIR CODE EXTRACTOR

## 🎯 OBJETIVO:
Reemplazar el código del nodo "Code - Extractor" con la versión blindada que:
- ✅ Retorna array de items
- ✅ Extrae email limpio (sin "Nombre <>")
- ✅ Parsea fechas robustamente
- ✅ Garantiza todos los campos (no undefined)

---

## 📋 INSTRUCCIONES PASO A PASO:

### PASO 1.1: Abrir n8n

1. **Abre tu navegador**
2. **Ve a:** `https://podium-n8n.lkrkth.easypanel.host/workflow/hlZdlklj1kYBnJ0fgZcN4`
3. **Espera** a que cargue el workflow completo

---

### PASO 1.2: Localizar el nodo "Code - Extractor"

1. **En el canvas**, busca cerca del inicio del workflow
2. **Secuencia:**
   ```
   📧 Gmail Trigger → 🔍 Code - Extractor → 📧 Gmail - ACK
   ```
3. **Click en:** `🔍 Code - Extractor`
4. Se abrirá el panel de configuración a la derecha

---

### PASO 1.3: Reemplazar el código

1. **En el panel**, verás el editor de código JavaScript
2. **Selecciona TODO el código actual** (Ctrl+A)
3. **Borra** todo
4. **Copia y pega** el siguiente código:

```javascript
// ============================================
// CODE EXTRACTOR - VERSIÓN BLINDADA
// ============================================

// Función para extraer email limpio de "Nombre <email@...>"
function extractEmail(raw) {
  if (!raw) return 'desconocido@email.com';
  
  // Buscar formato "Nombre <email>"
  const m = String(raw).match(/<([^>]+)>/);
  if (m?.[1]) return m[1].trim();
  
  // Buscar email directo
  const m2 = String(raw).match(/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/i);
  return m2?.[0] || 'desconocido@email.com';
}

// Función para parsear fecha robusta
function parseDate(d) {
  if (!d) return new Date().toISOString();
  
  const n = Number(d);
  if (!Number.isNaN(n) && n > 1000000000) {
    return new Date(n).toISOString();
  }
  
  return new Date(d).toISOString();
}

// Obtener item del input
const item = $input.first().json;

// EXTRAER DATOS CON NORMALIZACIÓN GARANTIZADA
const from_email = extractEmail(item.from || item.From || item.sender);
const subject = item.subject || item.Subject || 'Sin asunto';
const body = item.text || item.body || item.snippet || item.bodyPreview || 'Sin contenido';
const email_id = item.id || item.messageId || item.threadId || 'unknown';
const received_at = parseDate(item.date || item.internalDate);

// Extraer nombre del remitente
let nombre_residente = 'No especificado';
try {
  const rawFrom = item.from || item.From || '';
  // Buscar nombre antes de <email>
  const nameMatch = String(rawFrom).match(/^([^<]+)</);
  if (nameMatch) {
    nombre_residente = nameMatch[1].trim();
  } else {
    // Usar parte antes del @
    const emailPart = from_email.split('@')[0];
    nombre_residente = emailPart
      .replace(/[._-]/g, ' ')
      .split(' ')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
      .join(' ');
  }
} catch (e) {
  nombre_residente = 'No especificado';
}

// Buscar apartamento en subject o body
let apartamento = 'No especificado';
try {
  const aptoRegex = /(?:apto|apartamento|apt|unidad)[\s:]*(\d{3,4})/i;
  const combined = subject + ' ' + body;
  const match = combined.match(aptoRegex);
  
  if (match) {
    apartamento = match[1];
  }
} catch (e) {
  apartamento = 'No especificado';
}

// Detectar si es crítico (palabras clave urgentes)
const criticalKeywords = ['tutela', 'demanda', 'urgente', 'emergencia', 'derecho de petición'];
const is_critical = criticalKeywords.some(keyword => 
  (subject + ' ' + body).toLowerCase().includes(keyword)
);

// ✅ RETORNAR ARRAY DE ITEMS (CRÍTICO PARA N8N)
return [{
  json: {
    email_id: email_id,
    from_email: from_email,  // ✅ Email limpio sin "Nombre <>"
    subject: subject,
    body: body,
    text: body,  // Alias
    snippet: body.substring(0, 200),  // Primeros 200 caracteres
    nombre_residente: nombre_residente,
    apartamento: apartamento,
    received_at: received_at,  // ✅ Fecha robusta en ISO format
    is_critical: is_critical,
    // Aliases para compatibilidad
    Subject: subject,
    From: from_email
  }
}];
```

---

### PASO 1.4: Guardar cambios

1. **Cierra el panel** del nodo (click en X o fuera del panel)
2. **Click en "Save"** (botón naranja arriba a la derecha)
3. **Espera** el mensaje "Workflow saved" ✅

---

### PASO 1.5: Verificar que funciona

1. **Click en el nodo "Code - Extractor"** nuevamente
2. **Click en "Execute node"** (botón "Execute step" en el panel)
3. **Ve a la pestaña "OUTPUT"**
4. **Verifica que veas:**
   - ✅ `from_email`: Solo el email (ej: "juan@gmail.com"), NO "Juan Pérez <juan@gmail.com>"
   - ✅ `subject`: Tiene valor (no undefined)
   - ✅ `body`: Tiene valor (no undefined)
   - ✅ `nombre_residente`: Tiene valor (no undefined)
   - ✅ `apartamento`: Tiene valor (no undefined)
   - ✅ `received_at`: Fecha en formato ISO (ej: "2026-01-23T16:00:00.000Z")

---

## ✅ VERIFICACIÓN EXITOSA:

Si ves todos los campos con valores (ninguno "undefined"), **PASO 1 COMPLETADO** ✅

---

## 📸 MÁNDAME CAPTURA:

Por favor envía captura del OUTPUT del nodo mostrando que los campos están correctos.

---

## 🎯 SIGUIENTE PASO:

Una vez verificado el Code Extractor, continuaremos con:
- **PASO 2:** Modificar Output 8 (Info General)
- **PASO 3:** Agregar nodos ACK a Outputs 0-7
- **PASO 4:** Modificar nodos Forward de Outputs 0-7

**¿Listo para continuar después de verificar este paso?**
