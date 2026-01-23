---
title: Administración avanzada - Automatización y copias de seguridad
description:  Redes híbridas. Linux Server e integración básica con Windows. 
---

Con la infraestructura de red funcionando correctamente, llega el momento de abordar uno de los aspectos más críticos de la administración de sistemas: la automatización de tareas y la implementación de políticas de respaldo. Un buen administrador no solo construye sistemas que funcionan, sino que también garantiza su continuidad y prevé los posibles problemas antes de que ocurran.

En este bloque aprenderemos a programar tareas automáticas que se ejecuten sin intervención manual, a implementar estrategias de copias de seguridad robustas y a utilizar las herramientas modernas que Linux pone a nuestra disposición para estos fines. La automatización no solo nos ahorra tiempo, sino que también reduce errores humanos y garantiza que las tareas críticas se realicen de forma consistente.

## 4.1. Planificación de tareas con cron

### Introducción a cron

El demonio `cron` es uno de los servicios más veteranos y utilizados en sistemas Unix/Linux. Su nombre proviene de "chronos" (tiempo en griego) y su función es ejecutar comandos o scripts en momentos específicos de forma automática. Desde limpiar archivos temporales cada noche hasta generar informes semanales o ejecutar actualizaciones mensuales, cron es la herramienta fundamental para la automatización de tareas periódicas.

Cada usuario del sistema puede tener su propia tabla de cron (crontab), lo que permite una gestión granular de las tareas automatizadas. Además, existe un crontab del sistema que permite ejecutar tareas con privilegios específicos.

### Comprender la sintaxis de crontab

La sintaxis de cron puede parecer críptica al principio, pero sigue una lógica muy clara. Cada línea en un crontab representa una tarea programada y tiene el siguiente formato:

```
* * * * * comando_a_ejecutar
│ │ │ │ │
│ │ │ │ └─── Día de la semana (0-7, donde 0 y 7 son domingo)
│ │ │ └──────── Mes (1-12)
│ │ └───────────── Día del mes (1-31)
│ └────────────────── Hora (0-23)
└─────────────────────── Minuto (0-59)
```

Cada asterisco puede sustituirse por valores específicos, rangos o listas. Veamos algunos ejemplos prácticos que ilustran las posibilidades:

```bash
# Ejecutar todos los días a las 3:30 AM
30 3 * * * /ruta/script.sh

# Ejecutar cada hora en punto
0 * * * * /ruta/comando

# Ejecutar de lunes a viernes a las 8:00 AM
0 8 * * 1-5 /ruta/trabajo.sh

# Ejecutar cada 15 minutos
*/15 * * * * /ruta/monitor.sh

# Ejecutar el primer día de cada mes a las 00:00
0 0 1 * * /ruta/mensual.sh

# Ejecutar los lunes, miércoles y viernes a las 18:30
30 18 * * 1,3,5 /ruta/informe.sh
```

### Gestionar tareas cron

Para trabajar con cron, disponemos de varios comandos que nos permiten editar, visualizar y eliminar tareas programadas:

```bash
# Editar el crontab del usuario actual
crontab -e

# Ver el crontab actual
crontab -l

# Eliminar el crontab del usuario actual
crontab -r

# Editar el crontab de otro usuario (como root)
sudo crontab -u nombre_usuario -e
```

Al ejecutar `crontab -e` por primera vez, el sistema nos pedirá que elijamos un editor de texto. Para usuarios nuevos, `nano` suele ser la opción más amigable.

### Variables de entorno en cron

Las tareas de cron se ejecutan en un entorno muy limitado, sin las variables de entorno habituales de una sesión de usuario. Esto puede causar problemas si nuestros scripts dependen de rutas o configuraciones específicas. Podemos definir variables al inicio del crontab:

```bash
# Definir variables de entorno
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=admin@iscasox.local

# Tareas programadas
0 2 * * * /home/admin/scripts/backup.sh
```

La variable `MAILTO` es especialmente útil, ya que cron enviará por correo la salida de los comandos a la dirección especificada (requiere un sistema de correo configurado).

### Ejemplo práctico: limpieza automática de logs

Vamos a crear una tarea que limpie archivos de log antiguos cada semana. Primero creamos el script:

