# 📋 REQUERIMIENTOS MAESTROS - DON JORGE

## 🎯 DOCUMENTO RAÍZ - REFERENCIA OBLIGATORIA

Este documento contiene TODOS los requerimientos oficiales de Don Jorge.
**NUNCA implementar nada que contradiga este documento.**

---

## 1️⃣ PRINCIPIOS GENERALES

- ✅ Canal oficial único: **crcpodiumph@gmail.com**
- ✅ WhatsApp: Solo informativo (NO soportes, NO trámites, NO decisiones)
- ✅ Todo correo debe generar: **etiqueta + registro + responsable + cierre**
- ✅ n8n organiza y automatiza; **decisiones finales son humanas**

---

## 2️⃣ CORREOS OFICIALES POR ÁREA

### ADMINISTRACIÓN
- masvalorcontacto@gmail.com

### JURÍDICO (demandas, tutelas, derechos de petición)
- auramaria64@hotmail.com

### CONTABILIDAD / CARTERA
- contabilidadcrcpodium@gmail.com
- greisy.garcia@gyosoluciones.com.co
- contacto@gyosoluciones.com.co

### SEGURIDAD / CCTV
- operaciones@seguridadp3.com
- jefe.operaciones@seguridadp3.com
- control@seguridadp3.com
- coordinador.supervision@seguridadp3.com

### MANTENIMIENTO
- masvalormantenimiento@gmail.com
- CC: masvalorcontacto@gmail.com

### COMITÉ DE CONVIVENCIA
- jandresmeza@outlook.es

---

## 3️⃣ PROCEDIMIENTOS ESPECÍFICOS

### 🎥 PROCEDIMIENTO CCTV

**Datos obligatorios que n8n DEBE validar:**
- ✅ Nombre solicitante
- ✅ Apartamento o local
- ✅ Fecha del evento
- ✅ Hora aproximada o rango
- ✅ Zona o cámara solicitada
- ✅ Motivo de la solicitud

**Flujo:**
1. n8n detecta correo entrante
2. **SI FALTAN DATOS** → respuesta automática solicitando información
3. **SI ESTÁ COMPLETO** → enviar acuse automático al residente
4. n8n reenvía solicitud a TODOS los correos de SEGURIDAD
5. Se activa contador de 3 días hábiles
6. Si no hay respuesta → recordatorio automático a seguridad
7. Seguridad responde con INFORME
8. n8n envía respuesta formal al residente (NO se adjuntan videos)

---

### ⚖️ PROCEDIMIENTO LEGAL

**Flujo:**
1. n8n marca el correo como **ALTA PRIORIDAD**
2. Se envía acuse de recibido automático
3. n8n reenvía automáticamente a: auramaria64@hotmail.com
4. Administración mantiene copia y control
5. Respuesta se gestiona dentro de términos legales

---

### 💰 PROCEDIMIENTO CARTERA Y ESTADOS DE CUENTA

**Flujo:**
1. n8n confirma recepción automática
2. n8n reenvía a contabilidad (3 correos)
3. Si solicitud es ESTADO DE CUENTA → consultar API Daytona
4. Si pago no aparece → solicitar soporte
5. Enviar respuesta al residente

---

### 🏘️ PROCEDIMIENTO CONVIVENCIA

**Flujo:**
1. Acuse automático al residente
2. Solicitud de informe a SEGURIDAD
3. Recepción de informe
4. Reenvío automático al Comité de Convivencia
5. Registro del caso

---

### 🔧 PROCEDIMIENTO MANTENIMIENTO

**Datos mínimos requeridos:**
- ✅ Ubicación exacta
- ✅ Descripción clara de la novedad
- ✅ Disponibilidad de ingreso

**Flujo:**
- n8n solicita datos mínimos
- Reenvía automáticamente a mantenimiento con copia a administración

---

### 📦 PROCEDIMIENTO TRASTEOS

**Flujo:**
- n8n valida estado de cartera en Daytona
- Si cumple → envía reglas y notifica a portería

---

### 🎉 PROCEDIMIENTO SALÓN SOCIAL

