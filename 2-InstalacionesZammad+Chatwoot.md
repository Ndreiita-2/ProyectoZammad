# Configuración de máquina virtual con Zammad

## Configuración de red

Editar archivo de configuración de red (netplan):

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

---

Configuración de red estática:

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

Aplicar configuración de red:

```bash
sudo netplan apply
```

---

## Actualización del sistema

Actualizar paquetes del sistema y reiniciar:

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

## Instalación de dependencias necesarias

Instalar herramientas básicas necesarias para repositorios y descargas:

```bash
sudo apt install curl apt-transport-https gnupg
```

---

## Instalación de Elasticsearch

Crear directorio para almacenar claves GPG:

```bash
sudo mkdir -p /etc/apt/keyrings
```

Descargar y guardar la clave GPG oficial de Elasticsearch:

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/keyrings/elastic.gpg
```

Añadir repositorio oficial de Elasticsearch:

```bash
echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```

Actualizar lista de paquetes:

```bash
sudo apt update
```

Instalar Elasticsearch:

```bash
sudo apt install elasticsearch -y
```

Habilitar Elasticsearch para que arranque con el sistema:

```bash
sudo systemctl enable elasticsearch
```

Iniciar servicio de Elasticsearch:

```bash
sudo systemctl start elasticsearch
```

Verificar estado del servicio:

```bash
sudo systemctl status elasticsearch
```

---

## Configuración de localización del sistema

Verificar configuración actual del idioma:

```bash
locale | grep "LANG="
```

Instalar paquete de localización si es necesario:

```bash
sudo apt install locales
```

Generar localización en inglés UTF-8:

```bash
sudo locale-gen en_US.UTF-8
```

Definir idioma por defecto del sistema:

```bash
echo "LANG=en_US.UTF-8" > sudo /etc/default/locale
```

Verificar configuración nuevamente o reiniciar si es necesario.

---

## Añadir repositorio de Zammad

Descargar y guardar clave GPG de Zammad:

```bash
curl -fsSL https://dl.packager.io/srv/zammad/zammad/key | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/pkgr-zammad.gpg > /dev/null \
   && sudo chmod 644 /etc/apt/keyrings/pkgr-zammad.gpg
```

Añadir repositorio oficial de Zammad:

```bash
printf "Types: deb
URIs: https://dl.packager.io/srv/deb/zammad/zammad/stable/ubuntu
Suites: 24.04
Components: main
Signed-By: /etc/apt/keyrings/pkgr-zammad.gpg" | \
sudo tee /etc/apt/sources.list.d/zammad.sources > /dev/null
```

---

## Instalación de Zammad

Actualizar repositorios:

```bash
sudo apt update
```

Instalar Zammad:

```bash
sudo apt install zammad
```

---

# Creación manual de base de datos y usuario

## Acceder a PostgreSQL como usuario administrador

```bash
sudo -i -u postgres psql
```

---

## Crear usuario para Zammad

Crear usuario de base de datos con contraseña:

```sql
CREATE USER <TU_USUARIO> WITH PASSWORD '<TU_CONTRASEÑA>';
```

---

## Crear la base de datos

Crear base de datos y asignar propietario:

```sql
CREATE DATABASE <TU_BD> OWNER <TU_USUARIO>;
```

---

## Asignar privilegios al usuario

Otorgar control total sobre la base de datos:

```sql
GRANT ALL PRIVILEGES ON DATABASE <TU_BD> TO <TU_USUARIO>;
```

---

## Salir de PostgreSQL

```sql
\q
```

---

## Configuración de conexión a base de datos en Zammad

Editar archivo de configuración:

```bash
sudo nano /opt/zammad/config/database.yml
```

Definir conexión a base de datos:

```yaml
production:
  adapter: postgresql
  database: <TU_BD>
  username: <TU_USUARIO>
  password: '<TU_CONTRASEÑA>'
  host: localhost
  encoding: unicode
```

---

## Inicialización de la base de datos

Aplicar estructura de base de datos:

```bash
sudo zammad run rake db:migrate
```

Cargar datos iniciales del sistema:

```bash
sudo zammad run rake db:seed
```

---

## Reinicio del servicio de Zammad

Reiniciar servicio para aplicar cambios:

```bash
sudo systemctl restart zammad
```

Verificar estado del servicio:

```bash
sudo systemctl status zammad
```

---

## Configuración de Nginx para Zammad

Editar archivo de configuración web:

```bash
sudo nano /etc/nginx/sites-available/zammad.conf
```

Definir dominio o IP del servidor:

```bash
server {
    server_name TU_DOMINIO_O_IP;
}
```

---

## Acceso local a Zammad

Acceder desde navegador:

```
http://192.168.136.X
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Configuración de máquina virtual con Chatwoot (DOCKER)

