# 🎯 PLAN COMPLETO DE IMPLEMENTACIÓN - WORKFLOW PODIUM

## 📋 NOMBRE DEL PROYECTO

**Repositorio GitHub:** `podium-workflow-documentation`

---

## 🎯 OBJETIVO

Implementar workflow n8n que cumpla 100% con los requerimientos de Don Jorge:
- ✅ Clasificación automática con IA
- ✅ ACK específico por categoría
- ✅ Forward de email original a departamentos
- ✅ **Validación de datos obligatorios** (CCTV y Mantenimiento)
- ✅ Respuesta IA para info general
- ✅ Labels y logging completo

---

## 📊 FASES DE IMPLEMENTACIÓN

### 🔴 FASE 1: CODE EXTRACTOR (COMPLETADO ✅)

- [x] Retornar array de items
- [x] Extraer email limpio
- [x] Parse date robusto
- [x] Garantizar todos los campos

---

### 🟡 FASE 2: OUTPUT 8 (INFORMACIÓN GENERAL)

- [ ] **PASO 2.1:** Modificar "Send IA Reply"
  - ACK + Respuesta en mismo email
  
- [ ] **PASO 2.2:** Modificar "Telegram Ask"
  - Send and Wait for Response
  - Response Type: Approval
  - Botones ✅ Aprobar / ❌ Rechazar
  
- [ ] **PASO 2.3:** Agregar nodo "IF Telegram Approved"
  - Validar $json.approved === true

---

### 🟢 FASE 3: OUTPUTS CON VALIDACIÓN DE DATOS

#### 📹 Output 1 - CCTV (CON VALIDACIÓN)

**Flujo completo:**

```
Router [Output 1]
  ↓
Code - Validar Datos CCTV
  ↓
IF Datos Completos?
  ├─ TRUE → Gmail ACK CCTV
  │           ↓
  │         Gmail Forward CCTV
  │           ↓
  │         Label CCTV
  │           ↓
  │         Supabase Log
  │
  └─ FALSE → Gmail Solicitar Datos CCTV
              ↓
            Label CCTV_INCOMPLETO
              ↓
            Supabase Log (incompleto)
```

**Datos obligatorios a validar:**
- Nombre solicitante
- Apartamento o local
- Fecha del evento
- Hora aproximada o rango
- Zona o cámara solicitada
- Motivo de la solicitud

**Pasos:**
- [ ] Agregar nodo "Code - Validar Datos CCTV"
- [ ] Agregar nodo "IF Datos Completos CCTV"
- [ ] Agregar nodo "Gmail Solicitar Datos CCTV"
- [ ] Agregar nodo "Gmail ACK CCTV"
- [ ] Modificar nodo "Gmail Forward CCTV"

---

#### 🔧 Output 5 - MANTENIMIENTO (CON VALIDACIÓN)

**Flujo completo:**

```
Router [Output 5]
  ↓
Code - Validar Datos Mantenimiento
  ↓
IF Datos Completos?
  ├─ TRUE → Gmail ACK Mantenimiento
  │           ↓
  │         Gmail Forward Mantenimiento
  │           ↓
  │         Label Mantenimiento
  │           ↓
  │         Supabase Log
  │
  └─ FALSE → Gmail Solicitar Datos Mantenimiento
              ↓
            Label MANTENIMIENTO_INCOMPLETO
              ↓
            Supabase Log (incompleto)
```

**Datos mínimos a validar:**
- Ubicación exacta
- Descripción clara de la novedad
- Disponibilidad de ingreso

**Pasos:**
- [ ] Agregar nodo "Code - Validar Datos Mantenimiento"
- [ ] Agregar nodo "IF Datos Completos Mantenimiento"
- [ ] Agregar nodo "Gmail Solicitar Datos Mantenimiento"
- [ ] Agregar nodo "Gmail ACK Mantenimiento"
- [ ] Modificar nodo "Gmail Forward Mantenimiento"

---

### 🔵 FASE 4: OUTPUTS SIN VALIDACIÓN (0, 2, 3, 4, 6, 7)

#### Output 0 - LEGAL

