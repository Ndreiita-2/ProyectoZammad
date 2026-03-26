--------------------------------------------------------------------------------------------------------------------------------------------------------
# Conexión entre Zammad 7 y Chatwoot 4.12.1 mediante middleware Node.js

---

# PARTE 1 — Crear el middleware Node.js desde cero

## 1. Instalar Node

En el servidor donde está Zammad:

```bash
node -v
```

Si no está instalado:

```bash
sudo apt update
sudo apt install nodejs npm -y
```

---

## 2. Crear carpeta del middleware

```bash
mkdir ~/chat-integration
cd ~/chat-integration
npm init -y
```

---

## 3. Instalar dependencias

```bash
npm install express axios dotenv
```

---

## 4. Ajustar package.json para ES Modules

Editar package.json y añadir:

```json
"type": "module"
```

---

## 5. Crear archivo .env

```bash
nano .env
```

Contenido:

```env
PORT=4000

ZAMMAD_URL=http://localhost:8080
ZAMMAD_TOKEN=TU_TOKEN_ZAMMAD

CHATWOOT_URL=http://192.168.136.Y:3000
CHATWOOT_TOKEN=TU_TOKEN_CHATWOOT
ACCOUNT_ID=1
```

Notas importantes:

* No incluir rutas como /app o /api
* Solo la URL base
* En producción usar dominio con HTTPS

---

## 6. Crear index.js

```bash
nano index.js
```

Código completo actualizado y compatible con Zammad 7 y Chatwoot 4.12.1:

```javascript
import express from "express";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

// Persistencia en memoria (usar Redis en producción)
const conversationMap = {};

/*
====================================
CHATWOOT → ZAMMAD
====================================
*/
app.post("/chatwoot", async (req, res) => {
  try {
    const event = req.body?.event;

    if (event !== "message_created") {
      return res.sendStatus(200);
    }

    const messageType = req.body?.message_type;
    if (messageType !== "incoming") {
      return res.sendStatus(200);
    }

    const conversationId = req.body?.conversation?.id;
    const message = req.body?.content || "";
    const contact = req.body?.sender || {};

    if (!conversationId) return res.sendStatus(200);

    const email = contact?.email;
    const name = contact?.name || "Sin nombre";

    if (!email) return res.sendStatus(200);

    let ticketId = conversationMap[conversationId];

    if (!ticketId) {
      let customerId;

      const newUser = await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/users`,
        {
          firstname: name,
          lastname: "-",
          email: email,
          role_ids: [3]
        },
        {
          headers: {
            Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
            "Content-Type": "application/json"
          }
        }
      ).catch(() => null);

      if (newUser) {
        customerId = newUser.data.id;
      } else {
        const userSearch = await axios.get(
          `${process.env.ZAMMAD_URL}/api/v1/users/search?query=${email}`,
          {
            headers: {
              Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`
            }
          }
        );
        customerId = userSearch.data[0]?.id;
      }

      const newTicket = await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/tickets`,
        {
          title: `Chat #${conversationId} - ${name}`,
          group: "Users",
          customer_id: customerId,
          article: {
            subject: "Nuevo mensaje desde Chatwoot",
            body: message.replace(/<[^>]*>?/gm, ""),
            type: "note",
            internal: false
          }
        },
        {
          headers: {
            Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
            "Content-Type": "application/json"
          }
        }
      );

      ticketId = newTicket.data.id;
      conversationMap[conversationId] = ticketId;

      return res.sendStatus(200);
    }

    await axios.post(
      `${process.env.ZAMMAD_URL}/api/v1/ticket_articles`,
      {
        ticket_id: ticketId,
        subject: "Nuevo mensaje desde Chatwoot",
        body: message.replace(/<[^>]*>?/gm, ""),
        type: "note",
        internal: false
      },
      {
        headers: {
          Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
          "Content-Type": "application/json"
        }
      }
    );

    return res.sendStatus(200);

  } catch (error) {
    console.error("Error Chatwoot → Zammad:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

/*
====================================
ZAMMAD → CHATWOOT
====================================
*/
app.post("/zammad", async (req, res) => {
  try {
    const article = req.body?.article;
    const ticket = req.body?.ticket;

    if (!article || article.internal) return res.sendStatus(200);

    const ticketId = ticket?.id;

    const conversationId = Object.keys(conversationMap)
      .find(key => conversationMap[key] === ticketId);

    if (!conversationId) return res.sendStatus(200);

    await axios.post(
      `${process.env.CHATWOOT_URL}/api/v1/accounts/${process.env.ACCOUNT_ID}/conversations/${conversationId}/messages`,
      {
        content: article.body,
        message_type: "outgoing"
      },
      {
        headers: {
          api_access_token: process.env.CHATWOOT_TOKEN,
          "Content-Type": "application/json"
        }
      }
    );

    return res.sendStatus(200);

  } catch (error) {
    console.error("Error Zammad → Chatwoot:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

app.listen(process.env.PORT || 4000, () => {
  console.log(`Middleware corriendo en puerto ${process.env.PORT || 4000}`);
});
```

---

## 7. Ejecutar middleware

```bash
node index.js
```

Salida esperada:

```
Middleware corriendo en puerto 4000
```

---

# PARTE 2 — Integración con Nginx

Editar configuración:

```bash
sudo nano /etc/nginx/sites-available/zammad.conf
```

Añadir dentro de server:

```nginx
location /webhook/chatwoot {
    proxy_pass http://localhost:4000/chatwoot;
}

location /webhook/zammad {
    proxy_pass http://localhost:4000/zammad;
}
```

Reiniciar Nginx:

```bash
sudo systemctl restart nginx
```

---

# PARTE 3 — Configuración en Chatwoot

Ir a:

Settings → Integrations → Webhooks

Añadir:

URL:

```
http://TU_SERVIDOR/webhook/chatwoot
```

Evento:

```
message_created
```

---

# PARTE 4 — Configuración en Zammad

Ir a:

Admin → Webhooks → New Webhook

Configurar:

URL:

```
http://TU_SERVIDOR/webhook/zammad
```

Evento:

```
Ticket update
```

---

# PARTE 5 — Tokens necesarios

## Token de Zammad

Ruta:

Admin → Security → API

Activar API

Luego:

Avatar → Profile → Token Access

Crear token y guardarlo

---

## Token de Chatwoot

Ruta:

Profile Settings → Access Tokens

Crear token

Notas:

* No usar inbox token
* No usar webhook token
* Usar únicamente API Access Token

---

# PARTE 6 — Consideraciones importantes

Persistencia:

* El sistema usa memoria
* Si reinicias, se pierden relaciones
* Recomendado usar Redis o base de datos

Producción:

* Usar HTTPS
* Usar dominio real
* Proteger endpoints con firewall

Seguridad:

* Limitar acceso por IP
* Añadir autenticación en webhooks si es necesario

---

# RESULTADO FINAL

* Mensaje en Chatwoot crea ticket en Zammad
* Respuesta en Zammad aparece en Chatwoot
* Comunicación bidireccional funcional
* Compatible con versiones actuales sin errores comunes

--------------------------------------------------------------------------------------------------------------------------------------------------------
# 🧱 PARTE 1 — Crear el middleware Node.js desde cero

## 1️⃣ Instalar Node

En el servidor donde está Zammad:

```bash
node -v
```

Si no está instalado:

```bash
sudo apt update
sudo apt install nodejs npm -y
```

---

## 2️⃣ Crear carpeta del middleware

```bash
mkdir chat-integration
cd chat-integration
npm init -y
```

---

## 3️⃣ Instalar dependencias

```bash
npm install express axios dotenv
```

---

## 4️⃣ Crear archivo `.env`

```bash
nano .env
```

Contenido:

```env
PORT=4000

ZAMMAD_URL=http://localhost:3000
ZAMMAD_TOKEN=TU_TOKEN_ZAMMAD

CHATWOOT_URL=https://chatwoot.tudominio.com
CHATWOOT_TOKEN=TU_TOKEN_CHATWOOT
ACCOUNT_ID=1
```

Guardar.

---

## 5️⃣ Crear `index.js`

```bash
nano index.js
```

Pega esto:

```js
import express from "express";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

/*
=============================
1️⃣ Chatwoot → Zammad
=============================
*/
app.post("/chatwoot", async (req, res) => {

  if (req.body.event !== "conversation_created") {
    return res.sendStatus(200);
  }

  const conversation = req.body.conversation;
  const contact = conversation.contact;

  try {
    await axios.post(
      `${process.env.ZAMMAD_URL}/api/v1/tickets`,
      {
        title: `Chat #${conversation.id} - ${contact.name}`,
        group: "Users",
        customer: contact.email || "chatwoot@local",
        article: {
          subject: "Nuevo mensaje desde Chatwoot",
          body: conversation.messages?.[0]?.content || "Nuevo chat iniciado",
          type: "note",
          internal: false
        },
        custom_fields: {
          chatwoot_id: conversation.id
        }
      },
      {
        headers: {
          Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`
        }
      }
    );

    console.log("✅ Ticket creado en Zammad");
    res.sendStatus(200);

  } catch (error) {
    console.error("❌ Error creando ticket:", error.response?.data || error.message);
    res.sendStatus(500);
  }
});


/*
=============================
2️⃣ Zammad → Chatwoot
=============================
*/
app.post("/zammad", async (req, res) => {

  const article = req.body.article;
  const ticket = req.body.ticket;

  if (!article || article.internal) return res.sendStatus(200);

  const chatwootId = ticket.custom_fields?.chatwoot_id;
  if (!chatwootId) return res.sendStatus(200);

  try {
    await axios.post(
      `${process.env.CHATWOOT_URL}/api/v1/accounts/${process.env.ACCOUNT_ID}/conversations/${chatwootId}/messages`,
      {
        content: article.body,
        message_type: "outgoing"
      },
      {
        headers: {
          api_access_token: process.env.CHATWOOT_TOKEN
        }
      }
    );

    console.log("📨 Respuesta enviada a Chatwoot");
    res.sendStatus(200);

  } catch (error) {
    console.error("❌ Error enviando a Chatwoot:", error.response?.data || error.message);
    res.sendStatus(500);
  }
});


app.listen(process.env.PORT, () => {
  console.log(`🚀 Middleware corriendo en puerto ${process.env.PORT}`);
});
```

---

## Ajustar package.json para ES Modules

Editar `package.json` y añadir:

```json
"type": "module"
```

---

## Ejecutar middleware

```bash
node index.js
```

Debe decir:

```
🚀 Middleware corriendo en puerto 4000
```

---

# PARTE 2 — Integrarlo con tu reverse proxy

En nginx añade:

En zammad.conf, dentro de servers.

```nginx
location /webhook/chatwoot {
    proxy_pass http://localhost:4000/chatwoot;
}

location /webhook/zammad {
    proxy_pass http://localhost:4000/zammad;
}
```

Reiniciar nginx:

```bash
sudo systemctl restart nginx
```

Ahora:

```
https://subdominio.ngrok-free.dev/webhook/chatwoot
https://subdominio.ngrok-free.dev/webhook/zammad
```

---

# PARTE 3 — Configuración en Chatwoot

En Chatwoot:

Settings → Integrations → Webhooks

Evento:

```
conversation_created
```

URL:

```
https://subdominio.ngrok-free.dev/webhook/chatwoot
```

---

# PARTE 4 — Configuración en Zammad

En Zammad:

Admin → Webhooks → Nuevo

Evento:

```
Ticket updated
```

URL:

```
https://Z.ngrok-free.dev/webhook/zammad
```


**GUÍA FINAL DEFINITIVA** para conectar:

👉 Chatwoot
👉 Zammad

---

# 🧠 ARQUITECTURA FINAL

```
Chatwoot  →  Middleware Node  →  Zammad
Chatwoot  ←  Middleware Node  ←  Zammad
```

El middleware hace toda la magia.

Carpeta:

```
~/chat-integration
```

---

# 1️⃣ CREAR TOKENS CORRECTOS

---

## 🔹 TOKEN DE ZAMMAD

Entra en Zammad:

```
Admin → Security → API
```

Activa API si no está activa.

Luego:

```
Avatar → Profile → Token Access
```

Crea un nuevo token.

Guárdalo.

---

## 🔹 TOKEN DE CHATWOOT (IMPORTANTE)

Entra en Chatwoot:

```
Avatar (arriba derecha)
→ Profile Settings
→ Access Tokens
→ Create New Token
```

⚠️ ESTE es el token correcto
⚠️ NO usar inbox token
⚠️ NO usar webhook token

Guárdalo.

---

# 2️⃣ CREAR PROYECTO NODE

```bash
mkdir ~/chat-integration
cd ~/chat-integration
npm init -y
npm install express axios dotenv
```

En `package.json` añade:

```json
"type": "module"
```

---

# 3️⃣ CREAR ARCHIVO .env

```bash
nano .env
```

Contenido correcto:

```env
PORT=4000

ZAMMAD_URL=http://localhost:8080
ZAMMAD_TOKEN=TU_TOKEN_DE_ZAMMAD

CHATWOOT_URL=http://192.168.136.121:3000
CHATWOOT_TOKEN=TU_TOKEN_API_DE_CHATWOOT
ACCOUNT_ID=1
```

⚠️ IMPORTANTE:

* NO pongas `/app`
* NO pongas `/accounts`
* SOLO la raíz del servidor

Ejemplo correcto:

```
http://192.168.136.121:3000
```

---

# 4️⃣ INDEX.JS FINAL CORRECTO

Reemplaza TODO por esto:

```js
import express from "express";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

// Mapa en memoria conversación → ticket
const conversationMap = {};

/*
====================================
CHATWOOT → ZAMMAD
====================================
*/
app.post("/chatwoot", async (req, res) => {
  try {
    const event = req.body?.event;
    if (!event) return res.sendStatus(200);

    if (event === "message_created" && req.body.message_type === "incoming") {

      const conversationId = req.body?.conversation?.id;
      const message = req.body?.content || "";
      const contact = req.body?.sender || {};

      if (!conversationId) return res.sendStatus(200);

      const email = contact?.email;
      const name = contact?.name || "Sin nombre";

      if (!email) return res.sendStatus(200);

      let ticketId = conversationMap[conversationId];

      // SI NO EXISTE → CREAR TICKET
      if (!ticketId) {

        let customerId;

        const newUser = await axios.post(
          `${process.env.ZAMMAD_URL}/api/v1/users`,
          {
            firstname: name,
            lastname: "-",
            email: email,
            role_ids: [3]
          },
          {
            headers: {
              Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
              "Content-Type": "application/json"
            }
          }
        ).catch(() => null);

        if (newUser) {
          customerId = newUser.data.id;
        } else {
          const userSearch = await axios.get(
            `${process.env.ZAMMAD_URL}/api/v1/users/search?query=${email}`,
            {
              headers: {
                Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`
              }
            }
          );
          customerId = userSearch.data[0]?.id;
        }

        const newTicket = await axios.post(
          `${process.env.ZAMMAD_URL}/api/v1/tickets`,
          {
            title: `Chat #${conversationId} - ${name}`,
            group: "Users",
            customer_id: customerId,
            article: {
              subject: "Nuevo mensaje desde Chatwoot",
              body: message.replace(/<[^>]*>?/gm, ""),
              type: "note",
              internal: false
            }
          },
          {
            headers: {
              Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
              "Content-Type": "application/json"
            }
          }
        );

        ticketId = newTicket.data.id;
        conversationMap[conversationId] = ticketId;

        return res.sendStatus(200);
      }

      // SI EXISTE → AÑADIR MENSAJE
      await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/ticket_articles`,
        {
          ticket_id: ticketId,
          subject: "Nuevo mensaje desde Chatwoot",
          body: message.replace(/<[^>]*>?/gm, ""),
          type: "note",
          internal: false
        },
        {
          headers: {
            Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
            "Content-Type": "application/json"
          }
        }
      );

      return res.sendStatus(200);
    }

    return res.sendStatus(200);

  } catch (error) {
    console.error("Error Chatwoot → Zammad:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

/*
====================================
ZAMMAD → CHATWOOT
====================================
*/
app.post("/zammad", async (req, res) => {
  try {

    const article = req.body?.article;
    const ticket = req.body?.ticket;

    if (!article || article.internal) return res.sendStatus(200);

    const ticketId = ticket?.id;

    const conversationId = Object.keys(conversationMap)
      .find(key => conversationMap[key] === ticketId);

    if (!conversationId) return res.sendStatus(200);

    await axios.post(
      `${process.env.CHATWOOT_URL}/api/v1/accounts/${process.env.ACCOUNT_ID}/conversations/${conversationId}/messages`,
      {
        content: article.body,
        message_type: "outgoing"
      },
      {
        headers: {
          api_access_token: process.env.CHATWOOT_TOKEN,
          "Content-Type": "application/json"
        }
      }
    );

    return res.sendStatus(200);

  } catch (error) {
    console.error("Error Zammad → Chatwoot:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

app.listen(process.env.PORT || 4000, () => {
  console.log(`Middleware corriendo en puerto ${process.env.PORT || 4000}`);
});
```

---

# 5️⃣ CONFIGURAR WEBHOOKS

---

## 🔹 En Chatwoot

```
Settings → Integrations → Webhooks
```

Añadir:

```
http://TU_SERVIDOR:4000/chatwoot
```

Evento:

```
message_created
```

---

## 🔹 En Zammad

```
Admin → Webhooks → New Webhook
```

URL:

```
http://TU_SERVIDOR:4000/zammad
```

Evento:

```
Ticket update
```

---

# 6️⃣ ARRANCAR

```bash
node index.js
```

---

# ✅ RESULTADO FINAL

✔ Mensaje en Chatwoot → crea ticket en Zammad
✔ Respuesta en Zammad → aparece en Chatwoot
✔ Sin errores SSL
✔ Sin Invalid Access Token
✔ Sin 422

---

# ⚠️ LIMITACIÓN ACTUAL

El `conversationMap` está en memoria.
Si reinicias Node, pierde la relación.

---
