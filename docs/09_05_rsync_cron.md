---
title: Sincronización con rsync y Automatización con cron
description: Copias incrementales con rsync y programación de tareas con cron
---

En esta sesión aprenderemos dos herramientas fundamentales: `rsync` para sincronizar directorios de forma eficiente (copiando solo cambios) y `cron` para automatizar tareas y ejecutar scripts a horas programadas sin intervención manual.

# PARTE 1: rsync - Sincronización de Directorios

## ¿Qué es rsync?

`rsync` es una herramienta que sincroniza directorios copiando solo los archivos nuevos o modificados, lo que lo hace mucho más eficiente que copiar todo cada vez.

**Diferencias con tar:**

| Característica | tar | rsync |
|----------------|-----|-------|
| Resultado | Archivo comprimido | Copia de directorios |
| Velocidad | Siempre copia todo | Solo copia cambios |
| Uso | Backups archivados | Sincronización, backups incrementales |
| Espacio | Un archivo por backup | Mantiene estructura de carpetas |

**Cuándo usar cada uno:**
- **tar**: Backups que quieres archivar y comprimir
- **rsync**: Mantener dos directorios sincronizados

## Comandos Básicos

La sintaxis básica es:

```bash
rsync -av origen/ destino/
```

**Opciones importantes:**
- `-a` → modo archivo (preserva permisos, fechas, enlaces)
- `-v` → verbose (muestra qué está haciendo)

!!! warning "La barra final importa"
    - `origen/` → copia el **CONTENIDO** de origen
    - `origen` → copia la **CARPETA** origen completa

!!! example "Sincronizar Documentos a backup"

    ```bash
    # Sincronizar Documentos a backup
    rsync -av ~/Documentos/ ~/backup_documentos/
    ```

    **Primera vez:** Copia todo  
    **Siguientes veces:** Solo copia archivos nuevos o modificados

## Opciones Útiles

**Ver qué haría sin copiar (simulación):**

```bash
rsync -avn origen/ destino/
```

La opción `-n` hace una simulación (dry-run) sin ejecutar nada. Muy útil para verificar qué se va a copiar antes de hacerlo.

!!! example "Simular sincronización"

    ```bash
    # Ver qué se copiaría sin copiar realmente
    rsync -avn ~/Documentos/ ~/backup_documentos/
    ```

**Mostrar progreso:**

```bash
rsync -av --progress origen/ destino/
```

**Excluir archivos:**

```bash
rsync -av --exclude='*.tmp' --exclude='.cache' origen/ destino/
```

**Sincronización espejo (elimina en destino lo que no está en origen):**

```bash
rsync -av --delete origen/ destino/
```

!!! warning "Cuidado con --delete"
    La opción `--delete` elimina archivos en destino que no existen en origen. Usar con precaución.

## Script de Sincronización Básico

!!! example "Script simple de sincronización"

    ```bash
    #!/bin/bash
    # Script de sincronización con rsync

    ORIGEN="$HOME/Documentos/"
    DESTINO="$HOME/backup_sync/"

    echo "Sincronizando $ORIGEN a $DESTINO..."

    rsync -av --progress "$ORIGEN" "$DESTINO"

    if [ $? -eq 0 ]; then
        echo "✓ Sincronización completada"
    else
        echo "✗ Error en la sincronización"
    fi
    ```

## Backup Incremental con rsync

Una de las ventajas principales de rsync es que solo copia lo que ha cambiado, ahorrando tiempo y espacio.

!!! example "Backup incremental"

    ```bash
    #!/bin/bash
    # Backup incremental

    ORIGEN="$HOME/Documentos/"
    DESTINO="$HOME/backup_incremental/"

    # Crear destino si no existe
    mkdir -p "$DESTINO"

    echo "Realizando backup incremental..."
    echo "Solo se copiarán archivos nuevos o modificados"

    rsync -av \
        --exclude='*.tmp' \
        --exclude='.cache' \
        "$ORIGEN" "$DESTINO"

    if [ $? -eq 0 ]; then
        echo "✓ Backup completado"
        
        # Mostrar estadísticas
        total=$(find "$DESTINO" -type f | wc -l)
        espacio=$(du -sh "$DESTINO" | cut -f1)
        
        echo "  Archivos totales: $total"
        echo "  Espacio usado: $espacio"
    fi
    ```

