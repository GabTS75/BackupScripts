# 🧩 10. Puesta en marcha del Script Automatizado

1.- Creo los directorios (entorno seguro) para la práctica, asigno usuario y permisos.

![image.png](/docs/10_imgs/image.png)

> *Obs.: Es mi primera vez y escribí mal el comando en una de las líneas, no lo tengáis en cuenta.*

2.- Creo directorio “documentos” y 3 archivos de texto para simular el contenido al que se le va a realizar el backup (copia de seguridad).

![image1.png](/docs/10_imgs/image1.png)

Ahora que ya tenemos el **entorno seguro** y también el directorio (**documentos**) con el contenido que se va a proteger, vamos a crear primero un **script básico** usando “**nano**”.

![image2.png](/docs/10_imgs/image2.png)

3.- Asignamos los permisos de ejecución al script.

![image3.png](/docs/10_imgs/image3.png)

A continuación vamos a probar manualmente y para ello ejecutamos el script, luego vemos el “log” para comprobar y finalmente listamos la copia creada.

![image4.png](/docs/10_imgs/image4.png)

Ahora que ya hemos comprobado que nuestro script funciona, toca la **copia de seguridad automatizada** con todo lo que hemos visto:

> **Recordemos:**
> Vamos a utilizar una máquina virtual con LINUX instalado y mediante scripting en BASH, realizaremos el copiado diario automatizado de un directorio a una carpeta comprimida.

Para ello, ahora tenemos 3 directorios (documentos, datos1 y datos2) dentro de “/home/gabts”, con su contenido de ejemplo al que vamos a realizar su backup automatizado.

![image5.png](/docs/10_imgs/image5.png)

4.- Tenemos un script `copia_profesional.sh` que hemos visto y que vamos a usar para esta parte de la práctica, **adaptado con los cambios necesarios para su ejecución** (el correo, los directorios, etc.), ahora le vamos a asignar el usuario y el permiso de ejecución:

![image6.png](/docs/10_imgs/image6.png)

5.- Usamos **cron**/**crontab** para programar que se ejecute **la copia de seguridad cada día a las 2:00 de la madrugada** (backups nocturnos)

![image7.png](/docs/10_imgs/image7.png)

6.- Utilizamos nano para agregar la tarea y nos muestra que se realizó correctamente.

![image8.png](/docs/10_imgs/image8.png)

7.- Y creamos el archivo oculto GPG (**.gpg_pass**) para el cifrado **con nuestra contraseña** y asignamos usuario y permiso **600**:

```bash
sudo nano /opt/scripts/backup/.gpg_pass
```

![image9.png](/docs/10_imgs/image9.png)

8.- Siempre es una buena práctica, antes de automatizar, comprobar manualmente nuestra copia de seguridad profesional para múltiples carpetas con todo lo que hemos visto:

- **tar**: Permite comprimir archivos y carpetas en un solo archivo.
- **rotación**: Evita la acumulación de backups en el tiempo.
- **verificación**: Realiza una comprobación para evitar errores.
- **cifrado**: Solución para evitar que cualquiera tenga acceso.
- **notificación**: Envía un correo usando msmtp (compatible Gmail).

![image10.png](/docs/10_imgs/image10.png)

Podemos ver que todo “**Ok**”, ahora ¡revisemos el correo!

![image11.png](/docs/10_imgs/image11.png)

![image12.png](/docs/10_imgs/image12.png)

9.- Ahora, verificamos las copias.

![image13.png](/docs/10_imgs/image13.png)

> **Nota:** Se puede ver las anteriores pruebas que realicé, la del día 03 de abril, luego el 05 y finalmente la prueba del 09 de abril con el cifrado en **gpg** y todo genial.

10.- Por último, revisamos el archivo “backup.log” para **visualizar el proceso**, ”**0 errores**”, ¡Ok! 💪

![image14.png](/docs/10_imgs/image14.png)

**¡Excelente!**, lo hemos conseguido. 🎉

Tenemos una **copia de seguridad profesional a múltiples carpetas con todas las características necesarias**, automatizada para ejecutarse todos los días a una hora determinada, eliminando aquello que supere los 30 días, verificando su integridad, con cifrado (seguridad) y notificaciones a nuestro correo por si algo no va bien.

Genial ¿no? 😎👍

---

🙌✨