## CONFIGURACIÓN DE RED

Editar archivo de configuración de red (netplan):

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

---

Configurar red estática:

```yaml
      addresses:
        - 192.168.136.Y/24
      routes:
        - to: default
          via: 192.168.136.1
      nameservers:
        addresses: [8.8.8.8,1.1.1.1]
```

Aplicar configuración de red:

```bash
sudo netplan apply
```

---

## Instalación de Docker

Actualizar sistema e instalar dependencias necesarias:

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

Crear directorio para almacenar claves GPG:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Descargar y guardar clave GPG oficial de Docker:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Asignar permisos de lectura a la clave:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Añadir repositorio oficial de Docker:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Actualizar lista de paquetes:

```bash
sudo apt update
```

Instalar Docker y componentes necesarios:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Habilitar Docker para que arranque con el sistema:

```bash
sudo systemctl enable docker
```

Iniciar servicio Docker:

```bash
sudo systemctl start docker
```

Añadir usuario actual al grupo docker (para evitar usar sudo):

```bash
sudo usermod -aG docker $USER
```

Cerrar sesión y volver a entrar para aplicar permisos.

---

## Crear proyecto Chatwoot

Crear directorio del proyecto:

```bash
sudo mkdir -p /srv/chatwoot
```

Acceder al directorio:

```bash
cd /srv/chatwoot
```

Crear archivo de configuración Docker:

```bash
nano docker-compose.yml
```

---

## docker-compose.yml

Definir servicios necesarios (PostgreSQL, Redis y Chatwoot):

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
    image: chatwoot/chatwoot:v4.12.1
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

---

## Generar clave secreta

Generar clave para `SECRET_KEY_BASE`:

```bash
openssl rand -hex 64
```

Sustituir el valor `CAMBIAR_POR_CLAVE` en el archivo docker-compose.yml.

---

## Levantar servicios

Iniciar contenedores en segundo plano:

```bash
sudo docker compose up -d
```

---

## Verificación de contenedores

Comprobar que los servicios están activos:

```bash
sudo docker ps
```

Ver logs en caso de error:

```bash
docker compose logs -f
```

---

## ACCESO LOCAL CHATWOOT

Acceder desde navegador:

```
http://192.168.136.Y:3000
```

## Archivo PID bloqueado después de reiniciar, solución:
```
docker compose down
```
```
docker compose up -d --force-recreate
```

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Perfecto. Aquí tienes la **guía finalizada y completa**, incorporando todo lo que hicimos para exponer Zammad a Internet usando **ngrok en el puerto 80**. Está lista para seguir paso a paso desde la instalación de ngrok hasta la configuración final de Nginx.

---

# Guía para exponer Zammad a Internet con ngrok (puerto 80)

## 1. Instalar y actualizar ngrok

1. Eliminar versión antigua de ngrok:

```bash
sudo rm /usr/local/bin/ngrok
```

2. Descargar la versión más reciente (Linux AMD64):

```bash
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
```

3. Descomprimir y mover el binario:

```bash
tar -xvzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/
```

4. Verificar versión:

```bash
ngrok version
```

Debe mostrar algo como `ngrok version 3.x.x`.

5. Autenticar ngrok con tu token (obtenido en [dashboard.ngrok.com](https://dashboard.ngrok.com/get-started/your-authtoken)):

```bash
ngrok config add-authtoken <TU_AUTH_TOKEN>
```

---

## 2. Configurar Nginx para trabajar con ngrok

1. Editar el archivo de configuración de Zammad:

```bash
sudo nano /etc/nginx/sites-available/zammad.conf
```

2. Modificar los siguientes valores:

* Cambiar `server_name` a `localhost`:

```nginx
server_name localhost;
```
* Mantener los `upstream` y proxys tal como están (Rails server y websockets).

3. Habilitar la configuración (si aún no lo estaba):

```bash
sudo ln -s /etc/nginx/sites-available/zammad.conf /etc/nginx/sites-enabled/zammad.conf
```

* Si el enlace ya existe, no hace falta volver a crearlo.

4. Deshabilitar el sitio por defecto de Nginx para evitar la página “Welcome”:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

5. Probar configuración y reiniciar Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

4. Reiniciar Zammad:

```bash
sudo systemctl restart zammad
```

---

## 4. Levantar túnel de ngrok

1. Ejecutar ngrok apuntando al puerto 80 (Nginx):

```bash
ngrok http 80
```

2. Copiar el enlace HTTPS que ngrok genera, por ejemplo:

```
https://polygalaceous-alaysia-unsilently.ngrok-free.dev/
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