---

# PARTE 2: cron - Automatización de Tareas

## ¿Qué es cron?

`cron` es un servicio que ejecuta comandos o scripts automáticamente en horarios programados. Funciona continuamente en segundo plano revisando si hay tareas que ejecutar.

**Usos comunes:**
- Backup todas las noches a las 2 AM
- Limpiar archivos temporales cada domingo
- Actualizar el sistema semanalmente
- Generar informes cada lunes
- Verificar servicios cada 5 minutos

**¿Cómo funciona?**
1. Cron revisa constantemente tu tabla de tareas (crontab)
2. Cuando llega la hora programada, ejecuta el comando
3. Todo sucede automáticamente, incluso sin estar conectado

## Sintaxis de crontab

Cada línea en el crontab tiene este formato:

```
* * * * * comando
│ │ │ │ │
│ │ │ │ └─── Día de semana (0-7, 0 y 7 = domingo)
│ │ │ └──────── Mes (1-12)
│ │ └───────────── Día del mes (1-31)
│ └────────────────── Hora (0-23)
└─────────────────────── Minuto (0-59)
```

**Tabla de patrones:**

| Patrón | Significado |
|--------|-------------|
| `*` | Cualquier valor |
| `*/5` | Cada 5 unidades |
| `2,4,6` | Valores específicos (2, 4 y 6) |
| `1-5` | Rango (1 a 5) |
| `0-23/2` | Cada 2 horas |

!!! example "Ejemplos de horarios comunes"

    ```bash
    # Todos los días a las 2:00 AM
    0 2 * * * /ruta/script.sh

    # Cada hora en punto
    0 * * * * /ruta/script.sh

    # Cada 15 minutos
    */15 * * * * /ruta/script.sh

    # Cada 30 minutos
    */30 * * * * /ruta/script.sh

    # De lunes a viernes a las 8:00 AM
    0 8 * * 1-5 /ruta/script.sh

    # Domingos a las 3:00 AM
    0 3 * * 0 /ruta/script.sh

    # Primer día de cada mes a las 00:00
    0 0 1 * * /ruta/script.sh

    # Lunes y viernes a las 18:30
    30 18 * * 1,5 /ruta/script.sh
    ```

## Gestionar crontab

**Ver tu crontab actual:**

```bash
crontab -l
```

Si no has creado ninguna tarea, dirá: `no crontab for usuario`

**Editar tu crontab:**

```bash
crontab -e
```

!!! note "Primera vez"
    Te preguntará qué editor usar. Elige `nano` (opción 1) si eres principiante.

**Eliminar todo tu crontab:**

```bash
crontab -r
```

!!! warning "Sin confirmación"
    Este comando borra todas tus tareas sin preguntar.

## Añadir Tareas a cron - Paso a Paso

**1. Crear el script:**
```bash
nano ~/scripts/backup_diario.sh
```

```bash
#!/bin/bash
# Script de backup para cron

FECHA=$(date +%Y%m%d)
tar -czf ~/backups/backup_$FECHA.tar.gz ~/Documentos

echo "$(date): Backup completado" >> ~/logs/backup.log
```

**2. Dar permisos:**
```bash
chmod +x ~/scripts/backup_diario.sh
```

**3. Editar crontab:**
```bash
crontab -e
```

**4. Añadir línea:**
```bash
# Backup diario a las 2 AM
0 2 * * * /home/usuario/scripts/backup_diario.sh
```

**5. Guardar y salir:**
- En nano: `Ctrl+O`, `Enter`, `Ctrl+X`

**6. Verificar:**
```bash
crontab -l
```

## Reglas Importantes para cron

