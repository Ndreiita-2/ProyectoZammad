# ¿CÓMO REEMPLAZAMOS ZENDESK?

La propuesta consiste en reemplazar Zendesk por un stack open source y autoalojado, combinando Zammad para gestión de tickets y soporte estructurado, Chatwoot para canales omnicanal y mensajería, y automatización / IA externa para optimizar procesos y productividad.

Esta solución nos permite centralizar todos los mensajes, automatizar respuestas, clasificar tickets automáticamente y mantener control total de los datos, todo sin depender de SaaS costosos.

## Stack final

* **Zammad** → Gestión avanzada de tickets y soporte interno
* **Chatwoot** → Entrada omnicanal y bandeja única de mensajes


* **n8n / Make / Zapier** → Orquestación de flujos y automatizaciones
* **OpenAI / LLM local / IA gestionada** → Inteligencia artificial para asistencia, clasificación y redacción

---

## Flujo de atención al cliente

1. Cliente envía mensaje desde cualquier canal (WhatsApp, Instagram, web, email, TikTok, etc.)
2. Chatwoot recibe el mensaje en **una bandeja unificada**
3. Webhook → crea automáticamente un ticket en Zammad
4. Zammad gestiona el ticket internamente, aplicando reglas, asignaciones y SLA
5. Respuesta del agente → vuelve a Chatwoot → cliente recibe respuesta desde el canal original

## ZAMMAD – Gestión avanzada de tickets

Zammad será la **plataforma principal de atención**, especializada en tickets y soporte estructurado.

* **Especializado en Helpdesk / Ticketing**

  * Sistema de tickets avanzado
  * Historial completo (auditado)
  * Checklists en tickets
  * Detección de duplicados
  * Multitarea (varios tickets abiertos)
  * Plantillas de respuesta

* **Multicanal**

  * Email (canal principal)
  * Web (formularios / tickets)
  * Chat web
  * SMS
  * Teléfono (CTI / VoIP)

* **Automatización y gestión operativa**

  * Reglas
  * Triggers
  * Macros
  * Escalados automáticos
  * Gestión de tiempos (SLA)
  * Priorización de tickets
  * Asignación automática

* **Personalización del sistema**

  * Campos personalizados
  * Roles y permisos
  * Grupos de trabajo
  * Plantillas
  * Flujos / procesos

* **Integración con sistemas externos**

  * API REST
  * Webhooks (Chatwoot y otros)
  * Sistemas de autenticación (SSO)
  * LDAP / Active Directory
  * Exchange (correo corporativo)
  * GitHub / GitLab
  * Sistemas de monitorización (Nagios, Zabbix, etc.)

* **IA integrada**

  * Resumen automático de tickets
  * Redacción asistida
  * Traducción de mensajes
  * Mejora de texto
  * Detección automática de idioma

* **IA externa (configurable)**

  * OpenAI / Azure
  * Modelos propios (LLM local)
  * IA gestionada por Zammad

* **AI Agents (automatización inteligente)**

  * Clasificación automática de tickets
  * Priorización de urgencia
  * Asignación a equipos
  * Reescritura de títulos

* **Gestión de clientes (tipo CRM)**

  * Perfiles de usuario
  * Historial de interacción
  * Organizaciones (empresas)
  * Integración con datos externos (Clearbit)

* **Seguridad y control**

  * Open source
  * Self-hosted
  * SSO (SAML, OpenID)
  * 2FA
  * Encriptación (S/MIME)
  * Control total de datos

---

## CHATWOOT – Entrada omnicanal y bandeja única

Chatwoot será la **plataforma secundaria**, encargada de centralizar todos los canales de comunicación y enviar mensajes a Zammad como tickets.

* **Bandeja unificada (Omnicanal)**

  * Todos los mensajes en una sola inbox
  * WhatsApp
  * Facebook Messenger
  * Instagram
  * TikTok
  * Otras redes sociales
  * Chat web (widget personalizable)
  * Email
  * API propia (para conectar cualquier canal)

* **Automatización y gestión operativa**

  * Reglas automáticas
  * Respuestas automáticas
  * SLA (tiempos de respuesta)
  * Asignación automática de conversaciones
  * Etiquetado y clasificación

* **Chatbots y automatización conversacional**

  * Dialogflow (Google)
  * Rasa (open source)
  * Flujos automatizados (árbol de decisiones)
  * Auto-respuestas
  * Clasificación de leads

* **IA integrada (Captain AI)**

  * Sugerencias automáticas de respuesta
  * Respuestas a preguntas frecuentes
  * Traducción en tiempo real
  * Mejora de respuestas
  * Uso de base de conocimiento

* **Integración con IA externa**

  * Webhooks para conexión con n8n / Make
  * API REST para flujos personalizados
  * Conexión con OpenAI u otros LLMs

---

## Analítica y Reporting

Para medir desempeño y tomar decisiones estratégicas:

* Métricas de tickets (Zammad)
* Métricas de conversación y engagement (Chatwoot)
* Exportación de datos para BI (Metabase, Power BI, Looker)
* Reportes de SLA y tiempos de respuesta
* Monitoreo de satisfacción del cliente

---

## Ventajas frente a Zendesk

* Open source → sin lock-in ni costos SaaS altos
* Autoalojado → control total de datos y privacidad
* Integración nativa con IA → automatización y asistencia inteligente
* Centralización de todos los canales → flujo omnicanal
* Personalización ilimitada → reglas, macros, flujos y roles
* Compatible con sistemas corporativos y herramientas externas

---

## Limitaciones y consideraciones

* Requiere configuración técnica y mantenimiento
* WhatsApp depende de API oficial
* Integraciones con algunas redes sociales pueden necesitar setup adicional
* No es plug & play como Zendesk

