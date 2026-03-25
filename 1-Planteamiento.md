# ¿CÓMO REEMPLAZAMOS ZENDESK?

## ZAMMAD
La idea parte de un software helpdesk tipo ticketing de código abierto llamado **Zammad**, el cuál ya cuenta con un sistema de creación y gestión de tickets muy completo, sin necesidad de meterle módulos ni nada adicional. 

Según el planteamiento, ésta es la plataforma de atención al cliente principal, por la cuál se van a responder a todos los mensajes mediante los tickets que se crean automáticamente.

Además de ser de lo más intuitivo a nivel sistemas, cuenta con las siguientes ventajas frente a cualquier otro software:

* Especializado en Helpdesk / Ticketing

  * Sistema de tickets avanzado
  * Historial completo (auditado)
  * Checklists en tickets
  * Detección de duplicados
  * Multitarea (varios tickets abiertos)
  * Plantillas de respuesta

* Multicanal

  * Email (canal principal)
  * Web (formularios / tickets)
  * Chat web
  * SMS
  * Teléfono (CTI / VoIP)

* Automatización y gestión operativa

  * Reglas
  * Triggers
  * Macros
  * Escalados automáticos
  * Gestión de tiempos (SLA)
  * Priorización de tickets
  * Asignación automática

* Personalización del sistema

  * Campos personalizados
  * Roles y permisos
  * Grupos de trabajo
  * Plantillas
  * Flujos / procesos

* Integración con sistemas externos

  * API REST
  * Webhooks (Chatwoot)
  * Sistemas de autenticación (SSO)
  * LDAP / Active Directory
  * Exchange (correo corporativo)
  * GitHub / GitLab
  * Sistemas de monitorización (Nagios, etc.)

* IA integrada

  * Resumen automático de tickets
  * Redacción asistida
  * Traducción de mensajes
  * Mejora de texto
  * Detección automática de idioma

* IA externa (configurable)

  * OpenAI / Azure
  * Modelos propios (LLM local)
  * IA gestionada por Zammad

* AI Agents (automatización inteligente)

  * Clasificación automática de tickets
  * Priorización de urgencia
  * Asignación a equipos
  * Reescritura de títulos

* Gestión de clientes (tipo CRM)

  * Perfiles de usuario
  * Historial de interacción
  * Organizaciones (empresas)
  * Integración con datos externos (Clearbit)

* Seguridad y control

  * Open source
  * Self-hosted
  * SSO (SAML, OpenID)
  * 2FA
  * Encriptación (S/MIME)
  * Control total de datos
---

## Chatwoot
Chatwoot es el software secundario, también de código abierto y autoalojado, que necesitamos para integrar las redes sociales y el chat de soporte en la web, teniendo todos los mensajes centralizados en un solo canal.

Éste se integra con Zammad mediante Webhooks, para que todos los mensajes que llegan al canal de Chatwoot creen ticket en Zammad y puedan ser respondidos ahí mismo.

Aquí nombro sus ventajas:

* Bandeja unificada (Omnicanal)

  * Todos los mensajes en una sola inbox
  * WhatsApp
  * Facebook Messenger
  * Instagram
  * TikTok
  * Otras redes sociales
  * Chat web (widget personalizable)
  * Email
  * API propia (para conectar cualquier canal)

* Automatización y gestión operativa

  * Reglas automáticas
  * Respuestas automáticas
  * SLA (tiempos de respuesta)
  * Asignación automática de conversaciones
  * Etiquetado y clasificación

* Chatbots y automatización conversacional

  * Dialogflow (Google)
  * Rasa (open source)
  * Flujos automatizados (árbol de decisiones)
  * Auto-respuestas
  * Clasificación de leads

* IA integrada (Captain AI)

  * Sugerencias automáticas de respuesta
  * Respuestas a preguntas frecuentes
  * Traducción en tiempo real
  * Mejora de respuestas
  * Uso de base de conocimiento

* Integración con IA externa

  * Webhooks (envío y recepción en tiempo real)
  * API REST completa
  * Integración con herramientas externas (n8n, Zapier, Make, etc.)
  * Conexión con modelos como OpenAI u otros LLMs
