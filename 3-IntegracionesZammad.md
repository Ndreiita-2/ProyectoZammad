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
