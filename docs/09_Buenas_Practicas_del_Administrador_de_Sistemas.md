# 💊 9. Buenas Prácticas del Administrador de Sistemas

Conocer **comandos** es solo una parte de este “ecosistema”.

Las siguientes **buenas prácticas** son las que hacen la diferencia entre un **técnico competente** de uno **excelente**.

## Documentación y comentarios (básico y elemental)

Un script sin comentarios es un script que solo su autor puede mantener y lo más seguro es que ni él mismo pueda dentro de unos meses. Se recomienda comentar:

- El propósito del script al inicio (qué hace, quién lo escribió y cuándo).
- Cada sección principal del código.
- Las variables cuyo nombre no sea totalmente explícito.
- Cualquier decisión “no obvia” (¿por qué usas 30 días de retención y no 7? por ejemplo).

## Verificar los Backups de forma regular/periódica

El peor momento para enterarse de que **un backup no funciona** es precisamente **cuando los necesitas**. Por lo tanto, es muy recomendable establecer una rutina de pruebas.

- Restaurar un archivo o directorio de prueba al menos una vez al mes.
- Usar el comando `tar -tzf` para **listar el contenido sin extraer** y verificar integridad.
- Comprueba el tamaño de las copias, si notas que son más pequeñas, algo falla.

```bash
# Listar el contenido de un backup sin extraerlo
tar -tzf /backup/datos/mi_backup_2025-06-15.tar.gz | head -20

# Restaurar solo un archivo concreto de dentro del backup
tar -xzf /backup/datos/mi_backup_2025-06-15.tar.gz \
		-C /tmp/recuperacion/ home/usuario/documentos/informe.pdf
```

## Realizar un Cheklist de Seguridad del Entorno de Backup

Antes de poner en marcha un sistema de backups, sería bueno repasar esta lista:

<aside>
💡

**Lista de comprobación / Sistema de backups**

☐   Los scripts tienen permisos 750 (solo el propietario puede ejecutarlos).

☐   El directorio de backups tiene permisos 750 o más restrictivos.

☐   Las contraseñas de cifrado están en archivos con permisos 600, no en el script.

☐   El crontab está configurado y los trabajos aparecen en crontab -l.

☐   Los logs se están escribiendo correctamente en cada ejecución.

☐   Se ha probado la restauración de al menos un archivo de backup.

☐   Las notificaciones por email funcionan (se ha recibido un correo de prueba).

☐   Existe al menos una copia en una ubicación diferente al servidor principal.

</aside>

---