# 📋 Podium Workflow Documentation

Documentación completa del workflow de automatización de correos para Conjunto Residencial Podium P.H.

## 🎯 Objetivo

Sistema automatizado de clasificación y gestión de correos electrónicos usando n8n, OpenAI y Gmail API.

## 📊 Características

- ✅ Clasificación automática con IA (9 categorías)
- ✅ Validación de datos obligatorios (CCTV y Mantenimiento)
- ✅ ACK específico por categoría
- ✅ Forward de email original a departamentos
- ✅ Respuesta automática para información general
- ✅ Aprobación manual vía Telegram
- ✅ Logging completo en Supabase

## 📁 Estructura del Repositorio

```
podium-workflow-documentation/
├── README.md                           # Este archivo
├── docs/                               # Documentación principal
│   ├── REQUERIMIENTOS_MAESTROS_DON_JORGE.md
│   ├── PLAN_COMPLETO_IMPLEMENTACION.md
│   └── MAQUETA_FLUJO_CORRECTO.md
├── implementation/                     # Guías paso a paso
│   ├── PASO_1_CODE_EXTRACTOR.md
│   ├── PASO_2_OUTPUT_8.md
│   └── PASO_3_VALIDACION_DATOS.md
├── code/                              # Código JavaScript
│   ├── code_extractor.js
│   ├── validar_datos_cctv.js
│   └── validar_datos_mantenimiento.js
└── diagrams/                          # Diagramas visuales
    └── workflow_completo.png
```

## 🚀 Implementación

### Fase 1: Code Extractor ✅
- Extracción robusta de datos
- Normalización de campos
- Manejo de fechas y emails

### Fase 2: Output 8 (Info General) 🔄
- ACK + Respuesta IA en mismo email
- Telegram Approval con botones
- Validación de aprobación

### Fase 3: Validación de Datos 📋
- CCTV: 6 campos obligatorios
- Mantenimiento: 3 campos mínimos
- Solicitud automática de datos faltantes

### Fase 4: Outputs Departamentos 📧
- 8 categorías con ACK + Forward
- Email original completo
- Labels y logging

## 📧 Categorías

1. **LEGAL** - Tutelas, demandas, derechos de petición
2. **CCTV** - Solicitudes de revisión de cámaras (con validación)
3. **CONVIVENCIA** - Reportes entre residentes
4. **CONTABILIDAD** - Pagos y estados de cuenta
5. **FACTURAS** - Facturas de proveedores
6. **MANTENIMIENTO** - Reparaciones (con validación)
7. **TRASTEOS** - Mudanzas
8. **SALON_SOCIAL** - Reservas de salón
9. **INFO_GENERAL** - Preguntas generales (respuesta IA)

## 🛠️ Tecnologías

- **n8n** - Automatización de workflows
- **OpenAI GPT-4** - Clasificación y respuestas
- **Gmail API** - Gestión de correos
- **Telegram Bot** - Aprobaciones manuales
- **Supabase** - Base de datos y logging
- **Vector Store** - RAG con Manual de Convivencia

## 📝 Requerimientos

Ver [REQUERIMIENTOS_MAESTROS_DON_JORGE.md](docs/REQUERIMIENTOS_MAESTROS_DON_JORGE.md) para especificaciones completas.

## 📊 Estadísticas

- **Nodos totales:** 56
- **Categorías:** 9
- **Plantillas de email:** 14
- **Validaciones:** 2 (CCTV y Mantenimiento)

## 👥 Autor

Desarrollado para Conjunto Residencial Podium P.H.

## 📄 Licencia

Documentación privada - Uso interno únicamente