```bash
#!/bin/bash
# Script: limpiar_logs.sh
# Descripción: Elimina logs de más de 30 días

LOG_DIR="/var/log/aplicaciones"
DIAS_RETENCION=30

# Encontrar y eliminar logs antiguos
find "$LOG_DIR" -name "*.log" -type f -mtime +$DIAS_RETENCION -delete

# Registrar la limpieza
echo "$(date): Limpieza de logs completada" >> /var/log/limpieza_logs.log
```

Le damos permisos de ejecución y lo programamos:

```bash
chmod +x /home/admin/scripts/limpiar_logs.sh

# Editamos el crontab
crontab -e

# Añadimos la tarea para ejecutar cada domingo a las 2 AM
0 2 * * 0 /home/admin/scripts/limpiar_logs.sh
```

### Cron del sistema

Además de los crontabs de usuario, Linux utiliza directorios especiales para tareas del sistema ubicados en `/etc`:

```bash
/etc/cron.d/          # Archivos de cron con sintaxis completa
/etc/cron.daily/      # Scripts ejecutados diariamente
/etc/cron.hourly/     # Scripts ejecutados cada hora
/etc/cron.weekly/     # Scripts ejecutados semanalmente
/etc/cron.monthly/    # Scripts ejecutados mensualmente
```

Para añadir una tarea al sistema, simplemente colocamos un script ejecutable en el directorio correspondiente. Por ejemplo, para una tarea diaria:

```bash
sudo nano /etc/cron.daily/actualizar_sistema

#!/bin/bash
apt update
apt upgrade -y
echo "$(date): Sistema actualizado" >> /var/log/actualizaciones.log

sudo chmod +x /etc/cron.daily/actualizar_sistema
```

### Depuración de problemas con cron

Cuando una tarea cron no funciona como esperamos, hay varios lugares donde buscar información:

```bash
# Ver el log de cron
sudo tail -f /var/log/syslog | grep CRON

# Verificar que el servicio cron está activo
sudo systemctl status cron

# Probar el comando manualmente con el PATH de cron
env -i /bin/bash --noprofile --norc
/ruta/al/comando
```

Un truco útil es redirigir toda la salida de la tarea a un archivo para depuración:

```bash
0 3 * * * /ruta/script.sh >> /var/log/mi_tarea.log 2>&1
```

## 4.2. Copias de seguridad con rsync y tar

### La importancia de las copias de seguridad

Las copias de seguridad son la última línea de defensa contra la pérdida de datos. No importa cuán robusto sea nuestro sistema, siempre existe el riesgo de fallos de hardware, errores humanos, ataques maliciosos o desastres naturales. Una estrategia de backup bien implementada puede ser la diferencia entre una pequeña molestia y una catástrofe empresarial.

Existen diferentes tipos de copias de seguridad, cada una con sus ventajas: las completas copian todo pero ocupan mucho espacio, las incrementales solo copian los cambios desde la última copia (de cualquier tipo), y las diferenciales copian los cambios desde la última copia completa. En nuestro entorno utilizaremos principalmente copias completas y sincronización incremental.

### Copias de seguridad con tar

La herramienta `tar` (Tape ARchive) es el método tradicional para crear archivos de respaldo en Linux. Permite empaquetar directorios completos en un único archivo, opcionalmente comprimido:

```bash
# Crear un backup completo comprimido de /home
sudo tar -czf /backups/home-$(date +%Y%m%d).tar.gz /home

# Opciones explicadas:
# -c: crear archivo
# -z: comprimir con gzip
# -f: especificar nombre del archivo
# $(date +%Y%m%d): añade la fecha al nombre (formato AAAAMMDD)
```

Para incluir más opciones útiles y hacer el backup más robusto:

```bash
# Backup con verificación y registro detallado
sudo tar -czf /backups/home-$(date +%Y%m%d).tar.gz \
  --exclude=/home/*/.cache \
  --exclude=/home/*/Downloads \
  -v /home > /var/log/backup-$(date +%Y%m%d).log 2>&1

# Opciones adicionales:
# --exclude: excluir directorios que no necesitamos respaldar
# -v: modo verbose (detallado)
```

