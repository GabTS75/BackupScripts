# 📨 07. Notificaciones por correo (para saber qué está pasando)

Si una copia de seguridad automática falla en silencio, es casi peor que no tener un backup, es decir, crees que tienes protección, pero “no la tienes”, entonces, las notificaciones son las que resuelven este problema.

## Notificación por correo electrónico

En servidores Linux tenemos a `msmtp`, este servicio es un **cliente SMTP minimalista** y **no tendrá problemas con Gmail** si lo usáis como Yo para esta práctica (lo que si pasará si usamos `mailutils` que lo probé y nada). Vamos a explicarlo por partes y detenidamente.

Al utilizar `mailutils`, se genera un archivo automáticamente (**dead.letter**) cuando un mensaje no pudo ser enviado o entregado, almacenando el contenido del correo fallido, como podemos ver:

![image.png](/docs/07_imgs/image.png)

`mailutils`, *es un servicio que permite enviar correos electrónicos desde el terminal en Linux*, pero tiene un problema con **Gmail**, puesto que Gmail exige que **quién envíe un correo** debe autenticarse con su servidor **SMTP** (smtp.gmail.com, puerto 587) usando **TLS** y una **contraseña de aplicación** (que es una diferente a tu contraseña normal) y `mailutils` en su configuración, **no sabe hacerlo**.

Entonces, mejor instalaremos `msmtp`:

```bash
sudo apt update
# Se recomienda actualizar primero, tener paciencia, puede tomar tiempo.

systemctl status unattended-upgrades
# Con esta línea de comando, puedes revisar “el estado” de esas actualizaciones
# antes de intentar instalar algo.

sudo apt install msmtp msmtp-mta -y
# msmtp-mta instalará un alias que hace que el comando mail del sistema 
# llame a msmtp por debajo.
```

## Contraseña de aplicación

Crearemos una contraseña de aplicación (**App Password**) porque Gmail no permite usar tu contraseña habitual desde scripts. Sigue los siguientes pasos en tu cuenta de Gmail:

1. En tu cuenta Google, ve a **Seguridad**.
2. Activa la **verificación en dos pasos** si no la tienes (es obligatorio para este paso).
3. Busca "**Contraseñas de aplicaciones**".
4. Crea una nueva, escribe como nombre de la aplicación "Correo-Linux" (por ejemplo), y finalmente **copia los 16 caracteres** que te da en el pop-up.

Esa es la contraseña que usarás en el archivo de configuración, **nunca tu contraseña real de Gmail**.

![image1.png](/docs/07_imgs/image1.png)

> **IMPOTANTE:** En el caso de que el servidor (Ubuntu Server) alguna vez se ve comprometido, solo tienes que borrar esa contraseña desde tu cuenta de **Gmail** y el atacante **no tendrá acceso** a tus correos ni a tu cuenta principal.

## Creamos el archivo de configuración de `msmtp`

Con esta configuración logramos que `msmtp` se pueda conectar con Gmail. **Debe estar en el directorio** `/home/tu_usuario` **del usuario que ejecuta el script** (o en `/etc/msmtprc` si es root):

```bash
# Ejecutamos el editor “nano”
sudo nano ~/.msmtprc
```

> **NOTA:** Cuando escribimos en la línea `sudo nano ~/.msmtprc`, viene a ser la abreviación de `sudo nano /home/tu_usuario/.msmtprc` (abrir/editar el archivo oculto **.msmtprc**), es decir, la virguilla (**`~`**) representa o equivale a la ruta absoluta de tu carpeta personal(`/home/tu_usuario`). Un ejemplo simple de uso sería: `ls ~/documentos` para listar el contenido de tu carpeta documentos.

Abierto el editor “nano”, escribiremos lo siguiente (reemplaza los valores en mayúsculas):

```bash
# Cuenta de correo para backups
account gmail
host [smtp.gmail.com](http://smtp.gmail.com/)
port 587
from [TU_CORREO@gmail.com](mailto:TU_CORREO@gmail.com)
auth on
user [TU_CORREO@gmail.com](mailto:TU_CORREO@gmail.com)
password TU_APP_PASSWORD_DE_16_CARACTERES
tls on
tls_starttls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile ~/.msmtp.log

# Esta línea indica cuál es la cuenta por defecto
account default : Gmail
```

![image2.png](/docs/07_imgs/image2.png)

Ahora asignaremos los permisos estrictos al archivo. Si no lo hacemos, **msmtp** se negará a usarlo por seguridad:

```bash
# Con 600, solo el dueño puede leer el archivo
chmod 600 ~/.msmtprc
```

![image3.png](/docs/07_imgs/image3.png)

> **Recordemos un poco sobre los permisos:**
> Darle **600** es “`-rw-------`“ para el archivo, es decir, el propietario solo puede **leer y escribir** (modificar), es el nivel más alto de privacidad.
> `-`: Archivo regular (**no directorio**).
> `rw-`: El propietario puede **leer y escribir**.
> `---`: El grupo **no puede** hacer nada.
> `---`: Los otros **no pueden** hacer nada.

## Prueba de envío antes de agregarlo al script

Veremos que, si todo está bien, **llegará el correo sin inconvenientes**, si no, revisaremos **~/.msmtp.log** para ver el error exacto y proceder a corregir el archivo **~/.msmtprc** con “**nano**”.

Ejecutamos:

```bash
echo "Correo de prueba desde Linux" | msmtp -a gmail TU_CORREO@gmail.com
```

![image4.png](/docs/07_imgs/image4.png)

![image5.png](/docs/07_imgs/image5.png)

![image6.png](/docs/07_imgs/image6.png)

> El envío se realizó exitosamente. ¡todo genial! 🎉

## Verificación de integridad

Es posible que un archivo **.tar.gz** pueda crearse sin errores pero estar **internamente corrompido o corrupto**. Para verificar la integridad antes de dar el backup por bueno, podríamos hacer:

```bash
# Probar la integridad del archivo tar.gz (no extrae, solo verifica)
tar -tzf "$ARCHIVO" > /dev/null 2>&1

if [ $? -eq 0 ]; then
		escribir_log "✓ Integridad del archivo verificada correctamente."
else
		escribir_log "✗ ERROR: El archivo de backup parece estar corrupto."
		# Aquí enviaríamos notificación de error urgente
fi
```

---