!!! warning "1. Usar rutas ABSOLUTAS"

    ```bash
    # ❌ Mal - ruta relativa
    0 2 * * * ./backup.sh

    # ✅ Bien - ruta absoluta
    0 2 * * * /home/usuario/scripts/backup.sh
    ```

!!! warning "2. Especificar rutas completas en el script"

    ```bash
    #!/bin/bash
    # ❌ Mal
    cd ~/backups
    tar -czf backup.tar.gz ~/Documentos

    # ✅ Bien
    tar -czf /home/usuario/backups/backup.tar.gz /home/usuario/Documentos
    ```

!!! warning "3. Redirigir salida a logs"

    ```bash
    # Sin log (output se pierde)
    0 2 * * * /home/usuario/backup.sh

    # Con log (puedes ver qué pasó)
    0 2 * * * /home/usuario/backup.sh >> /home/usuario/backup.log 2>&1
    ```

    **Explicación:**
    - `>>` → Añade al archivo (no sobrescribe)
    - `2>&1` → Redirige errores también al mismo archivo

## Ver Logs de cron

**Ver ejecuciones de cron:**

```bash
grep CRON /var/log/syslog | tail -n 20
```

**Seguir en tiempo real:**

```bash
sudo tail -f /var/log/syslog | grep CRON
```

**Ver logs de tu script:**

Si tu script registra en un log propio:

```bash
tail -f ~/logs/backup.log
```

## Ejemplos Completos

!!! example "Ejemplo 1: Backup Diario Automático"

    **Script:** `~/scripts/backup_auto.sh`
    ```bash
    #!/bin/bash
    # Backup automático para ejecutar con cron

    LOG="/home/$USER/logs/backup.log"
    DESTINO="/home/$USER/backups"
    FECHA=$(date +%Y%m%d_%H%M%S)

    # Crear directorios si no existen
    mkdir -p "$DESTINO"
    mkdir -p "$(dirname $LOG)"

    # Registrar inicio
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Iniciando backup" >> "$LOG"

    # Crear backup
    tar -czf "$DESTINO/backup_$FECHA.tar.gz" \
        --exclude='*.tmp' \
        --exclude='.cache' \
        /home/$USER/Documentos 2>> "$LOG"

    if [ $? -eq 0 ]; then
        tamano=$(du -h "$DESTINO/backup_$FECHA.tar.gz" | cut -f1)
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup OK: $tamano" >> "$LOG"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - ERROR en backup" >> "$LOG"
    fi

    # Eliminar backups antiguos (más de 7 días)
    find "$DESTINO" -name "backup_*.tar.gz" -mtime +7 -delete
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Limpieza completada" >> "$LOG"
    ```

    **Crontab:**
    ```bash
    # Ejecutar todos los días a las 2 AM
    0 2 * * * /home/usuario/scripts/backup_auto.sh
    ```

---

!!! example "Ejemplo 2: Sincronización con rsync"

    **Script:** `~/scripts/sincronizar.sh`
    ```bash
    #!/bin/bash
    # Sincronización automática con rsync

    ORIGEN="/home/$USER/Documentos/"
    DESTINO="/home/$USER/backup_sync/"
    LOG="/home/$USER/logs/sync.log"

    mkdir -p "$DESTINO"
    mkdir -p "$(dirname $LOG)"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - Iniciando sincronización" >> "$LOG"

    rsync -av \
        --exclude='*.tmp' \
        --exclude='.cache' \
        --delete \
        "$ORIGEN" "$DESTINO" >> "$LOG" 2>&1

    if [ $? -eq 0 ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Sincronización OK" >> "$LOG"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - ERROR" >> "$LOG"
    fi
    ```

    **Crontab:**
    ```bash
    # Sincronizar cada hora
    0 * * * * /home/usuario/scripts/sincronizar.sh
    ```

---

