# 🪄 06. Automatización con cron y crontab

Y llegamos a la parte importante, programarlo para que se ejecute solo, sin intervención nuestra (automatizar).

## Editamos el crontab de usuario

Cada usuario tiene su propio **crontab** (tabla de tareas programadas), así es que vamos a editarlo:

```bash
# Abrir el crontab del usuario actual para editarlo
crontab -e

# La primera vez te preguntará qué editor usar. Elige nano (opción 1)
# para trabajar más cómodamente.

# Ver las tareas programadas actualmente
crontab -l

# Borrar todo el crontab (¡cuidado!)
crontab -r
```

## Sintaxis de crontab (explicado al detalle)

El formato de cada línea del crontab, **tiene cinco campos de tiempo**, seguidos del comando. Ver la siguiente tabla de referencia completa:

| Campo | Valores válidos / Significado |
| --- | --- |
| Minuto | 0-59 (el minuto exacto en que se ejecuta) |
| Hora | 0-23 (la hora del día, en formato 24h) |
| Día del mes | 1-31 (el día del mes) |
| Mes | 1-12 (el número del mes) |
| Día de la semana | 0-7 (0 y 7 = domingo, 1 = lunes, ..., 6 = sábado) |
| *  (asterisco) | Significa 'cualquier valor' o 'todos' |
| */N (barra) | Significa 'cada N unidades' (ej.: */5 minutos = cada 5 minutos) |
| ,  (coma) | Lista de valores (ej.: 1,3,5 = los días 1, 3 y 5) |
| -  (guión) | Rango (ej.: 8-17 = desde las 8h hasta las 17h) |

## Ejemplo de programación crontab

```bash
# ──────────────────────────────────────────────
# EJEMPLOS DE TAREAS PROGRAMADAS CON CRON
# ──────────────────────────────────────────────

# Cada día a las 2:00 de la madrugada (ideal para backups nocturnos)
0 2 * * * /opt/scripts/backup/copia_avanzada.sh

# Cada lunes a las 3:30 (backup semanal)
30 3 * * 1 /opt/scripts/backup/copia_semanal.sh

# El primer día de cada mes a medianoche (backup mensual)
0 0 1 * * /opt/scripts/backup/copia_mensual.sh

# Cada 6 horas (para datos muy críticos)
0 */6 * * * /opt/scripts/backup/copia_critica.sh

# Cada 30 minutos solo en horario laboral (lunes a viernes, 8h-17h)
*/30 8-17 * * 1-5 /opt/scripts/backup/copia_laboral.sh

# Redirigir la salida al log (recomendado para ver posibles errores)
0 2 * * * /opt/scripts/backup/copia_avanzada.sh >> /var/log/backup/cron.log 2>&1
```

> **IMPORTANTE:**
>
> - En **Bash**, la salida de un comando va a 'dos canales': la salida estándar (**stdout**, código 1) y la salida de errores (**stderr**, código 2).
> - El símbolo `>>` redirige la salida estándar al archivo de **log**.
> - El `2>&1` hace que los errores (**stderr**) vayan al mismo sitio que la salida estándar. Así no se pierden los mensajes de error.

---
