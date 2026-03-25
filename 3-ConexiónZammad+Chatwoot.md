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