**Flujo:**
- n8n valida estado de cuenta en Daytona
- Consulta Google Calendar
- Envía costos, normas y requisitos

---

### 📄 PROCEDIMIENTO FACTURAS PROVEEDORES

**Flujo:**
- n8n detecta adjuntos
- Reenvía automáticamente a contabilidad
- Solicita causación

---

## 4️⃣ PLANTILLAS DE CORREOS OFICIALES

### 📧 1. Acuse General de Recepción

**Asunto:** Hemos recibido tu solicitud – Podium P.H.

```
Hola,

Gracias por escribirnos. Confirmamos que tu mensaje fue recibido y registrado correctamente.

Tu solicitud será revisada según el procedimiento correspondiente. Si necesitamos información adicional, te contactaremos por este medio.

Cordial saludo,
Administración Podium P.H.
```

---

### 🎥 2. CCTV – Acuse al Residente

**Asunto:** Solicitud de CCTV recibida – Podium P.H.

```
Hola,

Hemos recibido tu solicitud de revisión de cámaras. La información fue enviada al operador de seguridad.

El tiempo estimado de respuesta es de hasta 3 días hábiles. Por políticas de seguridad, no se entregan videos, solo informe escrito.

Administración Podium P.H.
```

---

### 🎥 3. CCTV – Solicitud a Seguridad

**Asunto:** Solicitud de verificación CCTV – Podium P.H.

```
Buen día,

Se solicita verificación de cámaras con base en la información recibida del residente.

Agradecemos remitir informe escrito indicando si se evidencian o no los hechos reportados.

Plazo máximo: 3 días hábiles.

Administración Podium P.H.
```

---

### 🎥 4. CCTV – Respuesta Final al Residente

**Asunto:** Resultado solicitud CCTV – Podium P.H.

```
Hola,

En atención a tu solicitud, informamos que según el informe de seguridad:

[Se evidenció / No se evidenció] el hecho reportado.

Este resultado queda registrado para trazabilidad.

Administración Podium P.H.
```

---

### ⚖️ 5. Legal – Alta Prioridad

**Asunto:** Requerimiento legal recibido – Podium P.H.

```
Hola,

Confirmamos la recepción de tu comunicación de carácter legal.

El caso fue marcado como alta prioridad y remitido al área jurídica para su trámite.

Administración Podium P.H.
```

---

### 💰 6. Cartera y Contabilidad – Acuse

**Asunto:** Solicitud contable recibida – Podium P.H.

```
Hola,

Hemos recibido tu solicitud relacionada con cartera o pagos. La consulta fue remitida al área contable para validación.

Te informaremos cualquier novedad.

Administración Podium P.H.
```

---

### 💰 7. Pago No Reflejado – Solicitud de Soporte

**Asunto:** Soporte de pago requerido – Podium P.H.

```
Hola,

Para validar tu pago, por favor envíanos el soporte indicando fecha, valor y medio utilizado.

Una vez recibido, continuamos con la revisión.

Administración Podium P.H.
```

---

### 🏘️ 8. Convivencia – Acuse

**Asunto:** Reporte de convivencia recibido – Podium P.H.

```
Hola,

Gracias por informarnos. Tu reporte fue recibido y será analizado conforme al Manual de Convivencia y el debido proceso.

Administración Podium P.H.
```

---

### 🔧 9. Mantenimiento – Solicitud de Información

**Asunto:** Información requerida – Mantenimiento Podium P.H.

```
Hola,

Para gestionar tu solicitud necesitamos ubicación exacta, descripción clara de la novedad y disponibilidad de ingreso.

Gracias por tu apoyo.

Administración Podium P.H.
```

---

### 📦 10. Trasteos – Acuse

**Asunto:** Solicitud de trasteo recibida – Podium P.H.

```
Hola,

Recibimos tu solicitud de trasteo. Será validada conforme a cartera y reglamentación interna.

Te informaremos los pasos a seguir.

Administración Podium P.H.
```

---

### 📝 11. Descargos – Acuse

**Asunto:** Descargos recibidos – Podium P.H.