- [ ] Agregar Gmail ACK Legal
- [ ] Modificar Gmail Forward Legal

#### Output 2 - CONVIVENCIA

- [ ] Agregar Gmail ACK Convivencia
- [ ] Modificar Gmail Forward Convivencia

#### Output 3 - CONTABILIDAD

- [ ] Agregar Gmail ACK Contabilidad
- [ ] Modificar Gmail Forward Contabilidad

#### Output 4 - FACTURAS PROVEEDORES

- [ ] Agregar Gmail ACK Facturas
- [ ] Modificar Gmail Forward Facturas

#### Output 6 - TRASTEOS

- [ ] Agregar Gmail ACK Trasteos
- [ ] Modificar Gmail Forward Trasteos

#### Output 7 - SALÓN SOCIAL

- [ ] Agregar Gmail ACK Salón
- [ ] Modificar Gmail Forward Salón

---

## 📝 CÓDIGO PARA VALIDACIÓN DE DATOS

### Code - Validar Datos CCTV

```javascript
// Obtener datos
const item = $input.first().json;
const subject = item.subject || '';
const body = item.body || '';
const combined = subject + ' ' + body;

// Validar campos obligatorios
const validations = {
  nombre: item.nombre_residente !== 'No especificado',
  apartamento: item.apartamento !== 'No especificado',
  fecha: /\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4}/.test(combined),
  hora: /\d{1,2}:\d{2}|hora|am|pm/i.test(combined),
  zona: /parqueadero|lobby|piso|zona|cámara|área/i.test(combined),
  motivo: body.length > 20 // Al menos 20 caracteres de descripción
};

// Contar cuántos campos están completos
const completos = Object.values(validations).filter(v => v).length;
const total = Object.keys(validations).length;

// Determinar si está completo (al menos 4 de 6 campos)
const datos_completos = completos >= 4;

// Generar lista de campos faltantes
const faltantes = [];
if (!validations.nombre) faltantes.push('Nombre completo');
if (!validations.apartamento) faltantes.push('Número de apartamento');
if (!validations.fecha) faltantes.push('Fecha del evento');
if (!validations.hora) faltantes.push('Hora aproximada');
if (!validations.zona) faltantes.push('Zona o cámara específica');
if (!validations.motivo) faltantes.push('Descripción detallada del motivo');

return [{
  json: {
    ...item,
    datos_completos: datos_completos,
    campos_faltantes: faltantes,
    validacion_cctv: {
      completos: completos,
      total: total,
      porcentaje: (completos / total * 100).toFixed(0)
    }
  }
}];
```

---

### Code - Validar Datos Mantenimiento

```javascript
// Obtener datos
const item = $input.first().json;
const subject = item.subject || '';
const body = item.body || '';
const combined = subject + ' ' + body;

// Validar campos mínimos
const validations = {
  ubicacion: /apto|apartamento|piso|zona|área|ubicación/i.test(combined),
  descripcion: body.length > 30, // Al menos 30 caracteres
  disponibilidad: /disponible|horario|lunes|martes|miércoles|jueves|viernes|sábado|domingo|mañana|tarde/i.test(combined)
};

// Contar campos completos
const completos = Object.values(validations).filter(v => v).length;
const total = Object.keys(validations).length;

// Determinar si está completo (todos los campos)
const datos_completos = completos === total;

// Generar lista de campos faltantes
const faltantes = [];
if (!validations.ubicacion) faltantes.push('Ubicación exacta (apartamento, zona)');
if (!validations.descripcion) faltantes.push('Descripción clara de la novedad');
if (!validations.disponibilidad) faltantes.push('Disponibilidad de ingreso (horario)');

return [{
  json: {
    ...item,
    datos_completos: datos_completos,
    campos_faltantes: faltantes,
    validacion_mantenimiento: {
      completos: completos,
      total: total,
      porcentaje: (completos / total * 100).toFixed(0)
    }
  }
}];
```

---

## 📧 PLANTILLAS PARA SOLICITAR DATOS

### Gmail Solicitar Datos CCTV

