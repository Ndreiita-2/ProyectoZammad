# Zammad
## CONFIGURACIÓN DE RED

Archivo netplan:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

---

```yaml
      addresses:
        - 192.168.136.X/24
      routes:
        - to: default
          via: 192.168.136.1
      nameservers:
        addresses: [8.8.8.8,1.1.1.1]
```

---

Aplicar:

```bash
sudo netplan apply
```

---

## Actualizar

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```
## ¿?

1. Instalar las herramientas necesarias

```bash
sudo apt install curl apt-transport-https gnupg
```

2. Instalar Elasticsearch

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/keyrings/elastic.gpg
```


```bash
echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```


```bash
sudo apt update
```

---

```bash
sudo apt install elasticsearch -y
```

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch
```



3. Asegurar la ubicación correcta
   
Enumera tus ajustes actuales de localización:

```bash
locale | grep "LANG="
```

Si lo anterior no regresa, puedes corregir este problema de la siguiente manera:

```bash
sudo apt install locales
```
```bash
sudo locale-gen en_US.UTF-8
```
```bash
echo "LANG=en_US.UTF-8" > sudo /etc/default/locale
```

Después de arreglarlo, asegúrate de revisar de nuevo la salida para incluir . Un reinicio puede ayudar si no tiene éxito.

Añadir repositorio

```bash
curl -fsSL https://dl.packager.io/srv/zammad/zammad/key | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/pkgr-zammad.gpg > /dev/null \
   && sudo chmod 644 /etc/apt/keyrings/pkgr-zammad.gpg
```
```bash
printf "Types: deb
URIs: https://dl.packager.io/srv/deb/zammad/zammad/stable/ubuntu
Suites: 24.04
Components: main
Signed-By: /etc/apt/keyrings/pkgr-zammad.gpg" | \
sudo tee /etc/apt/sources.list.d/zammad.sources > /dev/null
```

Instalar Zammad

```bash
sudo apt update
```
```bash
sudo apt install zammad
```
---

¿?¿?¿?¿?¿??¿¿?¿?¿?¿?¿?¿??¿?¿?¿?¿?¿?¿?¿?¿?¿?¿?¿?¿

## Instalar dependencias + Nginx

```bash
sudo apt install curl gnupg apt-transport-https ca-certificates lsb-release nginx postgresql redis-server -y
```

---

## Instalar Elasticsearch

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update
sudo apt install elasticsearch -y
```

Editar:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Asegurar:

```
network.host: localhost
```

Activar:

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

---

## Instalar Zammad

```bash
curl -fsSL https://dl.packager.io/srv/zammad/zammad/key | sudo gpg --dearmor -o /usr/share/keyrings/zammad.gpg

echo "deb [signed-by=/usr/share/keyrings/zammad.gpg] https://dl.packager.io/srv/deb/zammad/zammad/stable/ubuntu 22.04 main" | sudo tee /etc/apt/sources.list.d/zammad.list

sudo apt update
sudo apt install zammad -y
```

Perfecto, aquí tienes la **Opción 1** lista para copiar y pegar en tu guía, con los campos de usuario y contraseña en blanco para que los completes tú:

---

## 🔹 Opción 1 – Crear base de datos y usuario manualmente (Zammad 7)

### 1️⃣ Acceder a PostgreSQL como `postgres`

```bash
sudo -i -u postgres psql
```

---

### 2️⃣ Crear usuario para Zammad

```sql
CREATE USER <TU_USUARIO> WITH PASSWORD '<TU_CONTRASEÑA>';
```

> 🔹 Reemplaza `<TU_USUARIO>` y `<TU_CONTRASEÑA>` por los que quieras usar para Zammad.

---

### 3️⃣ Crear la base de datos y asignarla al usuario

```sql
CREATE DATABASE <TU_BD> OWNER <TU_USUARIO>;
```

> 🔹 `<TU_BD>` puede ser por ejemplo `zammad` o cualquier nombre que prefieras.
> 🔹 `<TU_USUARIO>` debe coincidir con el usuario que creaste en el paso anterior.

---

### 4️⃣ Dar privilegios completos al usuario sobre la base de datos

```sql
GRANT ALL PRIVILEGES ON DATABASE <TU_BD> TO <TU_USUARIO>;
```

---

### 5️⃣ Salir de PostgreSQL

```sql
\q
```

---

### 6️⃣ Configurar Zammad para usar este usuario y base de datos

Editar el archivo `/opt/zammad/config/database.yml` (o equivalente según tu versión):

```yaml
production:
  adapter: postgresql
  database: <TU_BD>
  username: <TU_USUARIO>
  password: '<TU_CONTRASEÑA>'
  host: localhost
  encoding: unicode
```

