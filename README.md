# 📧 Email Triage Automático — n8n + Claude API + Airtable + Slack

> Workflow de automatización para clasificación inteligente de correos de clientes usando un LLM (Claude de Anthropic), con registro en Airtable y escalación por Slack.

---

## 🧠 Descripción

Este workflow automatiza la recepción, clasificación y enrutamiento de correos entrantes en una bandeja de soporte. Utiliza la API de Claude para analizar el contenido de cada correo y devolver una categoría, nivel de urgencia y resumen en formato JSON estructurado.

**Tiempo de ejecución:** ~887ms por correo  
**Estado:** ✅ Producción

---

## ⚙️ Stack tecnológico

| Herramienta | Rol |
|-------------|-----|
| [n8n](https://n8n.io) | Orquestador del workflow (self-hosted) |
| Gmail API v1 | Trigger de entrada y archivo de spam |
| [Claude API](https://console.anthropic.com) (claude-sonnet-4) | Clasificación con LLM |
| Airtable API v0 | Base de datos de clasificaciones |
| Slack API | Notificaciones de escalación urgente |

---

## 🔁 Flujo del workflow

```
Gmail Trigger
    └── Limpiar y Preparar Texto
        └── Construir Prompt de Clasificación
            └── Clasificar con Claude API  ← POST api.anthropic.com/v1/messages
                └── Parsear Clasificación (JSON)
                    └── Guardar en Airtable
                        └── Evaluar Enrutamiento
                            ├── [urgencia = alta] → IF Escalación Urgente → Notificar Slack
                            └── [categoría = spam] → IF Es Spam → Archivar en Gmail
```

---

## 📋 Nodos del workflow

| Nodo | Tipo | Descripción |
|------|------|-------------|
| Gmail Trigger — Nuevo Correo | Trigger | Detecta correos nuevos en la bandeja configurada |
| Limpiar y Preparar Texto | Code | Extrae texto plano, elimina HTML, trunca a 500 chars |
| Construir Prompt de Clasificación | Code | Arma el prompt estructurado para el LLM |
| Clasificar con Claude | HTTP Request | POST a Claude API, devuelve JSON con categoría/urgencia/resumen |
| Parsear Clasificación | Code | Extrae y valida el JSON de la respuesta del LLM |
| Guardar en Airtable | Airtable | Crea registro con metadatos del correo clasificado |
| Evaluar Enrutamiento | Code | Determina la rama según categoría y urgencia |
| IF Escalación Urgente | IF | Bifurca si urgencia = alta |
| Notificar Slack — Escalación | Slack | Envía alerta al canal #escalaciones |
| IF Es Spam | IF | Bifurca si categoría = spam |
| Archivar Correo en Gmail | Gmail | Aplica etiqueta y archiva el correo |

---

## 🔐 Configuración de credenciales

> ⚠️ **Nunca hardcodees credenciales en el workflow.** Usar el Credential Store de n8n.

### Variables necesarias

| Credencial | Tipo | Dónde obtenerla |
|------------|------|-----------------|
| `ANTHROPIC_API_KEY` | API Key | [console.anthropic.com](https://console.anthropic.com) |
| Gmail OAuth2 | OAuth2 | Google Cloud Console — scopes: `gmail.readonly`, `gmail.modify` |
| Airtable PAT | Personal Access Token | [airtable.com/account](https://airtable.com/account) — scopes: `data.records:read`, `data.records:write` |
| Slack Bot Token | `xoxb-...` | [api.slack.com/apps](https://api.slack.com/apps) — scope: `chat:write` |

### Pasos para configurar en n8n

1. Ir a **Settings > Credentials > New**
2. Crear cada credencial con el tipo correspondiente
3. Referenciarlas en los nodos correspondientes del workflow

---

## 🚀 Instalación y uso

### Requisitos

- n8n v1.x (self-hosted o Cloud)
- Node.js 18+
- Cuentas activas en: Google, Anthropic, Airtable, Slack

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/email-triage-automatico.git
cd email-triage-automatico

# 2. Importar el workflow en n8n
#    n8n > Workflows > Import > seleccionar email_triage.json

# 3. Configurar las credenciales (ver sección anterior)

# 4. Activar el workflow en n8n

# 5. Enviar un correo de prueba a la bandeja configurada
#    y verificar en n8n > Executions que todos los nodos muestren Success
```

---

## 📊 Métricas de rendimiento (ejecución de prueba)

| Nodo | Tiempo | Estado |
|------|--------|--------|
| Gmail Trigger | 1.346s | ✅ Success |
| Limpiar y Preparar Texto | 139ms | ✅ Success |
| Construir Prompt | 19ms | ✅ Success |
| **Clasificar con Claude API** | **1.582s** | ✅ Success |
| Parsear Clasificación | 20ms | ✅ Success |
| **Guardar en Airtable** | **1.123s** | ✅ Success |
| Evaluar Enrutamiento | 18ms | ✅ Success |
| IF Escalación Urgente | 2ms | ✅ Success |
| Notificar Slack | 477ms | ✅ Success |
| **Total** | **887ms** | ✅ Success |

---

## 🗂️ Estructura del repositorio

```
email-triage-automatico/
├── README.md
├── workflow/
│   └── email_triage.json        # Workflow exportado de n8n
├── docs/
│   ├── arquitectura.png         # Diagrama del workflow
│   └── logs_ejecucion.png       # Evidencia de ejecución exitosa
└── .gitignore
```

---

## 🔒 Seguridad

- Todas las credenciales se gestionan exclusivamente a través del **Credential Store de n8n**
- El servidor n8n está protegido con **Cloudflare Zero Trust**
- Cada integración tiene permisos de **mínimo privilegio**
- Las credenciales se rotan cada **90 días**
- El workflow exportado (`email_triage.json`) **no contiene** ninguna credencial

---

## 🛠️ Mantenimiento

| Frecuencia | Tarea |
|------------|-------|
| Diario | Monitoreo de errores — alerta automática a Slack si hay fallos |
| Semanal | Revisión de logs y latencia en n8n Executions |
| Mensual | Revisión del prompt de clasificación según feedback del equipo |
| Trimestral | Rotación de credenciales y auditoría de accesos |
| Ante cambios | Exportar JSON actualizado y commit a este repositorio |

---

## 📄 Licencia

Este proyecto es de uso educativo. Todos los derechos reservados © Ana Ferreira, 2026.  
Para consultoría o implementación profesional: contactar a través de GitHub.

---

> Desarrollado como proyecto final del módulo de **Automatización de Procesos con Herramientas No-Code, APIs y LLMs** — 2026
