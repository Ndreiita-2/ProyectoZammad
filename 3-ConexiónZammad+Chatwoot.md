Perfecto. He revisado las dos guías que me pasaste y he unificado todo en **una guía corregida y coherente**, compatible con **Zammad 7** y **Chatwoot 4.12.1**, usando un **middleware Node.js**. Incluye la instalación, configuración, Nginx, y pasos de prueba.

---

# Guía final corregida: Integración bidireccional Chatwoot ↔ Zammad con Node.js

---

## 1. Preparar el middleware Node.js

### 1.1 Instalar Node.js

En el servidor donde estará el middleware (puede ser la misma máquina que Chatwoot o Zammad):

```bash
sudo apt update
sudo apt install nodejs npm -y
node -v
npm -v
```

---

### 1.2 Crear carpeta del middleware

```bash
sudo mkdir -p /opt/chatwoot_zammad_middleware
cd /opt/chatwoot_zammad_middleware
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

ZAMMAD_URL=http://IP_O_DOMINIO_ZAMMAD
ZAMMAD_TOKEN=TU_TOKEN_ZAMMAD

CHATWOOT_URL=http://IP_O_DOMINIO_CHATWOOT:3000
CHATWOOT_TOKEN=TU_TOKEN_CHATWOOT
ACCOUNT_ID=1
```

> Notas:
>
> * No incluir rutas como `/app` o `/api`.
> * En producción usar dominio con HTTPS.

---

### 1.5 Crear `index.js` (middleware)

```bash
nano index.js
```

Contenido (resumido y compatible):

```js
import express from "express";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

// Persistencia temporal de conversaciones
const conversationMap = {};

// Chatwoot → Zammad
app.post("/chatwoot", async (req, res) => {
  try {
    const event = req.body?.event;
    if (event !== "message_created") return res.sendStatus(200);

    const messageType = req.body?.message_type;
    if (messageType !== "incoming") return res.sendStatus(200);

    const conversationId = req.body?.conversation?.id;
    const message = req.body?.content || "";
    const contact = req.body?.sender || {};

    if (!conversationId || !contact?.email) return res.sendStatus(200);

    let ticketId = conversationMap[conversationId];

    if (!ticketId) {
      // Crear usuario si no existe
      let customerId;
      const newUser = await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/users`,
        {
          firstname: contact?.name || "Sin nombre",
          lastname: "-",
          email: contact.email,
          role_ids: [3]
        },
        {
          headers: { Authorization: `Token token=${process.env.ZAMMAD_TOKEN}` }
        }
      ).catch(() => null);

      if (newUser) customerId = newUser.data.id;
      else {
        const userSearch = await axios.get(
          `${process.env.ZAMMAD_URL}/api/v1/users/search?query=${contact.email}`,
          { headers: { Authorization: `Token token=${process.env.ZAMMAD_TOKEN}` } }
        );
        customerId = userSearch.data[0]?.id;
      }

      const newTicket = await axios.post(
        `${process.env.ZAMMAD_URL}/api/v1/tickets`,
        {
          title: `Chat #${conversationId} - ${contact?.name || "Cliente"}`,
          group: "Users",
          customer_id: customerId,
          article: { subject: "Nuevo mensaje desde Chatwoot", body: message.replace(/<[^>]*>?/gm, ""), type: "note", internal: false }
        },
        { headers: { Authorization: `Token token=${process.env.ZAMMAD_TOKEN}` } }
      );

      ticketId = newTicket.data.id;
      conversationMap[conversationId] = ticketId;
      return res.sendStatus(200);
    }

    // Agregar artículo a ticket existente
    await axios.post(
      `${process.env.ZAMMAD_URL}/api/v1/ticket_articles`,
      { ticket_id: ticketId, subject: "Nuevo mensaje desde Chatwoot", body: message.replace(/<[^>]*>?/gm, ""), type: "note", internal: false },
      { headers: { Authorization: `Token token=${process.env.ZAMMAD_TOKEN}` } }
    );

    return res.sendStatus(200);

  } catch (error) {
    console.error("Error Chatwoot → Zammad:", error.response?.data || error.message);
    return res.sendStatus(500);
  }
});

// Zammad → Chatwoot
app.post("/zammad", async (req, res) => {
  try {
    const article = req.body?.article;
    const ticket = req.body?.ticket;

    if (!article || article.internal) return res.sendStatus(200);

    const ticketId = ticket?.id;
    const conversationId = Object.keys(conversationMap).find(k => conversationMap[k] === ticketId);
    if (!conversationId) return res.sendStatus(200);

    await axios.post(
      `${process.env.CHATWOOT_URL}/api/v1/accounts/${process.env.ACCOUNT_ID}/conversations/${conversationId}/messages`,
      { content: article.body, message_type: "outgoing" },
      { headers: { api_access_token: process.env.CHATWOOT_TOKEN } }
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

## 2. Ejecutar el middleware

```bash
node index.js
```

Salida esperada:

```
Middleware corriendo en puerto 4000
```

* Para producción:

```bash
sudo npm install -g pm2
pm2 start index.js --name chatwoot-zammad
pm2 save
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
http://TU_SERVIDOR/webhook/zammad
```

3. Crear Trigger en Zammad:

   * Condición: `Ticket → State → is → open` (para pruebas)
   * Acción: `Webhook → tu webhook`
   * Evento: `Article is created`

---

## 6. Tokens necesarios

* **Zammad:** Admin → Security → API → Crear Token
* **Chatwoot:** Profile Settings → Access Tokens → Crear token (usar solo API Access Token)
