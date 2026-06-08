# 🛠️ 02. Preparación del entorno de trabajo seguro

## Vamos a preparar el entorno

Previo a crear cualquier script, debemos preparar la estructura de directorios con los permisos correctos. Ejecuta los siguientes comandos en tu terminal:

```bash
# Paso 1: Crear el directorio donde guardaremos los scripts
sudo mkdir -p /opt/scripts/backup

# Paso 2: Crear el directorio de destino para las copias
sudo mkdir -p /backup/datos

# Paso 3: Crear el directorio para los registros (logs)
sudo mkdir -p /var/log/backup

# Paso 4: Asignamos propietario (sustituye 'tuusuario' por tu nombre de usuario)
sudo chown -R tuusuario:tuusuario /opt/scripts/backup
sudo chown -R tuusuario:tuusuario /backup
sudo chown -R tuusuario:tuusuario /var/log/backup

# Paso 5: Dar permisos correctos a los directorios
chmod 750 /opt/scripts/backup   # Solo el propietario puede leer/escribir/entrar
chmod 750 /backup               # Solo el propietario accede a sus copias
chmod 750 /var/log/backup       # Solo el propietario accede a sus logs
```

En Linux los permisos de `r` + `w` + `x`, son representados en octal como 4 + 2 + 1, tener los 3 permisos equivale a tener el valor de 7, como se ve en la representación siguiente:

- `rwx` **= 7** (4+2+1)
- `rw-` **= 6** (4+2+0)
- `r-x` **= 5** (4+0+1)
- `r--` **= 4** (4+0+0)
- `---` **= 0** (Sin permisos)

Entonces, **¿porqué 750 y no 777?**, pues porque el permiso 777 significa que cualquier usuario del sistema puede leer, modificar y ejecutar (`rwxrwxrwx`), sería un enorme riesgo de seguridad. Sin embargo, con el permiso 750 (`rwxr-x---`), el propietario tiene el control total, el grupo puede leer y ejecutar y el resto de usuarios (todos los demás) no pueden ni ver qué hay dentro.

---
