# 📌 8. Script Completo y Profesional

Contiene todo lo anteriormente visto, con explicación línea por línea, recomiendo guardarlo como: `copia_profesional.sh`

```bash
#!/bin/bash
# ================================================================
#  PRÁCTICA DE UN SCRIPT PROFESIONAL DE BACKUP
#
#  Combina todo lo que se requiere para un backup profesional
#
#  *tar: Permite comprimir archivos y carpetas en un solo archivo.
#  *rotación: Evita la acumulación de backups en el tiempo.
#  *verificación: Realiza una comprobación para evitar errores.
#  *cifrado: Solución para evitar que cualquiera tenga acceso.
#  *notificación: Envía un correo usando msmtp (compatible Gmail).
#
#  AUTOR: [Nombre]
#  FECHA: [dd-mm-yyyy]
# ================================================================

# ── CONFIGURACIÓN ─────────────────────────────────────────────────
# Aquí definimos todas las variables que controlan el comportamiento
# del script. Tenerlas juntas al principio facilita el mantenimiento:
# si quieres cambiar algo, no tienes que buscar por todo el código.

ORIGENES=("/home/$USER/documentos" "/home/$USER/datos1" "/home/$USER/datos2")
# ORIGENES es un array (lista) de carpetas que queremos copiar.
# $USER se expande automáticamente al nombre del usuario actual.

DESTINO="/backup/datos"
# Carpeta donde se guardarán todos los archivos de backup.

DIAS_RETENCION=30
# Política de retención: se eliminarán las copias con más de 30 días.
# Es el equivalente digital a "tirar los yogures caducados".

EMAIL="admin@midominio.com"
# Dirección de correo que recibirá las notificaciones.
# Puede ser tu Gmail, una cuenta corporativa, etc.

REMITENTE="backup@midominio.com"
# NUEVO: La dirección que aparece en el campo "De:" del correo.
# Debe coincidir con la cuenta configurada en ~/.msmtprc.
# Cambiar este valor no afecta a ninguna otra parte del script.

ARCHIVO_PASS="/opt/scripts/backup/.gpg_pass"
# Ruta al archivo que contiene la contraseña para el cifrado GPG.
# Este archivo debe tener permisos 600 (solo lectura del propietario).
# La contraseña NUNCA se escribe directamente en el script.

LOG="/var/log/backup/backup.log"
# Archivo donde el script registra todo lo que hace.
# Es fundamental para saber qué pasó si algo falla a las 2 AM.

FECHA=$(date +"%Y-%m-%d_%H-%M")
# Captura la fecha y hora actuales en formato legible y sin espacios.
# El paréntesis con $(...) ejecuta el comando date y guarda su resultado.
# Ejemplo de valor: 2026-04-04_02-00

HOST=$(hostname)
# Captura el nombre del equipo o servidor donde se ejecuta el script.
# Muy útil si el mismo script se usa en varios servidores a la vez:
# así sabes exactamente desde cuál servidor llegó la notificación.

ERRORES=0
# Contador de errores inicializado a cero.
# Lo iremos incrementando si algo falla durante el proceso.

# ── FUNCIONES ─────────────────────────────────────────────────────
# Las funciones son bloques de código reutilizables con nombre propio.
# En lugar de repetir el mismo código varias veces, lo escribimos
# una vez como función y lo llamamos cuando lo necesitemos.

log() { echo "[$(date '+%H:%M:%S')] $1" >> "$LOG"; }
# Función log(): escribe una línea en el archivo de log.
# $1 es el primer argumento que le pasemos al llamarla.
# Ejemplo de uso: log "Iniciando backup"
# Genera en el log: [02:00:01] Iniciando backup
# El >> añade la línea al final del archivo sin borrar lo anterior.

notificar() {
		# COMPARACIÓN respecto a la versión con mailutils.
		# Con mailutils usaríamos: echo -e "$2" | mail -s "$1" "$EMAIL"
		# Con msmtp, construimos el correo completo a mano y lo pasamos a msmtp.
		
		# printf construye el cuerpo del correo con las cabeceras estándar
		# que necesitará msmtp para enviarlo correctamente por SMTP:
		#   Subject: → es el asunto del correo
		#   From:    → dirección del remitente (la que configuramos en msmtprc)
		#   To:      → destinatario
		#   (línea vacía) → separador obligatorio entre cabeceras y cuerpo
		#   $2       → el cuerpo del mensaje
		
		printf "Subject: %s\nFrom: %s\nTo: %s\n\n%b\n" \
				"$1" "$REMITENTE" "$EMAIL" "$2" \
				| msmtp --account=gmail "$EMAIL" 2>/dev/null
		
		# msmtp --account=gmail → usa la cuenta "gmail" del archivo ~/.msmtprc
		# "$EMAIL" → destinatario final del correo
		# 2>/dev/null → si msmtp da algún aviso menor, no lo muestra en pantalla
		#               (los errores graves sí quedarán registrados en ~/.msmtp.log)
}

# ── INICIO ────────────────────────────────────────────────────────
log "===== INICIO BACKUP — $HOST ====="
# Marcamos en el log el comienzo del proceso con el nombre del servidor.

[ ! -d "$DESTINO" ] && mkdir -p "$DESTINO"
# Si el directorio de destino NO existe (!), lo creamos con mkdir -p.
# La -p hace que cree también los directorios intermedios si faltan.
# Esta línea es una forma compacta del clásico: if [ ! -d ... ]; then

# ── BUCLE DE COPIAS ───────────────────────────────────────────────
# Recorremos el array ORIGENES con un bucle for.
# En cada iteración, la variable ORIGEN toma el valor de un directorio.

for ORIGEN in "${ORIGENES[@]}"; do

		[ ! -d "$ORIGEN" ] && log "[SKIP] No existe: $ORIGEN" && continue
		# Si el directorio de origen no existe, lo anotamos en el log
		# y pasamos al siguiente con 'continue' (no es un error grave,
		# puede que simplemente ese directorio no exista en este servidor).
		
		NOMBRE=$(echo "$ORIGEN" | tr '/' '_' | sed 's/^_//')
		# Convertimos la ruta del origen en un nombre válido para el archivo.
		# tr '/' '_' → sustituye todas las barras '/' por guiones bajos '_'
		# sed 's/^_//' → elimina el guion bajo inicial que queda al principio
		# Ejemplo: /home/usuario/documentos → home_usuario_documentos
		
		ARCHIVO="$DESTINO/${NOMBRE}_${FECHA}.tar.gz"
		# Construimos la ruta completa del archivo de backup con nombre único.
		# Ejemplo: /backup/datos/home_usuario_documentos_2026-04-04_02-00.tar.gz
		
		log "Copiando: $ORIGEN"
		tar -czpf "$ARCHIVO" "$ORIGEN" 2>> "$LOG" || { ERRORES=$((ERRORES+1)); continue; }
		# tar -czpf: c=crear, z=comprimir con gzip, p=preservar permisos, f=nombre
		# Los errores de tar (2>>) se añaden también al log.
		# || { ... } → si tar falla (devuelve código distinto de 0), incrementamos
		# el contador de errores y pasamos al siguiente origen con 'continue'.
		
		# ── Verificar integridad ──
		tar -tzf "$ARCHIVO" > /dev/null 2>&1 || {
				log "✗ Archivo corrupto: $ARCHIVO"
				rm -f "$ARCHIVO"
				ERRORES=$((ERRORES+1))
				continue
		}
		# tar -tzf lista el contenido del .tar.gz sin extraerlo.
		# Si el archivo está corrupto, tar devuelve error y entramos en el bloque ||.
		# Eliminamos el archivo dañado para no confundirlo con uno válido.
		# > /dev/null suprime la salida normal (no nos interesa ver la lista aquí).
		
		# ── Cifrar si existe el archivo de contraseña ──
		if [ -f "$ARCHIVO_PASS" ]; then
				gpg --batch --yes --passphrase-file "$ARCHIVO_PASS" \
						--symmetric --cipher-algo AES256 "$ARCHIVO" 2>> "$LOG"
				[ $? -eq 0 ] && rm -f "$ARCHIVO" && log "✓ Cifrado: ${ARCHIVO}.gpg"
		fi
		# Si el archivo de contraseña GPG existe, ciframos el .tar.gz con AES256.
		# --batch → modo no interactivo (no pide confirmaciones)
		# --symmetric → cifrado con contraseña (no necesita clave pública/privada)
		# Si el cifrado tiene éxito ($? -eq 0), borramos el .tar.gz sin cifrar.
		# Así nunca conviven en disco la copia cifrada y la sin cifrar.
		
		TAMANO=$(du -sh "${ARCHIVO}"* 2>/dev/null | head -1 | cut -f1)
		log "✓ OK: $NOMBRE ($TAMANO)"
		# du -sh obtiene el tamaño del archivo en formato legible (ej: 24M, 1.2G).
		# El * al final del nombre cubre tanto .tar.gz como .tar.gz.gpg.
		# head -1 toma solo la primera línea por si hubiera más de un resultado.
		# cut -f1 extrae solo la columna del tamaño, sin la ruta.

done

# ── ROTACIÓN ──────────────────────────────────────────────────────
log "Rotación: eliminando copias > $DIAS_RETENCION días"
find "$DESTINO" \( -name "*.tar.gz" -o -name "*.gpg" \) \
		-mtime +$DIAS_RETENCION -delete
# find busca archivos dentro de $DESTINO que cumplan TODAS estas condiciones:
# \( -name "*.tar.gz" -o -name "*.gpg" \) → que sean .tar.gz O .gpg
# -mtime +30 → modificados hace MÁS de 30 días
# -delete → los elimina directamente (sin moverlos a papelera)
# Es nuestra política de retención automática: solo guardamos 30 días.

# ── RESUMEN ───────────────────────────────────────────────────────
LIBRE=$(df -sh "$DESTINO" | awk 'NR==2{print $4}')
log "Espacio libre: $LIBRE | Errores: $ERRORES"
log "===== FIN BACKUP ====="
# df -sh muestra el uso de disco del sistema de archivos donde está $DESTINO.
# awk 'NR==2{print $4}' → coge la segunda línea (la de datos) y extrae
# la columna 4 (espacio disponible). Ejemplo de resultado: 47G

# ── NOTIFICACIÓN ──────────────────────────────────────────────────
if [ $ERRORES -eq 0 ]; then
		notificar "[OK] Backup $HOST - $FECHA" \
				"Backup completado sin errores.\nEspacio libre: $LIBRE"
else
		notificar "[ERROR] Backup $HOST - $FECHA" \
				"Se encontraron $ERRORES errores.\n$(tail -10 $LOG)"
fi

# Si no hubo errores, enviamos una notificación de éxito.
# Si hubo errores, enviamos las últimas 10 líneas del log como cuerpo
# del correo, para que el administrador vea exactamente qué falló
# sin necesidad de conectarse al servidor.

exit $ERRORES
# Salimos con el número de errores como código de salida.
# exit 0 = todo correcto.
# exit N (N > 0) = hubo N errores. Esto permite que otros sistemas
# (como cron o scripts de monitorización) detecten si hubo problemas.
```

---