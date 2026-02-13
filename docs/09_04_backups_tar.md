---
title: Uso Profesional de tar para Backups
description: Crear, comprimir y gestionar backups con tar
---

`tar` (Tape ARchive) es la herramienta estándar de Linux para crear archivos comprimidos. Se utiliza principalmente para backups porque empaqueta múltiples archivos en uno solo, comprime para ahorrar espacio y preserva permisos y estructura de directorios.

**Usos comunes:**
- Backup de documentos importantes
- Backup de configuraciones del sistema
- Archivar proyectos completos
- Distribuir software

## Comandos Básicos

La sintaxis básica para crear un backup comprimido es:

```bash
tar -czf backup.tar.gz /ruta/a/respaldar
```

**Opciones explicadas:**
- `-c` → crear archivo (create)
- `-z` → comprimir con gzip
- `-f` → especificar nombre del archivo (file)

!!! warning "El orden importa"
    Siempre usa `-czf` para crear o `-xzf` para extraer. El orden de las opciones es importante.

!!! example "Ejemplo básico: Backup de Documentos"

    ```bash
    # Hacer backup de Documentos
    tar -czf backup_documentos.tar.gz /home/usuario/Documentos
    ```

    Esto crea el archivo `backup_documentos.tar.gz` con todo el contenido de Documentos comprimido.

## Añadir Fecha al Nombre

Es fundamental incluir la fecha en el nombre del backup para saber cuándo se creó y mantener múltiples versiones.

**Formatos de fecha útiles:**
```bash
# Formato: AAAAMMDD
fecha=$(date +%Y%m%d)
echo $fecha
# Salida: 20260213

# Formato: AAAAMMDD_HHMMSS
fecha=$(date +%Y%m%d_%H%M%S)
echo $fecha
# Salida: 20260213_143527
```

!!! example "Backup con fecha"

    ```bash
    #!/bin/bash

    fecha=$(date +%Y%m%d)
    tar -czf "backup_$fecha.tar.gz" /home/usuario/Documentos

    echo "Backup creado: backup_$fecha.tar.gz"
    ```

    **Resultado:** `backup_20260213.tar.gz`

## Extraer Backups

Para recuperar los datos de un backup, usamos la opción `-x` (extract):

```bash
# Extraer en directorio actual
tar -xzf backup.tar.gz

# Extraer en ubicación específica
tar -xzf backup.tar.gz -C /ruta/destino/
```

!!! tip "Ver contenido sin extraer"
    Antes de extraer, puedes ver qué contiene el backup:
    ```bash
    tar -tzf backup.tar.gz
    
    # Ver solo los primeros 10 archivos
    tar -tzf backup.tar.gz | head -n 10
    ```

## Excluir Archivos Innecesarios

No todos los archivos deben incluirse en el backup. Hay archivos temporales, cachés y descargas que ocupan mucho espacio sin aportar valor.

**Archivos comunes a excluir:**
- `*.tmp` → Archivos temporales
- `*.cache` → Archivos de caché
- `*~` → Copias de respaldo de editores
- `Descargas` → Carpeta de descargas
- `.cache` → Caché del sistema
- `.thumbnails` → Miniaturas de imágenes

!!! example "Backup con exclusiones"

    ```bash
    #!/bin/bash

    tar -czf backup.tar.gz \
        --exclude='*.tmp' \
        --exclude='*.cache' \
        --exclude='*~' \
        --exclude='Descargas' \
        --exclude='.cache' \
        /home/usuario/
    ```

!!! example "Backup limpio del home"

    ```bash
    #!/bin/bash

    tar -czf backup_limpio.tar.gz \
        --exclude='Descargas' \
        --exclude='.cache' \
        --exclude='.thumbnails' \
        /home/usuario/Documentos
    ```

## Verificar Tamaño y Contenido

Después de crear un backup, es útil verificar su tamaño:

```bash
# Ver tamaño legible
du -h backup.tar.gz
# Salida: 2.3M    backup.tar.gz
```

