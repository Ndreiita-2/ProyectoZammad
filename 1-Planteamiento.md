# ¿CÓMO REEMPLAZAMOS ZENDESK?

## ZAMMAD
La idea parte de un software de código abierto llamado **Zammad**, el cuál ya cuenta con un sistema de creación y gestión de tickets muy completo, sin necesidad de meterle módulos ni nada adicional. 

Según el planteamiento, ésta es la plataforma de atención al cliente principal, por la cuál se van a responder a todos los mensajes mediante los tickets que se crean automáticamente.

Además de ser de lo más intuitivo a nivel sistemas, cuenta con las siguientes ventajas frente a cualquier otro software:

- Especializado en Helpdesk  
- Es de código abierto y autoalojado
- Tiene multicanal activo
- Sistemas de automatización avanzada
    - Reglas
    - Triggers
    - Macros
    - Escalados
- Gestión de respuestas avanzada
    - Tiempos
    - Prioridades
- Personalización
    - Campos
    - Roles
    - Grupos
    - Plantillas
    - Procesos
- Integración con sistemas corporativos
    - Webhooks
    - Auth
    - API
  
## Chatwoot
Chatwoot es el software secundario, también de código abierto y autoalojado, que necesitamos para integrar las redes sociales y el chat de soporte en la web, teniendo todos los mensajes centralizados en un solo canal.

Éste se integra con Zammad mediante Webhooks, para que todos los mensajes que llegan al canal de Chatwoot creen ticket en Zammad y puedan ser respondidos ahí mismo.

Aquí nombro sus ventajas:

- Todos los mesnajes llegan a una sola bandeja (inbox)
    - WhatsApp 
    - Facebook Messenger
    - Instagram 
    - TikTok
    - Muchas más RRSS ...

    - Chat web personalizable (widget en la web)
    - API propia (para conectar lo que queramos)

- IA integrada (Captain AI)
    - Sugiere respuestas automáticamente
    - Responde preguntas frecuentes
    - Traduce mensajes en tiempo real
    - Aprende de tu base de conocimiento


- Chatbots (automatización real)
    - Dialogflow (Google)
    - Rasa (open source)
    - Flujos automatizados (tipo árbol de decisiones)
    - Auto-respuestas y clasificación de leads
- Automatizaciones (muy clave)
    - Reglas 
    - Respuestas automáticas
    - SLA (tiempos de respuesta)
- Integraciones con IA externas (lo más potente)
    - Webhooks (envía/recibe datos en tiempo real)
    - API REST completa
