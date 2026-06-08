# ✌️ 04. Script Avanzado con Rotación de Copias

## Análisis del problema

Un script básico tiene un problema importante ***“genera copias indefinidas sin borrar las antiguas”*** y en poco tiempo, **el disco duro estará lleno**. La solución es implementar una política de **rotación de copias**, en pocas palabras, quedarnos con las más recientes, las más antiguas eliminarlas automáticamente.

Creamos el archivo, **/opt/scripts/backup/copia_avanzada.sh**, con el siguiente contenido:

```bash
#!/bin/bash
# ============================================================
# SCRIPT: copia_avanzada.sh
# DESCRIPCIÓN: Copia múltiples orígenes, rota copias antiguas
#              y genera un informe detallado.
# ============================================================

# ----- CONFIGURACIÓN -----
DESTINO="/backup/datos"
DIAS_RETENCION=7              # Guardar copias de los últimos 7 días
FECHA=$(date +"%Y-%m-%d_%H-%M")
LOG="/var/log/backup/backup.log"

# Lista de directorios a copiar (separados por espacio)
ORIGENES=("/home/$USER/documentos" "/home/$USER/proyectos" "/etc")

# ----- FUNCIÓN: escribir en log -----
# Las funciones son bloques de código reutilizables
escribir_log() {
	echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG"
}

# ----- INICIO -----
escribir_log "========== INICIO DE BACKUP =========="
ERRORES=0

# ----- BUCLE: copiar cada directorio -----
# Recorremos el array ORIGENES con un bucle for
for ORIGEN in "${ORIGENES[@]}"; do

	# Verificar que el origen existe
	if [ ! -d "$ORIGEN" ]; then
	    escribir_log "[AVISO] No existe: $ORIGEN — Saltando."
	    continue   # Pasar al siguiente elemento del bucle
	fi

	# Crear nombre único: sustituimos '/' por '_' en la ruta
	NOMBRE=$(echo "$ORIGEN" | tr '/' '_' | sed 's/^_//')
	ARCHIVO="$DESTINO/${NOMBRE}_${FECHA}.tar.gz"

	escribir_log "Copiando: $ORIGEN ..."
	tar -czpf "$ARCHIVO" "$ORIGEN" 2>> "$LOG"

	if [ $? -eq 0 ]; then
	    TAMANO=$(du -sh "$ARCHIVO" | cut -f1)
	    escribir_log "✓ OK: $ARCHIVO ($TAMANO)"
	else
	    escribir_log "✗ ERROR al copiar: $ORIGEN"
	    ERRORES=$((ERRORES + 1))
	fi
done

# ----- ROTACIÓN: eliminar copias antiguas -----
escribir_log "Eliminando copias con más de $DIAS_RETENCION días..."

# find busca archivos más antiguos de N días y los elimina
find "$DESTINO" -name "*.tar.gz" -mtime +$DIAS_RETENCION -delete
escribir_log "Rotación completada."

# ----- RESUMEN FINAL -----
ESPACIO=$(df -sh "$DESTINO" | awk 'NR==2 {print $4}')
escribir_log "Espacio libre en $DESTINO: $ESPACIO"
escribir_log "Errores encontrados: $ERRORES"
escribir_log "========== FIN DE BACKUP =========="

exit $ERRORES   # Devuelve 0 si no hubo errores
```

Vamos a desglosar el script para entenderlo mejor:

**Arrays (listas):** La variable **ORIGENES=(...)** es un array, es decir, una lista de valores. En Bash se accede a todos sus elementos con **${ORIGENES[@]}**. Son muy útiles cuando tienes varios valores del mismo tipo.

**Funciones:** La función **escribir_log()** es un bloque de código con nombre que podemos llamar tantas veces como queramos. Evita repetir el mismo código en varios sitios (el famoso principio **DRY**: **D**on't **R**epeat **Y**ourself).

**El comando find con -mtime:** Busca (find) archivos según su antigüedad. El parámetro **-mtime +7** significa *'archivos modificados hace más de 7 días'*. Combinado con **-delete**, elimina solo los archivos que cumplen ese criterio.

---