!!! example "Script con verificación de tamaño"

    ```bash
    #!/bin/bash

    tar -czf backup.tar.gz /home/usuario/Documentos

    tamano=$(du -h backup.tar.gz | cut -f1)
    echo "Backup creado: $tamano"
    ```

!!! example "Comparar tamaño original vs comprimido"

    ```bash
    #!/bin/bash

    # Tamaño original
    original=$(du -sh /home/usuario/Documentos | cut -f1)

    # Crear backup
    tar -czf backup.tar.gz /home/usuario/Documentos

    # Tamaño del backup
    comprimido=$(du -h backup.tar.gz | cut -f1)

    echo "Tamaño original: $original"
    echo "Tamaño comprimido: $comprimido"
    ```

## Verificar Integridad del Backup

Es crucial verificar que el backup se creó correctamente y no está corrupto.

**Método 1: Verificar código de salida**

```bash
#!/bin/bash

tar -czf backup.tar.gz /home/usuario/Documentos

if [ $? -eq 0 ]; then
    echo "✓ Backup creado correctamente"
else
    echo "✗ Error al crear el backup"
fi
```

!!! note "Código de salida"
    La variable `$?` contiene el código de salida del último comando:
    - `0` = éxito
    - Cualquier otro número = error

**Método 2: Intentar listar contenido**

```bash
#!/bin/bash

# Crear backup
tar -czf backup.tar.gz /home/usuario/Documentos

# Verificar integridad
if tar -tzf backup.tar.gz > /dev/null 2>&1; then
    echo "✓ Backup íntegro"
else
    echo "✗ Backup corrupto"
    rm backup.tar.gz
fi
```

## Rotación de Backups Antiguos

Mantener todos los backups ocupa mucho espacio. Es necesario eliminar los antiguos automáticamente.

```bash
# Eliminar backups de más de 7 días
find /backups -name "backup_*.tar.gz" -mtime +7 -delete
```

**Explicación:**
- `find /backups` → buscar en /backups
- `-name "backup_*.tar.gz"` → archivos que coincidan con el patrón
- `-mtime +7` → modificados hace más de 7 días
- `-delete` → eliminarlos

!!! example "Script con rotación"

    ```bash
    #!/bin/bash

    DESTINO="/backups"
    DIAS_RETENCION=7

    echo "Eliminando backups antiguos (>$DIAS_RETENCION días)..."

    # Contar cuántos se eliminarán
    cantidad=$(find "$DESTINO" -name "backup_*.tar.gz" -mtime +$DIAS_RETENCION | wc -l)

    if [ $cantidad -gt 0 ]; then
        find "$DESTINO" -name "backup_*.tar.gz" -mtime +$DIAS_RETENCION -delete
        echo "✓ Eliminados $cantidad backups antiguos"
    else
        echo "○ No hay backups antiguos que eliminar"
    fi
    ```

## Script de Backup Completo

!!! example "Versión básica"

    ```bash
    #!/bin/bash
    # Script de backup básico

    ORIGEN="/home/$USER/Documentos"
    DESTINO="/backups"
    FECHA=$(date +%Y%m%d_%H%M%S)
    NOMBRE="backup_$FECHA.tar.gz"

    # Crear directorio de backups si no existe
    if [ ! -d "$DESTINO" ]; then
        mkdir -p "$DESTINO"
        echo "✓ Directorio de backups creado"
    fi

    # Realizar backup
    echo "Iniciando backup..."
    tar -czf "$DESTINO/$NOMBRE" "$ORIGEN"

    # Verificar resultado
    if [ $? -eq 0 ]; then
        tamano=$(du -h "$DESTINO/$NOMBRE" | cut -f1)
        echo "✓ Backup creado: $NOMBRE"
        echo "  Tamaño: $tamano"
    else
        echo "✗ Error al crear el backup"
        exit 1
    fi
    ```