> 🔹 Asegúrate de reemplazar los campos `<TU_BD>`, `<TU_USUARIO>` y `<TU_CONTRASEÑA>` con los valores que hayas elegido.

---

### 7️⃣ Inicializar la base de datos

```bash
sudo zammad run rake db:migrate
sudo zammad run rake db:seed
```

> Esto aplica todas las migraciones y carga los datos iniciales.

---

### 8️⃣ Reiniciar el servicio de Zammad

```bash
sudo systemctl restart zammad
sudo systemctl status zammad
```


Reiniciar:

```bash
sudo systemctl restart zammad
```

Crear archivo de configuración para Zammad

```bash
sudo nano /etc/nginx/sites-available/zammad.conf
```

```bash
server {
    server_name TU_DOMINIO_O_IP;
}
```
---

## ACCESO LOCAL ZAMMAD

```
http://192.168.136.X
```

---


# INSTALACIÓN CHATWOOT (DOCKER)

## CONFIGURACIÓN DE RED

Archivo netplan:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```
---
```yaml
      addresses:
        - 192.168.136.Y/24
      routes:
        - to: default
          via: 192.168.136.1
      nameservers:
        addresses: [8.8.8.8,1.1.1.1]
```

Aplicar:

```bash
sudo netplan apply
```

## Instalar Docker

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Cerrar sesión y volver a entrar.

---

## Crear proyecto

```bash
sudo mkdir -p /srv/chatwoot
cd /srv/chatwoot
nano docker-compose.yml
```

---

## docker-compose.yml

```yaml
services:

  postgres:
    image: pgvector/pgvector:pg15
    restart: unless-stopped
    environment:
      POSTGRES_USER: chatwoot
      POSTGRES_PASSWORD: ChatwootDB2026!
      POSTGRES_DB: chatwoot
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    restart: unless-stopped

  chatwoot:
    image: chatwoot/chatwoot:latest
  ¿ image: chatwoot/chatwoot:v4.11.0
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
    command: >
      sh -c "bundle exec rails db:chatwoot_prepare &&
             bundle exec rails s -p 3000 -b 0.0.0.0"
    environment:
      RAILS_ENV: production
      SECRET_KEY_BASE: "CAMBIAR_POR_CLAVE"
      FRONTEND_URL: "http://192.168.136.Y:3000"
      REDIS_URL: redis://redis:6379
      POSTGRES_HOST: postgres
      POSTGRES_USERNAME: chatwoot
      POSTGRES_PASSWORD: ChatwootDB2026!
      POSTGRES_DATABASE: chatwoot
    ports:
      - "3000:3000"

volumes:
  postgres_data:
```

Generar clave:

```bash
openssl rand -hex 64
```

Levantar:

```bash
docker compose up -d
```

---

## ACCESO LOCAL CHATWOOT

```
http://192.168.136.Y:3000
```

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Exponer con Ngrok

Proyecto: **ngrok**

Instalar:

```bash
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update
sudo apt install ngrok -y
```

Configurar token:

```bash
ngrok config add-authtoken TU_TOKEN_AQUI
```

Exponer:

```bash
zm@zammad:~$ ngrok http 80

zm@zammad:~/chat-integration$ node index.js
```

Acceso remoto:

```
https://Z.ngrok-free.dev/
```


# TOKEN INVALID AL REINICIAR

## Cambiar la Base URL manualmente

En tu servidor ejecuta:
```
sudo zammad run rails c
```
Ahora pega esto:
```
Setting.set('fqdn', 'localhost')
```
y luego:
```
Setting.set('http_type', 'http')
```
Esto hace que Zammad deje de esperar la URL de ngrok.

Sal de la consola:

exit
## Reiniciar Zammad
```
sudo systemctl restart zammad
```
Espera unos segundos.



# CAMBIAR CONTRASEÑA DEL USUARIO ADMIN EN CHATWOOT

## Ir al directorio

```bash
cd /srv/chatwoot
```

---

## Entrar a la consola Rails

```bash
docker compose exec chatwoot bundle exec rails c
```

---

## Verificar usuario existente

```ruby
User.pluck(:email)
```

Debe mostrar:

```
["andrea.chamorro@creatorsco.com"]
```

---

## Cambiar contraseña

```ruby
user = User.find_by(email: "andrea.chamorro@creatorsco.com")
user.password = "Pruebas1?"
user.password_confirmation = "Pruebas1?"
user.save!
```

---

## Ahora puedes entrar con:

Email:

```
andrea.chamorro@creatorsco.com
```

Contraseña:

```
Pruebas1?
```

# Archivo PID bloqueado después de reiniciar, solución:
```
docker compose down
```
```
docker compose up -d --force-recreate
```
---