```
Hola,

Confirmamos la recepción de tus descargos. Serán revisados conforme al procedimiento interno.

Administración Podium P.H.
```

---

### 🎉 12. Salón Social – Acuse

**Asunto:** Solicitud Salón Social – Podium P.H.

```
Hola,

Hemos recibido tu solicitud. Validaremos disponibilidad, estado de cartera y condiciones.

Pronto te enviaremos la información completa.

Administración Podium P.H.
```

---

### 📅 13. Citas – Disponibilidad

**Asunto:** Disponibilidad de cita – Podium P.H.

```
Hola,

Por favor selecciona uno de los horarios disponibles en el enlace enviado para agendar tu cita.

Administración Podium P.H.
```

---

### 📄 14. Facturas de Proveedores – Acuse

**Asunto:** Factura recibida – Podium P.H.

```
Buen día,

Confirmamos la recepción de la factura. Fue remitida al área contable para revisión y causación.

Administración Podium P.H.
```

---

## 5️⃣ AUTOMATIZACIONES TRANSVERSALES

- ✅ Etiquetas automáticas
- ✅ Respuestas fuera de horario
- ✅ Alertas por no respuesta
- ✅ Recordatorios de cartera
- ✅ Registro para auditoría

---

## 6️⃣ VALIDACIONES OBLIGATORIAS

### ⚠️ CCTV - Validar datos obligatorios

**SI FALTAN DATOS:**
- Enviar plantilla #9 (Mantenimiento - Solicitud de información) adaptada para CCTV
- NO reenviar a seguridad hasta tener datos completos

### ⚠️ MANTENIMIENTO - Validar datos mínimos

**SI FALTAN DATOS:**
- Enviar plantilla #9 (Mantenimiento - Solicitud de información)
- NO reenviar hasta tener datos completos

---

## 7️⃣ REGLAS DE IMPLEMENTACIÓN

### ✅ FLUJO PARA OUTPUTS 0-7 (DEPARTAMENTOS):

```
Email llega → IA clasifica
  ↓
1. VALIDAR DATOS (si aplica)
   ├─ Datos completos → Continuar
   └─ Datos incompletos → Solicitar información y DETENER
  ↓
2. ACK AL RESIDENTE (plantilla específica)
  ↓
3. FORWARD A DEPARTAMENTO (email original + contexto)
  ↓
4. LABEL en Gmail
  ↓
5. LOG en Supabase
```

### ✅ FLUJO PARA OUTPUT 8 (INFO GENERAL):

```
Email llega → IA clasifica
  ↓
ACK + RESPUESTA IA (en mismo email)
  ↓
IF Auto-Approve (>= 85%)
  ├─ TRUE → Enviar directamente
  └─ FALSE → Telegram Approval → Enviar si aprueba
  ↓
LABEL + LOG
```

---

## 8️⃣ PRIORIDADES

### 🔴 ALTA PRIORIDAD (Marcar como crítico):
- Tutelas
- Demandas
- Derechos de petición
- Palabras clave: "abogado", "urgente", "emergencia"

### 🟡 MEDIA PRIORIDAD:
- CCTV (3 días hábiles)
- Convivencia
- Contabilidad

### 🟢 NORMAL:
- Información general
- Mantenimiento
- Trasteos
- Salón Social

---

## ✅ CHECKLIST DE CUMPLIMIENTO

Antes de dar por terminada la implementación, verificar:

- [ ] Todas las plantillas de email coinciden exactamente con este documento
- [ ] CCTV valida datos obligatorios antes de reenviar
- [ ] Mantenimiento valida datos mínimos antes de reenviar
- [ ] Output 8 incluye ACK + Respuesta en mismo email
- [ ] Telegram usa botones de aprobación (no texto)
- [ ] Todos los departamentos reciben email original completo
- [ ] Labels se aplican correctamente
- [ ] Supabase registra todos los casos
- [ ] Emails críticos se marcan como ALTA PRIORIDAD

---

**ESTE DOCUMENTO ES LA FUENTE DE VERDAD. NUNCA CONTRADECIR.**
