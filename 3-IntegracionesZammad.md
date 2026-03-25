# Integración de Microsoft 365 con Zammad mediante OAuth (IMAP y SMTP)

## 1. Requisitos previos

* Cuenta activa en Microsoft 365
* Acceso de administrador en Azure
* Acceso administrador en Zammad
* Dominio verificado en Microsoft 365
* Zammad accesible por HTTPS (dominio público o túnel como ngrok)

---

# 2. Configuración inicial en Zammad (URL pública correcta)

Antes de crear la aplicación en Azure, es obligatorio que Zammad tenga configurada su URL pública correcta.

## 2.1 Cambiar la URL base del sistema

Ir a:

Admin → Sistema → Configuración → URL (o Base URL)

Cambiar la dirección IP interna (por ejemplo 192.168.x.x) por el dominio público, por ejemplo:

```
https://subdominio.ngrok-free.dev
```

Guardar los cambios.

Esto es imprescindible porque Zammad utilizará esta URL para construir automáticamente la URI de redirección OAuth.

---

# 3. Creación de la aplicación en Azure

## 3.1 Acceder al portal

1. Ir a https://portal.azure.com
2. Entrar en Microsoft Entra ID
3. Seleccionar Registros de aplicaciones
4. Pulsar Nuevo registro

---

## 3.2 Registrar la aplicación

Configurar:

* Nombre: Zammad Mail Integration
* Tipos de cuenta compatibles: Cuentas en este directorio organizativo solamente
* URI de redirección:

  * Tipo: Web
  * URI:

```
https://TU_DOMINIO_ZAMMAD/api/v1/external_credentials/microsoft365/callback
```

Ejemplo con ngrok:

```
https://subdominio.ngrok-free.dev/api/v1/external_credentials/microsoft365/callback
```

Crear la aplicación.

---

## 3.3 Obtener identificadores

Copiar:

* Id. de aplicación (cliente)
* Id. de directorio (inquilino)

---

# 4. Crear secreto del cliente

1. Ir a Certificados y secretos
2. Nuevo secreto de cliente
3. Definir duración
4. Crear

Copiar el valor del secreto inmediatamente.

---

# 5. Configurar permisos API

Ir a:

Permisos de API → Agregar un permiso

Seleccionar Microsoft Graph → Permisos delegados

Añadir:

* IMAP.AccessAsUser.All
* SMTP.Send
* offline_access
* User.Read

Guardar.

Después:

Pulsar Conceder consentimiento de administrador.

---

# 6. Configuración en Zammad

Ir a:

Admin → Channels → Email

Seleccionar:

Microsoft 365 IMAP Email

---

## 6.1 Configurar App

Pulsar Configurar App e introducir:

* Tenant ID
* Client ID
* Client Secret

Guardar.

---

## 6.2 Añadir cuenta

Pulsar Añadir cuenta.

Iniciar sesión con la cuenta de Microsoft 365 y autorizar.

Debe aparecer:

* Entrante: activo
* Saliente: activo

---

# 7. Configuración correcta del canal entrante

Editar el canal entrante y verificar:

* Grupo de destino: el deseado
* Directorio: Inbox
* Mantener mensajes en el servidor: yes

Guardar.

Esto garantiza que los correos permanezcan en el buzón tras ser procesados.


-----------------------------------------------------Zammad-Odoo----------------------------------------------------------------

Perfecto Andrea 😄  
Aquí tienes **LA GUÍA COMPLETA, PROFESIONAL, ORDENADA Y DEFINITIVA** para integrar **Zammad ↔ Odoo**, con **todas las configuraciones correctas**, incluyendo:

*   Creación del módulo
*   Rutas correctas
*   Limpieza HTML
*   Recepción de mensajes de agente y cliente
*   Triggers correctos
*   Webhook
*   Consideraciones reales según tu entorno
*   Solución de los problemas que tú tuviste

Esta guía está **lista para copiar y pegar** en tu documentación.