Para restaurar un backup creado con tar:

```bash
# Listar contenido del archivo sin extraer
sudo tar -tzf /backups/home-20250116.tar.gz

# Extraer en ubicación específica
sudo tar -xzf /backups/home-20250116.tar.gz -C /restore/

# Restaurar un archivo o directorio específico
sudo tar -xzf /backups/home-20250116.tar.gz -C /restore/ home/usuario/documentos
```

### Script automatizado de backup con tar

Creamos un script más completo que gestione rotación de backups:

```bash
#!/bin/bash
# Script: backup_tar.sh
# Descripción: Backup diario con rotación de 7 días

ORIGEN="/home /etc /var/www"
DESTINO="/backups"
FECHA=$(date +%Y%m%d_%H%M%S)
NOMBRE="backup-$FECHA.tar.gz"
DIAS_RETENCION=7

# Crear directorio de backups si no existe
mkdir -p "$DESTINO"

# Realizar backup
echo "Iniciando backup: $(date)"
tar -czf "$DESTINO/$NOMBRE" $ORIGEN \
  --exclude='*.cache' \
  --exclude='*/tmp/*' \
  2>> "$DESTINO/backup.log"

# Verificar si el backup se completó correctamente
if [ $? -eq 0 ]; then
    echo "Backup completado: $NOMBRE" | tee -a "$DESTINO/backup.log"
    
    # Eliminar backups antiguos
    find "$DESTINO" -name "backup-*.tar.gz" -type f -mtime +$DIAS_RETENCION -delete
    echo "Backups antiguos eliminados" | tee -a "$DESTINO/backup.log"
else
    echo "ERROR: Backup falló" | tee -a "$DESTINO/backup.log"
    exit 1
fi
```

### Sincronización incremental con rsync

Mientras que `tar` crea archivos comprimidos, `rsync` es ideal para sincronizar directorios manteniendo copias espejo. Solo transfiere los archivos que han cambiado, lo que lo hace muy eficiente para backups incrementales:

```bash
# Sintaxis básica de rsync
rsync -av /origen/ /destino/

# Opciones fundamentales:
# -a: modo archivo (preserva permisos, fechas, enlaces)
# -v: verbose (muestra progreso)
```

Para un backup más completo con rsync:

```bash
# Backup con más opciones
sudo rsync -avh --delete --progress \
  --exclude='*.cache' \
  --exclude='Downloads/' \
  /home/ /backups/home/

# Opciones adicionales:
# -h: formato human-readable
# --delete: elimina en destino archivos que ya no existen en origen
# --progress: muestra progreso de cada archivo
```

Una de las características más potentes de rsync es la capacidad de crear backups incrementales con enlaces duros, ahorrando espacio:

```bash
#!/bin/bash
# Script: backup_rsync_incremental.sh
# Backup incremental con enlaces duros

ORIGEN="/home"
DESTINO="/backups"
FECHA=$(date +%Y%m%d_%H%M%S)
ULTIMO_BACKUP="$DESTINO/actual"
NUEVO_BACKUP="$DESTINO/backup-$FECHA"

# Realizar backup incremental
rsync -av --delete \
  --link-dest="$ULTIMO_BACKUP" \
  "$ORIGEN/" "$NUEVO_BACKUP/"

# Actualizar enlace simbólico al último backup
rm -f "$ULTIMO_BACKUP"
ln -s "$NUEVO_BACKUP" "$ULTIMO_BACKUP"

echo "Backup incremental completado: $NUEVO_BACKUP"
```

Este método crea copias completas aparentes, pero los archivos sin cambios son enlaces duros al backup anterior, ahorrando espacio significativo.

### Backup remoto con rsync

Una buena práctica es mantener copias fuera del servidor. Rsync puede sincronizar con servidores remotos mediante SSH:

```bash
# Backup a servidor remoto
rsync -avz -e ssh /home/ usuario@servidor-backup:/backups/servidor1/

# Opciones:
# -z: compresión durante la transferencia
# -e ssh: usar SSH como transporte

# Backup desde servidor remoto (pull)
rsync -avz -e ssh usuario@servidor:/importante/ /backups/remoto/
```

Para automatizar esto sin introducir contraseñas, configuramos autenticación por clave SSH:

```bash
# Generar par de claves (si no existe)
ssh-keygen -t rsa -b 4096

# Copiar clave pública al servidor remoto
ssh-copy-id usuario@servidor-backup

# Ahora rsync funcionará sin contraseña
```

### Estrategia de backup 3-2-1

Una práctica recomendada en backups es la regla 3-2-1: mantener al menos 3 copias de los datos, en 2 tipos de medios diferentes, con 1 copia fuera del sitio. Un script que implementa parte de esta estrategia:

```bash
#!/bin/bash
# Script: backup_completo.sh
# Implementa backup local y remoto

ORIGEN="/home /etc /var/www"
LOCAL="/backups/local"
REMOTO="admin@backup-server:/backups/servidor1"
FECHA=$(date +%Y%m%d)

# Backup local con tar
echo "=== Backup local ==="
tar -czf "$LOCAL/full-$FECHA.tar.gz" $ORIGEN

# Sincronización incremental local
echo "=== Sincronización incremental ==="
rsync -av --delete $ORIGEN "$LOCAL/sync/"

# Envío a servidor remoto
echo "=== Backup remoto ==="
rsync -avz -e ssh "$LOCAL/" "$REMOTO/"

echo "Proceso de backup completado: $(date)"
```

## 4.3. Systemd timers: la alternativa moderna a cron

### Introducción a systemd timers

Aunque cron ha sido la herramienta estándar durante décadas, systemd introduce su propio sistema de temporización que ofrece ventajas significativas: mejor integración con el sistema, capacidad de gestión de dependencias, logs centralizados, y mayor flexibilidad en la programación. Los systemd timers son la evolución natural de cron para distribuciones modernas.

Un timer en systemd funciona en combinación con una unidad de servicio. El timer define cuándo ejecutar la tarea, mientras que el servicio define qué ejecutar. Esta separación de responsabilidades proporciona mayor claridad y flexibilidad.

### Crear un servicio básico

Primero creamos la unidad de servicio que realizará la tarea. Supongamos que queremos automatizar un backup:

```bash
sudo nano /etc/systemd/system/backup-diario.service
```

```ini
[Unit]
Description=Backup diario del sistema
After=network.target

[Service]
Type=oneshot
User=root
ExecStart=/usr/local/bin/backup.sh
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Los parámetros más importantes:

- `Type=oneshot` indica que el servicio se ejecuta una vez y termina
- `User=root` especifica con qué usuario se ejecuta
- `StandardOutput=journal` redirige la salida al journal de systemd para fácil consulta

### Crear el timer correspondiente

Ahora creamos el timer que programará la ejecución del servicio:

```bash
sudo nano /etc/systemd/system/backup-diario.timer
```

```ini
[Unit]
Description=Timer para backup diario
Requires=backup-diario.service

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=30min

[Install]
WantedBy=timers.target
```

Las opciones del timer explicadas:

- `OnCalendar=daily` ejecuta diariamente (equivalente a `OnCalendar=*-*-* 00:00:00`)
- `Persistent=true` ejecuta la tarea si se perdió por tener el sistema apagado
- `RandomizedDelaySec=30min` añade un retraso aleatorio (útil para distribuir carga)

### Activar y gestionar el timer

Para que el timer funcione, debemos activarlo e iniciarlo:

```bash
# Recargar systemd para que reconozca las nuevas unidades
sudo systemctl daemon-reload

# Habilitar el timer (se activará al arranque)
sudo systemctl enable backup-diario.timer

# Iniciar el timer inmediatamente
sudo systemctl start backup-diario.timer

# Verificar el estado
sudo systemctl status backup-diario.timer

# Ver todos los timers activos
systemctl list-timers --all
```

### Sintaxis de OnCalendar

La directiva `OnCalendar` admite una sintaxis flexible para programar ejecuciones:

```ini
# Cada minuto
OnCalendar=*-*-* *:*:00

# Cada hora en punto
OnCalendar=hourly
OnCalendar=*-*-* *:00:00

# Diario a las 3:30 AM
OnCalendar=*-*-* 03:30:00

# Semanalmente los lunes a las 8 AM
OnCalendar=Mon *-*-* 08:00:00