**To:** `={{ $json.from_email }}`
**Subject:** `Información adicional requerida – CCTV Podium P.H.`
**Message:**

```javascript
={{
  '<p>Hola,</p>' +
  '<p>Hemos recibido tu solicitud de revisión de cámaras. Para poder procesarla, necesitamos la siguiente información adicional:</p>' +
  '<ul>' +
  ($json.campos_faltantes || []).map(campo => '<li>' + campo + '</li>').join('') +
  '</ul>' +
  '<p>Por favor responde a este correo con los datos solicitados.</p>' +
  '<p>Cordial saludo,<br>Administración Podium P.H.</p>'
}}
```

---

### Gmail Solicitar Datos Mantenimiento

**To:** `={{ $json.from_email }}`
**Subject:** `Información requerida – Mantenimiento Podium P.H.`
**Message:**

```javascript
={{
  '<p>Hola,</p>' +
  '<p>Para gestionar tu solicitud de mantenimiento necesitamos:</p>' +
  '<ul>' +
  ($json.campos_faltantes || []).map(campo => '<li>' + campo + '</li>').join('') +
  '</ul>' +
  '<p>Gracias por tu apoyo.</p>' +
  '<p>Administración Podium P.H.</p>'
}}
```

---

## 📊 RESUMEN DE NODOS TOTALES

| Componente | Nodos Actuales | Nodos Nuevos | Total Final |
|------------|----------------|--------------|-------------|
| Recepción | 3 | 0 | 3 |
| IA | 3 | 0 | 3 |
| Router | 1 | 0 | 1 |
| Output 0 (Legal) | 3 | 1 ACK | 4 |
| Output 1 (CCTV) | 3 | 1 Code + 1 IF + 1 Solicitar + 1 ACK | 7 |
| Output 2 (Convivencia) | 3 | 1 ACK | 4 |
| Output 3 (Contabilidad) | 3 | 1 ACK | 4 |
| Output 4 (Facturas) | 3 | 1 ACK | 4 |
| Output 5 (Mantenimiento) | 3 | 1 Code + 1 IF + 1 Solicitar + 1 ACK | 7 |
| Output 6 (Trasteos) | 3 | 1 ACK | 4 |
| Output 7 (Salón) | 3 | 1 ACK | 4 |
| Output 8 (Info) | 6 | 1 IF | 7 |
| **TOTAL** | **40** | **+16** | **56** |

---

## 📁 ESTRUCTURA GITHUB

```
podium-workflow-documentation/
├── README.md
├── docs/
│   ├── REQUERIMIENTOS_MAESTROS_DON_JORGE.md
│   ├── PLAN_COMPLETO_IMPLEMENTACION.md
│   ├── MAQUETA_FLUJO_COMPLETO.md
│   └── diagrams/
│       ├── workflow_completo.png
│       └── flujo_validacion_datos.png
├── implementation/
│   ├── PASO_1_CODE_EXTRACTOR.md
│   ├── PASO_2_OUTPUT_8.md
│   ├── PASO_3_CCTV_VALIDACION.md
│   ├── PASO_4_MANTENIMIENTO_VALIDACION.md
│   └── PASO_5_OUTPUTS_RESTANTES.md
├── code/
│   ├── code_extractor.js
│   ├── validar_datos_cctv.js
│   ├── validar_datos_mantenimiento.js
│   └── templates/
│       ├── gmail_ack_templates.js
│       └── gmail_forward_templates.js
└── verification/
    ├── test_cases.md
    └── checklist_final.md
```

---

## ✅ CHECKLIST FINAL

- [ ] Code Extractor blindado
- [ ] Output 8 con ACK + Respuesta
- [ ] Telegram con botones de aprobación
- [ ] CCTV con validación de datos
- [ ] Mantenimiento con validación de datos
- [ ] Outputs 0,2,3,4,6,7 con ACK + Forward
- [ ] Todas las plantillas correctas
- [ ] Labels aplicados
- [ ] Supabase logging completo
- [ ] Pruebas de cada categoría
- [ ] Documentación en GitHub

---

**PRÓXIMO PASO:** Crear imagen actualizada del flujo con validación de datos.
