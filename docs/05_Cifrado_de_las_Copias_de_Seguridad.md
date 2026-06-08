# 🔐 5. Cifrado de las Copias de Seguridad

## Protegiendo las copias (cifrado con GPG)

Está claro que **existe un riesgo latente** si tenemos copias de seguridad sin una protección adecuada, así que vamos a solucionarlo con el cifrado.

## Instalación y verificación de GPG

**GPG** (**G**NU **P**rivacy **G**uard) normalmente viene instalado por defecto. Vamos a comprobarlo:

```bash
# Verificar si GPG está instalado
gpg --version

# Si no está, instalarlo (en Ubuntu/Debian):
sudo apt update && sudo apt install gnupg -y

# Podéis verificar el “estado” de las actualizaciones, por si demora.
systemctl status unattended-upgrades
```

## Cifrado con contraseña simétrica

La forma más simple y sencilla de cifrar una copia de seguridad es con una contraseña. El **cifrado simétrico** utiliza la misma clave para poder **cifrar y descifrar**. Para la **automatización**, podemos evitar pedirla de forma interactiva usando `--batch` y `--passphrase-fd`.

```bash
# Cifrar un archivo de backup con contraseña (forma interactiva)
gpg --symmetric --cipher-algo AES256 backup_2025-06-15.tar.gz
# Genera: backup_2025-06-15.tar.gz.gpg

# Para automatización (contraseña en variable de entorno):
export GPG_PASS="mi_contraseña_segura"
echo "$GPG_PASS" | gpg --batch --yes --passphrase-fd 0 \
		--symmetric --cipher-algo AES256 backup.tar.gz
		
# Descifrar el backup cuando lo necesitemos:
gpg --decrypt backup_2025-06-15.tar.gz.gpg > backup_recuperado.tar.gz
```

> **NOTA IMPORTANTE:**
> 
> 
> NUNCA escribas la contraseña directamente en el script si este archivo va a estar en un lugar compartido. En entornos de producción, usa un gestor de secretos o guarda la contraseña en un archivo con permisos **600** (solo lectura del propietario) en un directorio seguro.
> 

## **Integración del cifrado en el script de backup**

Cifrado automático al final del script, añadirlo después de crear el **.tar.gz**:

```bash
# Añadir después de la creación del tar.gz:

# Ruta al archivo con la contraseña (permisos 600)
ARCHIVO_PASS="/opt/scripts/backup/.gpg_pass"

# Cifrar el backup
gpg --batch --yes --passphrase-file "$ARCHIVO_PASS" \
		--symmetric --cipher-algo AES256 "$ARCHIVO"

if [ $? -eq 0 ]; then
		# Si el cifrado fue bien, borrar el .tar.gz sin cifrar
		rm -f "$ARCHIVO"
		escribir_log "✓ Backup cifrado correctamente."
else
		escribir_log "✗ ERROR en el cifrado."
fi
```

---