***

# 💙 GUÍA PROFESIONAL COMPLETA

# **Integración Zammad → Odoo (CRM) con sincronización completa de conversación**

> **Objetivo:**  
> Sincronizar automáticamente tickets de Zammad hacia Odoo CRM, incluyendo:  
> ✔ Creación del lead a partir del ticket  
> ✔ Mensajes del cliente  
> ✔ Mensajes del agente  
> ✔ Eliminación de HTML  
> ✔ Historial completo en un solo campo en Odoo

***

# 🔷 **1. Requisitos previos**

## 1.1 Odoo instalado sin Docker

Tu Odoo está funcionando como:

    Servicio: odoo.service
    Puerto: 8069
    BD: odoo
    Usuario: odoo

## 1.2 Crear carpeta de módulos externos

    sudo mkdir -p /mnt/extra-addons
    sudo chown -R odoo:odoo /mnt/extra-addons

## 1.3 Configurar addons\_path

Editar:

    sudo nano /etc/odoo/odoo.conf

Debe contener:

    addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons

Reiniciar Odoo:

    sudo systemctl restart odoo

***

# 🔷 **2. Crear módulo de integración zammad\_bridge**

Ruta:

    /mnt/extra-addons/zammad_bridge

Crear estructura:

    zammad_bridge/
    ├── __init__.py
    ├── __manifest__.py
    ├── models/
    │   ├── __init__.py
    │   └── crm_lead.py
    └── controllers/
        ├── __init__.py
        └── main.py

***

# 🔷 **3. Archivos del módulo**

## 3.1 `__manifest__.py`  **(nombre obligatorio)**

```python
{
   'name': 'Zammad Bridge',
   'version': '1.1',
   'category': 'Tools',
   'summary': 'Sync Zammad tickets & conversations to Odoo CRM',
   'depends': ['crm'],
   'data': [],
   'installable': True,
   'application': False,
}
```

## 3.2 `__init__.py`

```python
from . import models
from . import controllers
```

## 3.3 `models/__init__.py`

```python
from . import crm_lead
```

## 3.4 `models/crm_lead.py`

```python
from odoo import models, fields

class Lead(models.Model):
    _inherit = 'crm.lead'

    zammad_ticket_id = fields.Integer("Zammad Ticket ID")
```

***

# 🔷 **4. Controlador para recibir tickets y mensajes — `main.py`**

Este archivo:

*   Recibe JSON completo de Zammad
*   Limpia HTML
*   Diferencia cliente/agente
*   Guarda conversación completa en el Lead

### 📄 **main.py (versión final corregida)**