!!! example "Versión profesional"

    ```bash
    #!/bin/bash
    # Script de backup profesional

    # Configuración
    ORIGEN="/home/$USER/Documentos"
    DESTINO="/backups"
    FECHA=$(date +%Y%m%d_%H%M%S)
    NOMBRE="backup_$FECHA.tar.gz"
    DIAS_RETENCION=7
    LOG="/var/log/backup.log"

    # Función de log
    log_mensaje() {
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG"
    }

    # Inicio
    log_mensaje "Iniciando backup"

    # Verificar que existe el origen
    if [ ! -d "$ORIGEN" ]; then
        log_mensaje "ERROR: $ORIGEN no existe"
        exit 1
    fi

    # Crear destino si no existe
    if [ ! -d "$DESTINO" ]; then
        mkdir -p "$DESTINO"
        log_mensaje "Directorio de backups creado"
    fi

    # Verificar espacio disponible (mínimo 1GB)
    espacio=$(df -BG "$DESTINO" | tail -n 1 | awk '{print $4}' | sed 's/G//')
    if [ $espacio -lt 1 ]; then
        log_mensaje "ERROR: Espacio insuficiente ($espacio GB)"
        exit 1
    fi

    log_mensaje "Espacio disponible: $espacio GB"

    # Crear backup
    log_mensaje "Creando backup: $NOMBRE"

    tar -czf "$DESTINO/$NOMBRE" \
        --exclude='*.tmp' \
        --exclude='*.cache' \
        --exclude='*~' \
        "$ORIGEN" 2>> "$LOG"

    # Verificar resultado
    if [ $? -eq 0 ]; then
        tamano=$(du -h "$DESTINO/$NOMBRE" | cut -f1)
        log_mensaje "OK: Backup creado - $tamano"
        
        # Verificar integridad
        if tar -tzf "$DESTINO/$NOMBRE" > /dev/null 2>&1; then
            log_mensaje "OK: Integridad verificada"
        else
            log_mensaje "ERROR: Backup corrupto"
            rm "$DESTINO/$NOMBRE"
            exit 1
        fi
    else
        log_mensaje "ERROR: Fallo al crear backup"
        exit 1
    fi

    # Rotación de backups antiguos
    log_mensaje "Eliminando backups antiguos (>$DIAS_RETENCION días)"

    cantidad=$(find "$DESTINO" -name "backup_*.tar.gz" -mtime +$DIAS_RETENCION | wc -l)

    if [ $cantidad -gt 0 ]; then
        find "$DESTINO" -name "backup_*.tar.gz" -mtime +$DIAS_RETENCION -delete
        log_mensaje "Eliminados $cantidad backups antiguos"
    else
        log_mensaje "No hay backups antiguos"
    fi

    # Resumen final
    total=$(ls -1 "$DESTINO"/backup_*.tar.gz 2>/dev/null | wc -l)
    espacio_usado=$(du -sh "$DESTINO" | cut -f1)

    log_mensaje "=========================================="
    log_mensaje "RESUMEN:"
    log_mensaje "- Backups totales: $total"
    log_mensaje "- Espacio usado: $espacio_usado"
    log_mensaje "=========================================="
    log_mensaje "Backup completado exitosamente"
    ```

## Ejercicios Prácticos

!!! question "Ejercicio 1: Backup Básico con Fecha"

    **Objetivo:** Crear un script de backup simple con fecha.

    **Instrucciones:**
    1. Crear archivo `backup_simple.sh`
    2. Hacer backup de `~/Documentos`
    3. Guardar en `~/backups/`
    4. Incluir fecha en el nombre

    ??? success "Solución"
        ```bash
        #!/bin/bash

        ORIGEN="$HOME/Documentos"
        DESTINO="$HOME/backups"
        FECHA=$(date +%Y%m%d)
        NOMBRE="backup_$FECHA.tar.gz"

        # Crear directorio si no existe
        mkdir -p "$DESTINO"

        # Crear backup
        echo "Creando backup..."
        tar -czf "$DESTINO/$NOMBRE" "$ORIGEN"

        if [ $? -eq 0 ]; then
            tamano=$(du -h "$DESTINO/$NOMBRE" | cut -f1)
            echo "✓ Backup creado: $NOMBRE"
            echo "  Tamaño: $tamano"
            echo "  Ubicación: $DESTINO/"
        else
            echo "✗ Error al crear backup"
        fi
        ```

    **Probar:**
    ```bash
    chmod +x backup_simple.sh
    ./backup_simple.sh
    ls -lh ~/backups/
    ```