# Mensualmente el día 1 a las 00:00
OnCalendar=*-*-01 00:00:00

# Cada 15 minutos
OnCalendar=*:0/15
```

Podemos probar expresiones de calendario con:

```bash
systemd-analyze calendar "Mon *-*-* 08:00:00"
```

### Timer con múltiples programaciones

Un timer puede tener múltiples directivas `OnCalendar` para ejecutar en diferentes momentos:

```ini
[Timer]
OnCalendar=Mon,Wed,Fri 08:00:00
OnCalendar=Tue,Thu 20:00:00
Persistent=true
```

### Ejemplo completo: limpieza automática de logs

Creamos un servicio y timer para limpiar logs antiguos:

```bash
# Servicio
sudo nano /etc/systemd/system/limpieza-logs.service
```

```ini
[Unit]
Description=Limpieza de logs antiguos
Documentation=man:find(1)

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'find /var/log -name "*.log" -type f -mtime +30 -delete'
StandardOutput=journal
```

```bash
# Timer
sudo nano /etc/systemd/system/limpieza-logs.timer
```

```ini
[Unit]
Description=Ejecutar limpieza de logs semanalmente

[Timer]
OnCalendar=weekly
OnCalendar=Sun 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Activamos el timer:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now limpieza-logs.timer
```

### Consultar logs de ejecución

Una gran ventaja de systemd timers es la integración con journalctl para ver logs de ejecución:

```bash
# Ver logs del servicio de backup
sudo journalctl -u backup-diario.service

# Ver solo las últimas 20 líneas
sudo journalctl -u backup-diario.service -n 20

# Seguir los logs en tiempo real
sudo journalctl -u backup-diario.service -f

# Ver logs desde hoy
sudo journalctl -u backup-diario.service --since today

# Ver logs entre fechas
sudo journalctl -u backup-diario.service --since "2025-01-01" --until "2025-01-15"
```

### Ejecutar manualmente un servicio de timer

Para probar un servicio sin esperar al timer, podemos ejecutarlo manualmente:

```bash
# Ejecutar el servicio ahora
sudo systemctl start backup-diario.service

# Ver si se ejecutó correctamente
sudo systemctl status backup-diario.service
```

### Ejemplo avanzado: backup con dependencias

Podemos crear servicios que dependan de otros o se ejecuten en cadena:

```bash
# Servicio de pre-backup (preparación)
sudo nano /etc/systemd/system/pre-backup.service
```

```ini
[Unit]
Description=Preparación antes del backup
Before=backup-completo.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/preparar-backup.sh
```

```bash
# Servicio de backup principal
sudo nano /etc/systemd/system/backup-completo.service
```

```ini
[Unit]
Description=Backup completo del sistema
After=pre-backup.service
Wants=pre-backup.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup-completo.sh
ExecStartPost=/usr/local/bin/notificar-backup.sh
```

La directiva `Wants=` asegura que se intente ejecutar pre-backup.service, y `After=` garantiza el orden de ejecución.

### Ventajas de systemd timers sobre cron

Para concluir, resumimos por qué podríamos preferir systemd timers:

Los systemd timers ofrecen gestión centralizada mediante systemctl, logs unificados accesibles con journalctl que incluyen salida estándar y errores, y capacidad de definir dependencias entre servicios. Además, la opción Persistent ejecuta tareas perdidas tras un apagado, algo que cron no hace, y el control granular de recursos permite limitar CPU, memoria o I/O de las tareas programadas mediante directivas como CPUQuota o MemoryLimit.

Sin embargo, cron sigue siendo válido para tareas sencillas y tiene la ventaja de ser universal en todos los sistemas Unix/Linux, mientras que systemd es específico de distribuciones que lo utilizan (la mayoría de las modernas, pero no todas).

---

Con estas herramientas de automatización y respaldo, nuestro sistema está preparado para funcionar de forma autónoma y resistente a fallos. La combinación de cron o systemd timers para tareas programadas, junto con estrategias de backup usando tar y rsync, nos proporciona una base sólida para la administración profesional de servidores Linux. En los siguientes bloques aplicaremos estos conocimientos a escenarios más específicos y avanzados.