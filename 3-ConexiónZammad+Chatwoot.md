
# Integración bidireccional Chatwoot ↔ Zammad con Node.js

## 1. Preparar el middleware Node.js

### 1.1 Instalar Node.js

En el servidor de Zammad:

```bash
sudo apt update
sudo apt install nodejs npm -y
node -v
npm -v
```

---

### 1.2 Crear carpeta del middleware

```bash
sudo mkdir ~/chat-integration
cd ~/chat-integration$
npm init -y
npm install express axios dotenv
```

* Esto instalará **Express**, **Axios** y **dotenv** para manejar variables de entorno.

---

### 1.3 Configurar ES Modules (opcional, recomendado)

Editar `package.json` y añadir:

```json
"type": "module"
```

---

### 1.4 Crear archivo `.env`

```bash
nano .env
```

Contenido:

```
PORT=4000

ZAMMAD_URL=http://localhost:3000
ZAMMAD_TOKEN=

CHATWOOT_URL=http://IP-CHATWOOT:3000
CHATWOOT_TOKEN=
ACCOUNT_ID=1
```

---

### 1.5 Crear `index.js` (middleware)

```bash
nano index.js
```

Contenido:
```
// index.js
import express from "express";
import axios from "axios";
import dotenv from "dotenv";
import fs from "fs";

dotenv.config();

const app = express();
app.use(express.json());

/* =========================================================
   Utilidad de limpieza: sin HTML, sin [zammad]/[chatwoot], sin firma
   ========================================================= */
function cleanBody(text = "") {
  let t = String(text);

  // Decodifica entidades básicas
  t = t.replace(/&lt;/g, "<").replace(/&gt;/g, ">").replace(/&amp;/g, "&");

  // Elimina etiquetas HTML
  t = t.replace(/<[^>]*>/g, "");

  // Elimina prefijos tipo [zammad] o [chatwoot] al inicio
  t = t.replace(/^\s*\[(zammad|chatwoot)\]\s*/i, "");

  // Corta tras separador de firma clásico
  t = t.split("\n-- ")[0];

  // Limpieza de espacios
  t = t.replace(/\s+\n/g, "\n").trim();

  return t;
}

/* =========================================================
   PERSISTENCIA conversationMap  (con JSON en disco)
   ========================================================= */
const MAP_FILE = "./conversationMap.json";
let conversationMap = {};
try {
  if (fs.existsSync(MAP_FILE)) {
    conversationMap = JSON.parse(fs.readFileSync(MAP_FILE, "utf8"));
  }
} catch {
  conversationMap = {};
}
function saveMap() {
  try {
    fs.writeFileSync(MAP_FILE, JSON.stringify(conversationMap, null, 2));
  } catch {
    // noop
  }
}

/* =========================================================
   CHATWOOT → ZAMMAD  (solo mensajes del cliente)
   ========================================================= */
app.post("/chatwoot", async (req, res) => {
  try {
    const event = req.body?.event;
    if (event !== "message_created") return res.sendStatus(200);

    const messageType = req.body?.message_type; // incoming / outgoing
    if (messageType !== "incoming") return res.sendStatus(200); // evita loops

    const conversationId = req.body?.conversation?.id;
    const raw = req.body?.content || "";
    const content = cleanBody(raw);
    if (!conversationId || !content) return res.sendStatus(200);

    const contact = req.body?.sender || {};
    const email = contact?.email;
    const name = contact?.name || "Cliente";

    if (!email) return res.sendStatus(200);

    let ticketId = conversationMap[conversationId];

    // 1) Si no hay ticket asociado, crearlo
    if (!ticketId) {
      // Crear / buscar usuario
      let customerId;
      try {
        const newUser = await axios.post(
          `${process.env.ZAMMAD_URL}/api/v1/users`,
          { firstname: name, lastname: "-", email, role_ids: [3] },
          {
            headers: {
              Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
              "Content-Type": "application/json",
            },
          }
        );
        customerId = newUser.data.id;
      } catch {
        const search = await axios.get(
          `${process.env.ZAMMAD_URL}/api/v1/users/search?query=${encodeURIComponent(email)}`,
          { headers: { Authorization: `Token token=${process.env.ZAMMAD_TOKEN}` } }
        );
        customerId = search.data?.[0]?.id;
      }

      const ticket = await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/tickets`,
        {
          title: `Chat #${conversationId} - ${name}`,
          group: "Users",
          customer_id: customerId,
          article: {
            subject: "Nuevo mensaje desde Chatwoot",
            body: content,
            type: "web",      // artículo de origen web (NO email)
            internal: false,  // público
          },
        },
        {
          headers: {
            Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
            "Content-Type": "application/json",
          },
        }
      );

      ticketId = ticket.data.id;
      conversationMap[conversationId] = ticketId;
      saveMap();

      return res.sendStatus(200);
    }

    // 2) Si ya existe, añadir artículo público
    await axios.post(
      `${process.env.ZAMMAD_URL}/api/v1/ticket_articles`,
      {
        ticket_id: ticketId,
        subject: "Mensaje desde Chatwoot",
        body: content,
        type: "web",     // mantenemos "web" para entradas de chat
        internal: false, // público
      },
      {
        headers: {
          Authorization: `Token token=${process.env.ZAMMAD_TOKEN}`,
          "Content-Type": "application/json",
        },
      }
    );

    return res.sendStatus(200);
  } catch (error) {
    console.error("❌ Error Chatwoot → Zammad:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

/* =========================================================
   ZAMMAD → CHATWOOT  (solo notas públicas de agente)
   ========================================================= */
app.post("/zammad", async (req, res) => {
  try {
    const article = req.body?.article;
    const ticket = req.body?.ticket;
    if (!article || !ticket) return res.sendStatus(200);

    // Filtros robustos (por si el trigger es laxo)
    const isNote = (article.type || "").toLowerCase() === "note";
    const isPublic = article.internal === false;
    const fromAgent =
      (article.sender || "").toLowerCase() === "agent" ||
      (article.sender_name || "").toLowerCase() === "agent";

    if (!(isNote && isPublic && fromAgent)) return res.sendStatus(200);

    const ticketId = ticket.id;

    // Recuperar conversación asociada
    let conversationId = Object.keys(conversationMap).find(
      (key) => conversationMap[key] === ticketId
    );

    // Fallback: si guardaste chatwoot_id como custom field en el ticket
    if (!conversationId) {
      const cf = ticket.custom_fields || ticket.preferences || {};
      if (cf.chatwoot_id) {
        conversationId = cf.chatwoot_id;
        conversationMap[conversationId] = ticketId;
        saveMap();
      }
    }

    if (!conversationId) {
      console.log("⚠ No se encontró conversación asociada a este ticket.");
      return res.sendStatus(200);
    }

    const content = cleanBody(article.body || "");
    if (!content) return res.sendStatus(200);

    await axios.post(
      `${process.env.CHATWOOT_URL}/api/v1/accounts/${process.env.ACCOUNT_ID}/conversations/${conversationId}/messages`,
      {
        content,                 // ← SIN prefijo [zammad]
        message_type: "outgoing",
        private: false,
      },
      {
        headers: {
          api_access_token: process.env.CHATWOOT_TOKEN,
          "Content-Type": "application/json",
        },
      }
    );

    return res.sendStatus(200);
  } catch (error) {
    console.error("❌ Error Zammad → Chatwoot:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

app.listen(process.env.PORT || 4000, () => {
  console.log("✅ Middleware activo en puerto", process.env.PORT || 4000);
});
```

## 2. Ejecutar el middleware

```bash
node index.js
```

Salida esperada:

```
Middleware corriendo en puerto 4000
```
---

## 3. Configurar Nginx para el middleware

Editar `zammad.conf`:

```bash
sudo nano /etc/nginx/sites-available/zammad.conf
```

Añadir dentro del bloque `server { ... }`:

```nginx
location /webhook/chatwoot {
    proxy_pass http://localhost:4000/chatwoot;
}

location /webhook/zammad {
    proxy_pass http://localhost:4000/zammad;
}
```

Probar y reiniciar Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## 4. Configurar Webhooks en Chatwoot

1. Ir a **Settings → Integrations → Webhooks → Create new**.
2. URL:

```
http://TU_SERVIDOR/webhook/chatwoot
```

3. Evento: `message_created`
4. Método: `POST`
5. Guardar.

> Si Chatwoot está en Docker y quieres usar HTTP local, activar:

```bash
ENABLE_INSECURE_WEBHOOKS=true
docker compose restart
```

---

## 5. Configurar Webhooks en Zammad

1. Admin → Webhooks → New Webhook
2. URL:

```
https://polygalaceous-alaysia-unsilently.ngrok-free.dev/webhook/zammad
```

3. Crear Trigger en Zammad:

   * Condición: `Ticket → State → is → open` (para pruebas)
   * Acción: `Webhook → tu webhook`
   * Evento: `Article is created`


W1.png
---


PORT=4000

ZAMMAD_URL=http://localhost:3000
ZAMMAD_TOKEN=JBBk5iyWNnkfJflzuwkdTeRRZwCY6kdREtwpIwmRidz5BLTRE7fnHNt487e9sZcg

CHATWOOT_URL=http://192.168.136.121:3000
CHATWOOT_TOKEN=8eRMbHM3U5tAtg1yW867342F
ACCOUNT_ID=1


## 6. Tokens necesarios

* **Zammad:** Admin → Security → API → Crear Token
* **Chatwoot:** Profile Settings → Access Tokens → Crear token (usar solo API Access Token)
