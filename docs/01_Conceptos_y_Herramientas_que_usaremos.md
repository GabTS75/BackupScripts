# 📝 1. Conceptos y Herramientas que usaremos

## **¿Qué aprendemos con esta guía?**

- ✅ ¿Qué son las copias de seguridad y por qué son indispensables?
- ✅ ¿Cómo funcionan los scripts Bash y para qué los usamos?
- ✅ ¿Cómo comprimir y gestionar archivos con tar y rsync?
- ✅ ¿Cómo programar tareas automáticas con cron?
- ✅ ¿Cómo proteger tus copias con permisos?
- ✅ Buenas prácticas reales de un administrador de sistemas.

## **¿Qué es una copia de seguridad y por qué importa?**

**El problema:** Todos sabemos que la información (los datos) son mucho más frágiles de lo que parecen. Un disco dañado, un comando mal escrito, un virus o cualquier error humano, todo puede afectar a la perdida de datos importantes y/o sensibles.

**Tipos de copias de seguridad:** Primero debemos tener en cuenta qué queremos copiar y cómo. Existen 3 tipos de copias principales:

- **Completa**: Copia todo el contenido del origen, sin excepciones. Ocupará más espacio y es ideal para copias semanales o mensuales.
- **Incremental**: Solo copia los archivos que se han modificado/cambiado desde la última copia (completa o incremental). Ocupará menos y para restaurar se requiere la copia completa + todas las incrementales.
- **Diferencial**: Copia los archivos que se han modificado/cambiado desde la última copia “completa”. Esta ocupará más espacio que la copia incremental pero es más fácil y rápido de restaurar, es decir, solo se necesitará la copia completa + la última diferencial)

**La regla 3-2-1:** Los profesionales siempre siguen esta regla, así que conviene memorizarla.

- **3**: Guarda al menos 3 copias de los datos (original + 2 copias)
- **2**: Usa al menos 2 tipos de soporte diferentes (por ejemplo, disco interno y otro externo)
- **1**: Guarda 1 copia al menos “fuera de la ubicación física” (en la nube, otro lugar, etc)

## **Las herramientas que usaremos**

Previamente, debemos conocer las herramientas que tenemos disponibles para ello.

`Bash` (**B**ourne **A**gain **Sh**ell) que para entenderlo mejor diremos que es el intérprete de comandos de Linux, es decir, un script Bash no es más que un texto con comandos que se ejecutan en orden (primero, segundo, tercero, etc.), como si lo hicieras tú mismo, pero sin estar delante de la consola/terminal.

`tar` El comando “**tar**” (**T**ape **AR**chive) permite empaquetar muchos archivos y carpetas en un solo archivo y a su vez comprimirlos para que ocupen menos espacio.

Algunas de las opciones más importantes de “**tar**” son:

- **-c** : Crear un nuevo archivo tar (Create)
- **-x** : Extraer el contenido (eXtract)
- **-z** : Comprimir/descomprimir con gzip (archivo .tar.gz)
- **-j** : Comprimir/descomprimir con bzip2 (archivo .tar.bz2, más compresión pero más lento)
- **-v** : Modo detallado, muestra lo que está haciendo (Verbose)
- **-f** : Especifica el nombre del archivo (File) — siempre va seguido del nombre
- **-p** : Preserva los permisos originales de los archivos (Permissions)

Ejemplo de uso:

```bash
# Crear una copia de /home/usuario/documentos en formato .tar.gz
tar -czf copia_documentos.tar.gz /home/usuario/documentos

# Extraer esa copia en la carpeta actual
tar -xzf copia_documentos.tar.gz
```

> *Se crea una copia comprimida de la carpeta documentos y luego se extrae.*
> 

`rsync` Es un sincronizador inteligente, es decir, “**tar**” hace copias completas y “**rsync**” que es más “listo”, solo copia los archivos que han cambiado/modificado desde la última sincronización. Resultando ser más rápido y eficiente en espacio.

Opciones principales:

- **-a** : Modo archivo: preserva permisos, fechas, enlaces simbólicos y copia recursivamente
- **-v** : Modo detallado (verbose)
- **-z** : Comprime durante la transferencia
- **--delete** : Elimina en el destino los archivos que ya no existen en el origen
- **--progress** : Muestra el progreso en tiempo real

Ejemplo de uso:

```bash
# Sincronizar /home/usuario/documentos con /backup/documentos
rsync -avz /home/usuario/documentos/ /backup/documentos/

# Copiar a un servidor remoto por SSH
rsync -avz /home/usuario/docs/ usuario@servidor:/backup/docs/
```

> *Sincronización de dos carpetas y copia a un servidor remoto*
> 

`cron` Este es un servicio de Linux que se encarga de ejecutar comandos o scripts de forma automática en momentos específicos. Funciona en segundo plano todo el tiempo.

Las instrucciones para “cron” se guardan en un archivo llamado “**crontab**” (es una abreviatura de cron table, **tabla de cron**). Cada línea define una tarea y cuándo ejecutarla, ejemplo:

```bash
# Formato de una línea crontab:
# minuto   hora   día_del_mes   mes    día_semana    comando
# (0-59)  (0-23)    (1-31)     (1-12)     (0-7)     /ruta/al/script.sh

# Ejemplos:
0 2 * * *  /scripts/backup.sh       # Cada día a las 02:00
0 0 * * 0  /scripts/backup_sem.sh   # Cada domingo a medianoche
*/30 * * * * /scripts/comprueba.sh  # Cada 30 minutos
```

## **Seguridad y permisos del script**

Antes que nada, debemos entender un principio fundamental de la administración de sistemas: **la seguridad no es un añadido, es la base**.

Un script de backup con malos permisos puede ser una puerta abierta para los atacantes.

> **RECORDEMOS, los permisos en Linux (rwx):**
> 
> 
> En Linux, cada archivo tiene tres tipos de permisos: `r` (lectura, read), `w` (escritura, write) y `x` (ejecución, execute). Estos se aplican para tres niveles: el propietario (`u`), el grupo (`g`) y el resto del mundo (`o`).
> 

`chmod` ****permite cambiar los permisos de un archivo y se puede usar con notación numérica u octal, ejemplo:

| **Número** | **Significado** |
| --- | --- |
| 7 | Lectura + Escritura + Ejecución (rwx): **Permiso total** |
| 6 | Lectura + Escritura (rw-): **Sin ejecución** |
| 5 | Lectura + Ejecución (r-x): **Sin escritura** |
| 4 | Solo Lectura (r--): **El más restrictivo útil** |
| 0 | Sin ningún permiso (---): **Completamente bloqueado** |

---