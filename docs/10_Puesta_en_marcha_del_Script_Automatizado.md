# 🧩 10. Puesta en marcha del Script Automatizado

1. Creo los directorios (entorno seguro) para la práctica, asigno usuario y permisos.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image.png)

> *Obs.: Es mi primera vez y escribí mal el comando en una de las líneas, no lo tengáis en cuenta.*
> 
1. Creo directorio “documentos” y 3 archivos de texto para simular el contenido al que se le va a realizar el backup (copia de seguridad).

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%201.png)

Ahora que ya tenemos el **entorno seguro** y también el directorio (**documentos**) con el contenido que se va a proteger, vamos a crear primero un **script básico** usando “**nano**”.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%202.png)

1. Asignamos los permisos de ejecución al script.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%203.png)

A continuación vamos a probar manualmente y para ello ejecutamos el script, luego vemos el “log” para comprobar y finalmente listamos la copia creada.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%204.png)

Ahora que ya hemos comprobado que nuestro script funciona, toca la **copia de seguridad automatizada** con todo lo que hemos visto:

> **Recordemos:**
> 
> 
> Vamos a utilizar una máquina virtual con LINUX instalado y mediante scripting en BASH, realizaremos el copiado diario automatizado de un directorio a una carpeta comprimida.
> 

Para ello, ahora tenemos 3 directorios (documentos, datos1 y datos2) dentro de “/home/gabts”, con su contenido de ejemplo al que vamos a realizar su backup automatizado.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%205.png)

1. Tenemos un script `copia_profesional.sh` que hemos visto y que vamos a usar para esta parte de la práctica, **adaptado con los cambios necesarios para su ejecución** (el correo, los directorios, etc.), ahora le vamos a asignar el usuario y el permiso de ejecución:

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%206.png)

1. Usamos **cron**/**crontab** para programar que se ejecute **la copia de seguridad cada día a las 2:00 de la madrugada** (backups nocturnos)

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%207.png)

1. Utilizamos nano para agregar la tarea y nos muestra que se realizó correctamente.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%208.png)

1. Y creamos el archivo oculto GPG (**.gpg_pass**) para el cifrado con nuestra contraseña y asignamos usuario y permiso **600**:

```bash
sudo nano /opt/scripts/backup/.gpg_pass
```

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%209.png)

1. Siempre es una buena práctica, antes de automatizar, comprobar manualmente nuestra copia de seguridad profesional para múltiples carpetas con todo lo que hemos visto:
- **tar**: Permite comprimir archivos y carpetas en un solo archivo.
- **rotación**: Evita la acumulación de backups en el tiempo.
- **verificación**: Realiza una comprobación para evitar errores.
- **cifrado**: Solución para evitar que cualquiera tenga acceso.
- **notificación**: Envía un correo usando msmtp (compatible Gmail).

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%2010.png)

Podemos ver que todo “Ok”, ahora ¡revisemos el correo!

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%2011.png)

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%2012.png)

1. Ahora, verificamos las copias.

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%2013.png)

> **Nota:** Se puede ver las anteriores pruebas que realicé, la del día 03 de abril, luego el 05 y finalmente la prueba del 09 de abril con el cifrado en **gpg** y todo genial.
> 
1. Por último, revisamos el archivo “backup.log” para **visualizar el proceso**, ”**0 errores**”, ¡Ok! 💪

![image.png](%F0%9F%A7%A9%2010%20Puesta%20en%20marcha%20del%20Script%20Automatizado/image%2014.png)

**¡Excelente!**, lo hemos conseguido. 🎉

Tenemos una **copia de seguridad profesional a múltiples carpetas con todas las características necesarias**, automatizada para ejecutarse todos los días a una hora determinada, eliminando aquello que supere los 30 días, verificando su integridad, con cifrado (seguridad) y notificaciones a nuestro correo por si algo no va bien.

Genial ¿no? 😎👍

---

🙌✨