!!! example "Ejemplo 3: Limpieza Semanal"

    **Script:** `~/scripts/limpieza.sh`
    ```bash
    #!/bin/bash
    # Limpieza semanal de archivos temporales

    LOG="/home/$USER/logs/limpieza.log"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - Iniciando limpieza" >> "$LOG"

    # Eliminar archivos .tmp
    eliminados=$(find /home/$USER -name "*.tmp" -type f | wc -l)
    find /home/$USER -name "*.tmp" -type f -delete

    echo "$(date '+%Y-%m-%d %H:%M:%S') - Eliminados $eliminados archivos .tmp" >> "$LOG"

    # Limpiar caché
    rm -rf /home/$USER/.cache/thumbnails/*
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Caché limpiada" >> "$LOG"
    ```

    **Crontab:**
    ```bash
    # Ejecutar cada domingo a las 3 AM
    0 3 * * 0 /home/usuario/scripts/limpieza.sh
    ```

## Ejercicios Prácticos

!!! question "Ejercicio 1: Sincronización con rsync"

    **Objetivo:** Crear un script que sincronice Documentos.

    **Instrucciones:**
    1. Crear `sincronizar.sh`
    2. Sincronizar `~/Documentos/` a `~/backup_sync/`
    3. Excluir `.cache` y `*.tmp`
    4. Mostrar mensaje de éxito

    ??? success "Solución"
        ```bash
        #!/bin/bash

        ORIGEN="$HOME/Documentos/"
        DESTINO="$HOME/backup_sync/"

        echo "Sincronizando..."

        rsync -av \
            --exclude='.cache' \
            --exclude='*.tmp' \
            "$ORIGEN" "$DESTINO"

        if [ $? -eq 0 ]; then
            echo "✓ Sincronización completada"
            archivos=$(find "$DESTINO" -type f | wc -l)
            echo "  Total de archivos: $archivos"
        else
            echo "✗ Error en sincronización"
        fi
        ```

    **Probar:**
    ```bash
    chmod +x sincronizar.sh
    ./sincronizar.sh

    # Modificar un archivo en Documentos
    echo "prueba" > ~/Documentos/test.txt

    # Ejecutar de nuevo (solo copiará el cambio)
    ./sincronizar.sh
    ```

---

!!! question "Ejercicio 2: Primera Tarea con cron"

    **Objetivo:** Programar un script simple que se ejecute cada 2 minutos (para pruebas).

    **Instrucciones:**
    1. Crear script `prueba_cron.sh` que escriba la fecha en un log
    2. Darle permisos
    3. Programarlo con cron para cada 2 minutos
    4. Verificar que funciona

    ??? success "Solución paso a paso"

        **1. Crear script:**
        ```bash
        nano ~/prueba_cron.sh
        ```

        ```bash
        #!/bin/bash
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Cron funcionando" >> ~/cron_test.log
        ```

        **2. Dar permisos:**
        ```bash
        chmod +x ~/prueba_cron.sh
        ```

        **3. Editar crontab:**
        ```bash
        crontab -e
        ```

        **Añadir:**
        ```bash
        # Prueba cada 2 minutos
        */2 * * * * /home/usuario/prueba_cron.sh
        ```

        **4. Verificar:**
        ```bash
        # Ver que se añadió
        crontab -l

        # Esperar 4-5 minutos y ver el log
        cat ~/cron_test.log
        ```

        **5. Eliminar la tarea cuando funcione:**
        ```bash
        crontab -e
        # Borrar o comentar la línea (añadir # al inicio)
        ```

---