---

!!! question "Ejercicio 2: Backup con Exclusiones"

    **Objetivo:** Crear backup excluyendo archivos innecesarios.

    **Instrucciones:**
    1. Crear archivo `backup_limpio.sh`
    2. Hacer backup de tu home
    3. Excluir: `.cache`, `Descargas`, `*.tmp`
    4. Mostrar tamaño

    ??? success "Solución"
        ```bash
        #!/bin/bash

        ORIGEN="$HOME"
        DESTINO="$HOME/backups"
        FECHA=$(date +%Y%m%d_%H%M%S)
        NOMBRE="backup_home_$FECHA.tar.gz"

        mkdir -p "$DESTINO"

        echo "Creando backup de $ORIGEN..."
        echo "Excluyendo archivos innecesarios..."

        tar -czf "$DESTINO/$NOMBRE" \
            --exclude='.cache' \
            --exclude='Descargas' \
            --exclude='*.tmp' \
            --exclude='backups' \
            "$ORIGEN" 2>/dev/null

        if [ $? -eq 0 ]; then
            tamano=$(du -h "$DESTINO/$NOMBRE" | cut -f1)
            echo "=========================================="
            echo "✓ Backup completado"
            echo "  Archivo: $NOMBRE"
            echo "  Tamaño: $tamano"
            echo "  Ubicación: $DESTINO/"
            echo "=========================================="
        else
            echo "✗ Error al crear backup"
        fi
        ```

---

!!! question "Ejercicio 3: Backup con Rotación (PARA ENTREGAR)"

    **Objetivo:** Crear un sistema de backup que mantenga solo los últimos 7 días.

    **Requisitos:**
    1. El script debe llamarse `backup_rotacion.sh`
    2. Debe hacer backup de `~/Documentos` en `~/backups/`
    3. Debe incluir fecha y hora en el nombre
    4. Debe excluir archivos `.tmp` y `.cache`
    5. Debe eliminar backups de más de 7 días
    6. Debe mostrar:
       - Mensaje de backup creado con tamaño
       - Cuántos backups antiguos se eliminaron
       - Total de backups existentes

    **Estructura esperada:**
    ```bash
    #!/bin/bash
    # Script: backup_rotacion.sh

    # Definir variables (origen, destino, fecha)
    # Crear directorio de backups si no existe
    # Crear backup con exclusiones
    # Verificar que se creó correctamente
    # Mostrar tamaño del backup
    # Eliminar backups antiguos (>7 días)
    # Contar backups eliminados
    # Mostrar resumen final
    ```

    **Ejemplo de salida esperada:**
    ```
    Creando backup...
    ✓ Backup creado: backup_20260213_143527.tar.gz
      Tamaño: 1.2M
      
    Eliminando backups antiguos...
    ✓ Eliminados 3 backups antiguos

    ==========================================
    RESUMEN:
    - Backups actuales: 5
    - Espacio usado: 6.8M
    ==========================================
    ```

    ??? tip "Pistas"
        ```bash
        # Eliminar antiguos
        find "$DESTINO" -name "backup_*.tar.gz" -mtime +7 -delete

        # Contar backups actuales
        total=$(ls -1 "$DESTINO"/backup_*.tar.gz 2>/dev/null | wc -l)

        # Espacio usado
        espacio=$(du -sh "$DESTINO" | cut -f1)
        ```

## Resumen de Comandos

!!! example "Crear backup"
    ```bash
    tar -czf archivo.tar.gz /ruta/          # Crear comprimido
    tar -czf archivo.tar.gz directorio/     # Sin ruta completa
    tar -czf backup.tar.gz dir1/ dir2/      # Múltiples directorios
    ```