```python

from odoo import http, fields
from odoo.http import request
import json
import re


def clean_html(text):
    if not text:
        return ""
    clean = re.sub(r'<[^>]+>', '', text)
    clean = clean.replace("&nbsp;", " ").strip()
    return clean


class ZammadWebhook(http.Controller):

    @http.route('/api/zammad_ticket', type='json', auth='public', methods=['POST'], csrf=False)
    def receive_ticket(self, **kwargs):
        try:
            raw_json = request.httprequest.data.decode("utf-8")
            data = json.loads(raw_json or "{}")

            ticket = data.get('ticket', {})
            state = (ticket.get('state') or "").lower()
            state_id = ticket.get('state_id')
            article = data.get('article', {})

            ticket_id = ticket.get('id') or article.get('ticket_id')
            title = ticket.get('title', 'Sin título')

            # =============================
            # 👤 CLIENTE
            # =============================
            customer = ticket.get('customer', {}) or {}

            customer_name = (
                customer.get('fullname')
                or f"{customer.get('firstname', '')} {customer.get('lastname', '')}".strip()
                or "Cliente"
            )

            customer_email = customer.get('email') or f"guest_{ticket_id}@chat.local"

            # =============================
            # 🧑💼 AGENTE
            # =============================
            created_by = article.get('created_by') or {}

            agent_name = (
                created_by.get('fullname')
                or f"{created_by.get('firstname', '')} {created_by.get('lastname', '')}".strip()
                or "Agente"
            )

            agent_email = created_by.get('email') or f"agente_{ticket_id}@empresa.com"

            # =============================
            # 💬 MENSAJE
            # =============================
            raw_body = article.get('body', '')
            body = clean_html(raw_body)

            sender = article.get("sender")

            if sender == 1:
                author_name = customer_name
                author_email = customer_email
            else:
                author_name = agent_name
                author_email = agent_email

            # No incluir el nombre dentro del cuerpo del mensaje
            message_final = body  # El cuerpo del mensaje sin el nombre del autor

            # =============================
            # 🧑🤝🧑 PARTNERS
            # =============================
            Partner = request.env['res.partner'].sudo()

            customer_partner = Partner.search([
                ('email', '=', customer_email),
                ('user_ids', '=', False)
            ], limit=1)

            if not customer_partner:
                customer_partner = Partner.create({
                    'name': customer_name,
                    'email': customer_email
                })

            agent_partner = Partner.search([
                ('email', '=', agent_email)
            ], limit=1)

            if not agent_partner:
                agent_partner = Partner.create({
                    'name': agent_name,
                    'email': agent_email
                })

            # =============================
            # 🎯 LEAD
            # =============================
            Lead = request.env['crm.lead'].sudo()

            lead = Lead.search([
                ('zammad_ticket_id', '=', ticket_id)
            ], limit=1)

            if not lead:
                lead = Lead.create({
                    'name': title,
                    'email_from': customer_email,
                    'partner_id': customer_partner.id,
                    'zammad_ticket_id': ticket_id,
                    'type': 'lead',
                })

            # asegurar partner
            if lead.partner_id != customer_partner:
                lead.partner_id = customer_partner.id

            # followers
            lead.message_subscribe([customer_partner.id])
            lead.message_subscribe([agent_partner.id])

            # =============================
            # 🔒 CIERRE
            # =============================
            if state == "closed" or state_id in [4, 5]:
                lead.write({
                    'date_closed': fields.Datetime.now()
                })

            # =============================
            # 💬 MENSAJE
            # =============================
            if body:
                lead.message_post(
                    body=message_final,  # Cuerpo solo con el texto limpio
                    author_id=(customer_partner.id if sender == 1 else agent_partner.id),
                    email_from=f"{author_name} <{author_email}>",  # Nombre en el encabezado
                    message_type='comment',
                    subtype_xmlid='mail.mt_comment'
                )

            return {'status': 'ok', 'lead_id': lead.id}

        except Exception as e:
            return {'status': 'error', 'error': str(e)}

```

***

# 🔷 **5. Reiniciar Odoo**

    sudo systemctl restart odoo

***

# 🔷 **6. Crear Webhook en Zammad**

**Admin → Integrations → Webhooks → New**

*   Nombre:

<!---->

    Sync Odoo

*   URL:

<!---->

    http://IP_DE_ODOO:8069/api/zammad_ticket

*   Método:

<!---->

    POST

*   Content-Type:

<!---->

    application/json

Guardar.

***

# 🔷 **7. Crear Triggers correctos en Zammad**

## 7.1 Trigger 1 — **Agente → Odoo**

✔ Envía notas públicas (respuestas del agente)

**Condiciones:**

    Artículo → Tipo → es → nota
    Artículo → Visibilidad → es → público
    Artículo → Remitente → es → agente

**Acción:**

    Webhook → Sync Odoo

***

## 7.2 Trigger 2 — **Cliente → Odoo (Universal FIX)**

Este es el **importante**, porque captura todos los tipos de mensajes entrantes.

**Condiciones EXACTAS:**

    Artículo → Tipo → no es → note
    Artículo → Visibilidad → es → público
    Artículo → Remitente → no contiene → agent

**Acción:**

    Webhook → Sync Odoo
    