!!! question "Ejercicio 3: Sistema de Backup Automatizado (PARA ENTREGAR)"

    **Objetivo:** Crear un sistema completo de backup automatizado con cron.

    **Requisitos:**
    1. Script llamado `backup_sistema.sh` que:
       - Haga backup de `~/Documentos` con tar
       - Lo guarde en `~/backups/` con fecha
       - Excluya `.cache` y `*.tmp`
       - Registre en `~/logs/backup.log` con fecha y hora
       - Elimine backups de más de 5 días
       - Muestre resumen al final del log

    2. Programar con cron para:
       - Ejecutarse todos los días a las 3 AM
       - Redirigir salida al log

    **Estructura del script:**
    ```bash
    #!/bin/bash
    # Sistema de backup automatizado

    LOG="$HOME/logs/backup.log"
    DESTINO="$HOME/backups"
    FECHA=$(date +%Y%m%d_%H%M%S)

    # Crear directorios
    # Registrar inicio
    # Crear backup con exclusiones
    # Verificar resultado
    # Registrar tamaño
    # Eliminar backups antiguos
    # Registrar resumen
    ```

    **Crontab esperado:**
    ```bash
    # Backup diario a las 3 AM
    0 3 * * * /home/usuario/backup_sistema.sh >> /home/usuario/logs/backup.log 2>&1
    ```

    **Ejemplo de log esperado:**
    ```
    2026-02-13 03:00:01 - Iniciando backup
    2026-02-13 03:00:15 - Backup OK: 2.3M
    2026-02-13 03:00:15 - Eliminados 2 backups antiguos
    2026-02-13 03:00:15 - Backups actuales: 5
    2026-02-13 03:00:15 - Espacio usado: 11.5M
    ```

    ??? tip "Pistas"
        ```bash
        # Registrar en log
        echo "$(date '+%Y-%m-%d %H:%M:%S') - mensaje" >> "$LOG"

        # Eliminar antiguos
        find "$DESTINO" -name "backup_*.tar.gz" -mtime +5 -delete

        # Contar backups
        total=$(ls -1 "$DESTINO"/backup_*.tar.gz 2>/dev/null | wc -l)
        ```

## Resumen de Comandos

!!! example "rsync"
    ```bash
    rsync -av origen/ destino/           # Sincronizar
    rsync -avn origen/ destino/          # Simular (dry-run)
    rsync -av --progress origen/ dest/   # Con progreso
    rsync -av --delete origen/ dest/     # Espejo (elimina en destino)
    rsync -av --exclude='*.tmp' or/ de/  # Excluir archivos
    ```

!!! example "cron"
    ```bash
    crontab -e         # Editar crontab
    crontab -l         # Ver crontab
    crontab -r         # Borrar crontab
    ```

!!! example "Sintaxis cron"
    ```bash
    */5 * * * *        # Cada 5 minutos
    0 * * * *          # Cada hora
    0 2 * * *          # Diario 2 AM
    0 3 * * 0          # Domingos 3 AM
    0 0 1 * *          # Día 1 de mes 00:00
    0 8 * * 1-5        # Lunes a viernes 8 AM
    ```

## Tareas Extra

!!! question "Tarea 1: Doble Backup"

    Crear un sistema que:
    1. Haga backup con tar cada noche
    2. Sincronice con rsync cada 6 horas
    3. Ambos registren en logs separados
    4. Programar ambos con cron

!!! question "Tarea 2: Monitor de Sistema"

    Crear un script `monitor.sh` que:
    1. Verifique espacio en disco
    2. Si está > 80%, registre alerta en log
    3. Programarlo para ejecutarse cada hora

    ??? tip "Pista"
        ```bash
        #!/bin/bash
        uso=$(df -h / | tail -n 1 | awk '{print $5}' | sed 's/%//')
        if [ $uso -gt 80 ]; then
            echo "$(date): ALERTA - Disco al $uso%" >> ~/alertas.log
        fi
        ```

## Errores Comunes en cron

!!! warning "1. Usar rutas relativas"
    ```bash
    # ❌ Mal
    0 2 * * * ./backup.sh

    # ✅ Bien  
    0 2 * * * /home/usuario/backup.sh
    ```

!!! warning "2. Olvidar permisos de ejecución"
    ```bash
    # Solución
    chmod +x /home/usuario/script.sh
    ```

!!! warning "3. Script funciona manual pero no con cron"
    **Causa:** Variables de entorno diferentes

    **Solución:** Usar rutas absolutas en TODO el script

!!! warning "4. No redirigir salida"
    ```bash
    # Sin esto, no verás qué pasó
    0 2 * * * /ruta/script.sh >> /ruta/log 2>&1
    ```