!!! example "Extraer backup"
    ```bash
    tar -xzf archivo.tar.gz                 # En directorio actual
    tar -xzf archivo.tar.gz -C /destino/    # En ubicación específica
    ```

!!! example "Listar contenido"
    ```bash
    tar -tzf archivo.tar.gz                 # Listar todo
    tar -tzf archivo.tar.gz | grep archivo  # Buscar archivo específico
    tar -tzf archivo.tar.gz | wc -l         # Contar archivos
    ```

!!! example "Exclusiones"
    ```bash
    tar -czf backup.tar.gz --exclude='*.tmp' dir/
    tar -czf backup.tar.gz --exclude='carpeta' dir/
    ```

!!! example "Verificar"
    ```bash
    # Verificar que se creó
    [ $? -eq 0 ] && echo "OK"

    # Verificar integridad
    tar -tzf backup.tar.gz > /dev/null 2>&1
    ```

## Tareas Extra

!!! question "Tarea 1: Backup Múltiple"

    Crear un script `backup_multiple.sh` que:
    1. Haga backup de 3 directorios diferentes:
       - Documentos
       - Imágenes  
       - Descargas
    2. Cada uno en un archivo separado con fecha
    3. Todos en la carpeta `~/backups/`
    4. Muestre un resumen al final con:
       - Nombre de cada backup
       - Tamaño de cada uno
       - Tamaño total

    **Ejemplo de salida:**
    ```
    ==========================================
    BACKUPS CREADOS:
    ==========================================
    ✓ backup_documentos_20260213.tar.gz - 2.1M
    ✓ backup_imagenes_20260213.tar.gz - 15M
    ✓ backup_descargas_20260213.tar.gz - 8.3M
    ------------------------------------------
    Tamaño total: 25.4M
    ==========================================
    ```

!!! question "Tarea 2: Script de Restauración"

    Crear un script `restaurar.sh` que:
    1. Muestre lista de backups disponibles en `~/backups/`
    2. Pida al usuario que elija uno (por número)
    3. Pregunte dónde restaurar
    4. Extraiga el backup en esa ubicación
    5. Confirme que se restauró correctamente

    ??? tip "Pistas"
        ```bash
        # Listar backups numerados
        ls -1 ~/backups/*.tar.gz | nl

        # Leer elección del usuario
        read -p "Elige backup (número): " numero

        # Obtener archivo seleccionado
        archivo=$(ls -1 ~/backups/*.tar.gz | sed -n "${numero}p")
        ```

## Buenas Prácticas de Backup

!!! tip "1. Siempre incluir fecha en el nombre"
    ```bash
    FECHA=$(date +%Y%m%d_%H%M%S)
    NOMBRE="backup_$FECHA.tar.gz"
    ```

!!! tip "2. Excluir archivos innecesarios"
    ```bash
    --exclude='*.tmp' \
    --exclude='*.cache' \
    --exclude='Descargas'
    ```

!!! tip "3. Verificar espacio antes de crear backup"
    ```bash
    espacio=$(df -BG /backups | tail -n 1 | awk '{print $4}' | sed 's/G//')
    if [ $espacio -lt 5 ]; then
        echo "ERROR: Espacio insuficiente"
        exit 1
    fi
    ```

!!! tip "4. Verificar integridad después de crear"
    ```bash
    if tar -tzf backup.tar.gz > /dev/null 2>&1; then
        echo "Backup verificado"
    else
        echo "Backup corrupto"
        rm backup.tar.gz
    fi
    ```

!!! tip "5. Registrar operaciones en log"
    ```bash
    echo "$(date): Backup creado" >> backup.log
    ```

!!! tip "6. Implementar rotación automática"
    ```bash
    find /backups -name "backup_*.tar.gz" -mtime +7 -delete
    ```

!!! tip "7. Mantener backup en múltiples ubicaciones"
    - Disco local
    - Disco externo
    - Servidor remoto (veremos con rsync)