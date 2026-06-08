# ☝️ 03. Primer Script de Copia de Seguridad

Ahora que ya tenemos la base, vamos a crear nuestro primer script de copia de seguridad. Iremos paso a paso, línea por línea y explicando qué hace cada parte. ¡Vamos!

## Estructura del script Bash

Todo script **Bash** comienza con una línea especial llamada **shebang** (`#!/bin/bash`). Esta le indica al sistema operativo el programa que debe usar para interpretar el resto del archivo.

## Creando un script

Abre un editor de texto (tu favorito, nano, vim o gedit) y crea el archivo, por ejemplo:

```bash
sudo nano /opt/scripts/backup/copia_basica.sh
```

Ahora escribe lo siguiente:

```bash
#!/bin/bash
# ============================================================
#  SCRIPT: copia_basica.sh
#  DESCRIPCIÓN: Realiza una copia de seguridad comprimida
#               con fecha y hora en el nombre del archivo.
#  AUTOR: [Tu nombre]
#  FECHA: [dd/mm/yyyy]
# ============================================================

# ----- SECCIÓN 1: VARIABLES DE CONFIGURACIÓN -----
# Usamos variables para que sea fácil cambiar rutas sin tocar
# el resto del script. Es como tener los ajustes en un panel.

ORIGEN="/home/$USER/documentos"        # Qué vamos a copiar
DESTINO="/backup/datos"                # Dónde guardar la copia
FECHA=$(date +"%Y-%m-%d_%H-%M")        # Fecha y hora actuales
NOMBRE_ARCHIVO="backup_$FECHA.tar.gz"  # Nombre único del archivo
LOG="/var/log/backup/backup.log"       # Archivo de registro

# ----- SECCIÓN 2: COMPROBACIONES PREVIAS -----
# Verificar que el directorio de origen existe
if [ ! -d "$ORIGEN" ]; then
	echo "[ERROR] El directorio $ORIGEN no existe." >> "$LOG"
	exit 1   # Salir con código de error 1
fi

# Verificar que el directorio de destino existe; si no, crearlo
if [ ! -d "$DESTINO" ]; then
	mkdir -p "$DESTINO"
fi

# ----- SECCIÓN 3: REALIZAR LA COPIA -----
echo "[$(date)] Iniciando copia de seguridad..." >> "$LOG"

# tar -czf: c=crear, z=comprimir con gzip, f=nombre del archivo
# -p: preservar permisos originales
tar -czpf "$DESTINO/$NOMBRE_ARCHIVO" "$ORIGEN" 2>> "$LOG"

# ----- SECCIÓN 4: COMPROBAR EL RESULTADO -----
# $? contiene el código de salida del último comando (0 = éxito)
if [ $? -eq 0 ]; then
	echo "[$(date)] ✓ Copia creada: $NOMBRE_ARCHIVO" >> "$LOG"
	echo "Tamaño: $(du -sh "$DESTINO/$NOMBRE_ARCHIVO" | cut -f1)" >> "$LOG"
else
	echo "[$(date)] ✗ ERROR al crear la copia." >> "$LOG"
	exit 1
fi

echo "[$(date)] Copia de seguridad completada." >> "$LOG"
exit 0
```

> *Los comentarios (líneas que empiezan con **#**) son fundamentales, ellos explican cada parte y facilitan el mantenimiento futuro.*

## Dar permisos de ejecución

Un script sin permiso de ejecución es como un coche sin llave, es decir, existe, pero no arranca. Entonces, después de guardar, ejecuta:

```bash
# Dar permisos de ejecución al propietario del archivo
chmod 750 /opt/scripts/backup/copia_basica.sh

# Verificar que los permisos se han aplicado correctamente
ls -l /opt/scripts/backup/copia_basica.sh

# Deberías ver algo como:
# -rwxr-x--- 1 tuusuario tuusuario 1024 fecha copia_basica.sh
```

## Probar el script manualmente

Antes de “**automatizar**”, siempre es recomendable probar que el script funciona a mano. Esto es como ensayar una obra de teatro antes del estreno.

```bash
# Ejecutar el script directamente
bash /opt/scripts/backup/copia_basica.sh

# Ver el log para comprobar que todo ha ido bien
cat /var/log/backup/backup.log

# Listar las copias creadas
ls -lh /backup/datos/
```

> *Deberías ver en el log, algo como:  '[2025-06-15 02:00] ✓ Copia creada: **backup_2025-06-15_02-00.tar.gz**'*
> *En el directorio:  **/backup/datos/**, el **archivo.tar.gz** recién creado con su tamaño.*
> *Si ves mensajes de error, vuelve a leer el script y comprueba que las rutas existen.*

---
