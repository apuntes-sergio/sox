---
title: Logs y Control con Redirección
description: Registrar operaciones, redirigir salida y buscar información con grep
---

## 🎯 Objetivos de la Sesión

Al finalizar esta sesión serás capaz de:
- Redirigir salida de comandos a archivos
- Registrar operaciones de scripts en logs
- Distinguir entre salida normal y errores
- Buscar información en logs con grep
- Filtrar y analizar resultados
- Crear scripts con logging profesional

---

## 📝 ¿Por qué son Importantes los Logs?

Los logs son **registros** de lo que hace el sistema o tus scripts. Son fundamentales para:

- **Saber qué pasó**: ¿Se ejecutó el backup?
- **Detectar errores**: ¿Qué falló y por qué?
- **Ver el historial**: ¿Cuándo se hizo el último backup?
- **Depurar problemas**: Buscar la causa de un fallo
- **Auditoría**: Saber quién hizo qué y cuándo

**Sin logs = volar a ciegas**

---

## ➡️ Redirección Básica

### Sobrescribir archivo (>)

```bash
# Borra el contenido anterior y escribe nuevo
echo "Hola" > archivo.txt
```

**Resultado:** `archivo.txt` contiene solo "Hola"

```bash
echo "Hola" > archivo.txt
echo "Adiós" > archivo.txt

# archivo.txt ahora solo contiene "Adiós"
```

### Añadir al final (>>)

```bash
# NO borra el contenido, añade al final
echo "Hola" >> archivo.txt
echo "Adiós" >> archivo.txt

# archivo.txt contiene:
# Hola
# Adiós
```

### ⚠️ Diferencia clave

| Operador | Acción | Uso |
|----------|--------|-----|
| `>` | Sobrescribe | Crear archivo nuevo |
| `>>` | Añade al final | Añadir a archivo existente (logs) |

**Para logs siempre usa `>>`**

---

## 📋 Redirigir Salida de Comandos

### Guardar listado de archivos

```bash
# Guardar listado en archivo
ls -la > listado.txt

# Ver el archivo
cat listado.txt
```

### Guardar salida de comandos útiles

```bash
# Guardar info de disco
df -h > espacio_disco.txt

# Guardar procesos
ps aux > procesos.txt

# Guardar fecha de ejecución
date >> registro.txt
```

---

## 🔴 Redirigir Errores

Hay dos tipos de salida:
- **Salida estándar (stdout)**: Output normal
- **Salida de error (stderr)**: Mensajes de error

### Redirigir solo errores (2>)

```bash
# Solo errores van al archivo
comando 2> errores.txt
```

### Redirigir salida Y errores juntos

```bash
# Método 1: A archivos separados
comando > salida.txt 2> errores.txt

# Método 2: Al mismo archivo
comando > todo.txt 2>&1

# Método 3: Forma corta (igual que método 2)
comando &> todo.txt
```

**Explicación de `2>&1`:**
- `2` = stderr (errores)
- `>&1` = redirigir al mismo sitio que stdout (salida normal)

---

## 📅 Logs con Fecha y Hora

Los logs **siempre** deben tener fecha y hora para saber cuándo pasó algo.

### Formato de fecha para logs

```bash
# Formato: YYYY-MM-DD HH:MM:SS
date '+%Y-%m-%d %H:%M:%S'
# Salida: 2026-02-13 14:30:15
```

### Registrar con fecha

```bash
#!/bin/bash

LOG="registro.log"

echo "$(date '+%Y-%m-%d %H:%M:%S') - Script iniciado" >> "$LOG"

# Hacer operaciones...

echo "$(date '+%Y-%m-%d %H:%M:%S') - Script finalizado" >> "$LOG"
```

**Resultado en registro.log:**
```
2026-02-13 14:30:15 - Script iniciado
2026-02-13 14:30:18 - Script finalizado
```

---

## 📊 Script con Logging Profesional

### Versión básica

```bash
#!/bin/bash
# Script con logging básico

LOG="operacion.log"

# Registrar inicio
echo "$(date '+%Y-%m-%d %H:%M:%S') - Iniciando operación" >> "$LOG"

# Hacer backup
tar -czf backup.tar.gz /home/usuario/Documentos 2>> "$LOG"

# Registrar resultado
if [ $? -eq 0 ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup OK" >> "$LOG"
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup FALLÓ" >> "$LOG"
fi

echo "$(date '+%Y-%m-%d %H:%M:%S') - Operación finalizada" >> "$LOG"
```

### Versión con función de log

```bash
#!/bin/bash
# Script con función de logging

LOG="/var/log/mi_script.log"

# Función para registrar
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG"
}

# Usar la función
log "Script iniciado"

# Hacer operaciones
tar -czf backup.tar.gz /home/usuario/Documentos 2>> "$LOG"

if [ $? -eq 0 ]; then
    log "Backup creado exitosamente"
else
    log "ERROR: Backup falló"
fi

log "Script finalizado"
```

---

## 🔍 grep - Buscar en Archivos

`grep` busca texto en archivos o en la salida de comandos.

### Sintaxis básica

```bash
grep "texto_a_buscar" archivo
```

### Ejemplos básicos

```bash
# Buscar "error" en un log
grep "error" backup.log

# Buscar "OK" en un log
grep "OK" backup.log
```

### Ignorar mayúsculas/minúsculas (-i)

```bash
# Busca error, Error, ERROR, eRRor, etc.
grep -i "error" backup.log
```

### Mostrar número de línea (-n)

```bash
# Muestra en qué línea está cada coincidencia
grep -n "error" backup.log

# Salida:
# 15:2026-02-13 14:30:20 - ERROR: Archivo no encontrado
# 48:2026-02-13 15:45:10 - ERROR: Sin espacio
```

### Contar coincidencias (-c)

```bash
# Cuenta cuántas veces aparece
grep -c "error" backup.log

# Salida: 5
```

### Buscar en múltiples archivos

```bash
# Buscar en todos los .log
grep "error" *.log

# Buscar recursivamente en directorio
grep -r "error" /var/log/
```

---

## 🔗 Combinar Comandos con grep

### Usando pipes (|)

El pipe `|` envía la salida de un comando a otro.

```bash
# Ver solo líneas con ERROR
cat backup.log | grep "ERROR"

# Ver procesos de un usuario
ps aux | grep usuario

# Ver conexiones de red de un puerto
ss -tulpn | grep :445
```

### Ejemplos prácticos

```bash
# Ver errores de hoy
grep "2026-02-13" backup.log | grep -i "error"

# Contar backups exitosos
grep "Backup OK" backup.log | wc -l

# Ver últimos errores
grep -i "error" backup.log | tail -n 5
```

---

## 📊 Analizar Logs

### Ver últimas líneas de un log

```bash
# Últimas 10 líneas
tail backup.log

# Últimas 20 líneas
tail -n 20 backup.log

# Seguir el log en tiempo real
tail -f backup.log
```

### Ver primeras líneas

```bash
# Primeras 10 líneas
head backup.log

# Primeras 5 líneas
head -n 5 backup.log
```

### Filtrar y analizar

```bash
# Ver solo backups del día 13
grep "2026-02-13" backup.log

# Ver errores del último mes de febrero
grep "2026-02" backup.log | grep -i "error"

# Contar operaciones por día
grep "2026-02-13" backup.log | wc -l
```

---

## 💻 Ejemplos Prácticos Completos

### Ejemplo 1: Script de Backup con Log Detallado

```bash
#!/bin/bash
# Backup con logging completo

ORIGEN="/home/$USER/Documentos"
DESTINO="/backups"
FECHA=$(date +%Y%m%d_%H%M%S)
NOMBRE="backup_$FECHA.tar.gz"
LOG="/var/log/backup.log"

# Función de log
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" >> "$LOG"
}

# Inicio
log "INFO" "=========================================="
log "INFO" "Iniciando backup"

# Verificar origen
if [ ! -d "$ORIGEN" ]; then
    log "ERROR" "Origen $ORIGEN no existe"
    exit 1
fi

log "INFO" "Origen verificado: $ORIGEN"

# Verificar espacio
espacio=$(df -BG "$DESTINO" | tail -n 1 | awk '{print $4}' | sed 's/G//')
log "INFO" "Espacio disponible: ${espacio}GB"

if [ $espacio -lt 5 ]; then
    log "ERROR" "Espacio insuficiente: ${espacio}GB"
    exit 1
fi

# Crear backup
log "INFO" "Creando backup: $NOMBRE"
tar -czf "$DESTINO/$NOMBRE" "$ORIGEN" 2>> "$LOG"

# Verificar resultado
if [ $? -eq 0 ]; then
    tamano=$(du -h "$DESTINO/$NOMBRE" | cut -f1)
    log "INFO" "Backup creado exitosamente: $tamano"
else
    log "ERROR" "Fallo al crear backup"
    exit 1
fi

# Limpiar antiguos
log "INFO" "Limpiando backups antiguos..."
eliminados=$(find "$DESTINO" -name "backup_*.tar.gz" -mtime +7 | wc -l)
find "$DESTINO" -name "backup_*.tar.gz" -mtime +7 -delete
log "INFO" "Eliminados $eliminados backups antiguos"

# Resumen
total=$(ls -1 "$DESTINO"/backup_*.tar.gz 2>/dev/null | wc -l)
log "INFO" "Backups actuales: $total"
log "INFO" "Backup completado exitosamente"
log "INFO" "=========================================="
```

---

### Ejemplo 2: Monitor con Alertas

```bash
#!/bin/bash
# Monitor de sistema con logging

LOG="/var/log/monitor.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" >> "$LOG"
}

# Verificar espacio en disco
uso=$(df -h / | tail -n 1 | awk '{print $5}' | sed 's/%//')

if [ $uso -gt 80 ]; then
    log "ALERTA" "Disco al ${uso}% - Poco espacio"
elif [ $uso -gt 60 ]; then
    log "WARNING" "Disco al ${uso}% - Vigilar espacio"
else
    log "INFO" "Disco al ${uso}% - OK"
fi

# Verificar memoria
mem_uso=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')

if [ $mem_uso -gt 90 ]; then
    log "ALERTA" "Memoria al ${mem_uso}% - Crítico"
elif [ $mem_uso -gt 70 ]; then
    log "WARNING" "Memoria al ${mem_uso}% - Alto"
else
    log "INFO" "Memoria al ${mem_uso}% - OK"
fi
```

---

### Ejemplo 3: Análisis de Logs

```bash
#!/bin/bash
# Script para analizar logs de backup

LOG="/var/log/backup.log"

echo "=========================================="
echo "   ANÁLISIS DE LOGS DE BACKUP"
echo "=========================================="
echo

# Total de backups
total=$(grep -c "Backup creado" "$LOG")
echo "Total de backups realizados: $total"

# Backups exitosos
exitosos=$(grep -c "exitosamente" "$LOG")
echo "Backups exitosos: $exitosos"

# Errores
errores=$(grep -c "ERROR" "$LOG")
echo "Errores totales: $errores"

echo
echo "--- Últimos 5 errores ---"
grep "ERROR" "$LOG" | tail -n 5

echo
echo "--- Backups de hoy ---"
fecha_hoy=$(date +%Y-%m-%d)
grep "$fecha_hoy" "$LOG" | grep "Backup creado"

echo
echo "=========================================="
```

---

## 💻 Ejercicios Prácticos

### Ejercicio 1: Logging Básico (EN CLASE - GUIADO)

**Objetivo:** Crear un script que registre sus operaciones en un log.

**Instrucciones:**
1. Crear `operaciones.sh`
2. Crear 3 archivos de prueba
3. Registrar cada operación en `operaciones.log` con fecha

**Solución:**
```bash
#!/bin/bash

LOG="operaciones.log"

# Registrar inicio
echo "$(date '+%Y-%m-%d %H:%M:%S') - Inicio de operaciones" >> "$LOG"

# Crear archivos
for i in {1..3}
do
    touch "archivo$i.txt"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Creado archivo$i.txt" >> "$LOG"
done

# Registrar fin
echo "$(date '+%Y-%m-%d %H:%M:%S') - Operaciones completadas" >> "$LOG"

echo "✓ Operaciones registradas en $LOG"
cat "$LOG"
```

**Probar:**
```bash
chmod +x operaciones.sh
./operaciones.sh
cat operaciones.log
```

---

### Ejercicio 2: Buscar en Logs (EN CLASE - GUIADO)

**Objetivo:** Analizar un log existente con grep.

**Instrucciones:**
1. Usar el log del ejercicio anterior
2. Buscar todas las líneas con "Creado"
3. Contar cuántos archivos se crearon

**Solución:**
```bash
#!/bin/bash
# analizar_log.sh

LOG="operaciones.log"

echo "=========================================="
echo "   ANÁLISIS DEL LOG"
echo "=========================================="

# Buscar operaciones de creación
echo "Archivos creados:"
grep "Creado" "$LOG"

echo
echo "Total de archivos creados:"
grep -c "Creado" "$LOG"

echo
echo "Fecha del primer archivo:"
grep "Creado" "$LOG" | head -n 1 | cut -d' ' -f1,2

echo
echo "Fecha del último archivo:"
grep "Creado" "$LOG" | tail -n 1 | cut -d' ' -f1,2

echo "=========================================="
```

---

### Ejercicio 3: Sistema de Backup con Logging Completo (PARA ENTREGAR)

**Objetivo:** Crear un sistema de backup con logging profesional.

**Requisitos:**
1. Script llamado `backup_logging.sh`
2. Hacer backup de `~/Documentos`
3. Log en `~/logs/backup.log` con:
   - Fecha y hora de cada operación
   - Nivel de mensaje: [INFO], [ERROR], [WARNING]
   - Verificación de espacio
   - Tamaño del backup
   - Número de backups eliminados
   - Resumen final
4. Función `log()` para registrar mensajes

**Estructura esperada:**
```bash
#!/bin/bash
# Sistema de backup con logging profesional

ORIGEN="$HOME/Documentos"
DESTINO="$HOME/backups"
FECHA=$(date +%Y%m%d_%H%M%S)
LOG="$HOME/logs/backup.log"

# TODO: Crear función log()
# TODO: Registrar inicio
# TODO: Verificar origen existe
# TODO: Verificar espacio (mínimo 2GB)
# TODO: Crear backup
# TODO: Verificar resultado
# TODO: Registrar tamaño
# TODO: Limpiar backups antiguos (>5 días)
# TODO: Registrar resumen
```

**Ejemplo de log esperado:**
```
2026-02-13 14:30:15 [INFO] ==========================================
2026-02-13 14:30:15 [INFO] Iniciando backup
2026-02-13 14:30:15 [INFO] Origen verificado: /home/usuario/Documentos
2026-02-13 14:30:15 [INFO] Espacio disponible: 15GB
2026-02-13 14:30:15 [INFO] Creando backup: backup_20260213_143015.tar.gz
2026-02-13 14:30:18 [INFO] Backup creado exitosamente: 2.3M
2026-02-13 14:30:18 [INFO] Limpiando backups antiguos...
2026-02-13 14:30:18 [INFO] Eliminados 2 backups antiguos
2026-02-13 14:30:18 [INFO] Backups actuales: 4
2026-02-13 14:30:18 [INFO] Backup completado exitosamente
2026-02-13 14:30:18 [INFO] ==========================================
```

**Crear también un script de análisis:**
```bash
#!/bin/bash
# analizar_backups.sh

LOG="$HOME/logs/backup.log"

echo "Total de backups: $(grep -c "Backup creado" "$LOG")"
echo "Errores: $(grep -c "\[ERROR\]" "$LOG")"
echo ""
echo "Últimos 5 backups:"
grep "Backup creado exitosamente" "$LOG" | tail -n 5
```

---

## 📝 Resumen de Comandos

### Redirección
```bash
comando > archivo          # Sobrescribir
comando >> archivo         # Añadir al final
comando 2> errores         # Solo errores
comando &> todo            # Salida + errores
comando >> log 2>&1        # Todo al mismo archivo
```

### grep
```bash
grep "texto" archivo       # Buscar
grep -i "texto" archivo    # Ignorar mayúsculas
grep -n "texto" archivo    # Con número de línea
grep -c "texto" archivo    # Contar coincidencias
grep -r "texto" dir/       # Buscar recursivo
grep -v "texto" archivo    # Invertir (no contiene)
```

### Analizar logs
```bash
tail archivo               # Últimas líneas
tail -f archivo           # Seguir en tiempo real
head archivo              # Primeras líneas
cat archivo | grep txt    # Filtrar contenido
```

### Fecha para logs
```bash
date '+%Y-%m-%d %H:%M:%S'  # 2026-02-13 14:30:15
```

---

## 🏠 Tarea para Casa

### Tarea 1: Monitor de Servicios

Crear un script `monitor_servicios.sh` que:
1. Verifique si están activos: `ssh`, `cron`
2. Registre el estado en `~/logs/servicios.log`
3. Use niveles: [OK], [ERROR]
4. Incluya fecha y hora

```bash
#!/bin/bash
LOG="$HOME/logs/servicios.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" >> "$LOG"
}

if systemctl is-active --quiet ssh; then
    log "OK" "SSH activo"
else
    log "ERROR" "SSH inactivo"
fi

# Repetir para cron...
```

### Tarea 2: Analizador de Logs Avanzado

Crear un script `analisis_completo.sh` que analice `backup.log` y muestre:
1. Total de backups
2. Backups exitosos vs fallidos
3. Día con más backups
4. Tamaño total de backups
5. Promedio de tamaño por backup

---

## ✅ Checklist de la Sesión

- [ ] Redirigir salida con `>`
- [ ] Añadir a archivo con `>>`
- [ ] Redirigir errores con `2>`
- [ ] Redirigir todo con `&>`
- [ ] Registrar con fecha y hora
- [ ] Crear función de logging
- [ ] Usar niveles de log (INFO, ERROR, WARNING)
- [ ] Buscar con `grep`
- [ ] Usar `grep -i`, `-n`, `-c`
- [ ] Combinar comandos con `|`
- [ ] Analizar logs con `tail` y `head`

---

## 💡 Buenas Prácticas de Logging

### 1. Siempre incluir fecha y hora
```bash
echo "$(date '+%Y-%m-%d %H:%M:%S') - mensaje" >> log
```

### 2. Usar niveles de severidad
```bash
[INFO] - Información normal
[WARNING] - Advertencia
[ERROR] - Error grave
[ALERTA] - Situación crítica
```

### 3. Crear función de log reutilizable
```bash
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" >> "$LOG"
}
```

### 4. Separar logs por tipo
```bash
LOG_BACKUP="/var/log/backup.log"
LOG_SISTEMA="/var/log/sistema.log"
LOG_ERRORES="/var/log/errores.log"
```

### 5. Rotar logs grandes
```bash
# Si el log pasa de 10MB, rotarlo
if [ $(stat -f%z "$LOG" 2>/dev/null || stat -c%s "$LOG") -gt 10485760 ]; then
    mv "$LOG" "$LOG.old"
fi
```

### 6. Registrar inicio y fin
```bash
log "INFO" "=========================================="
log "INFO" "Script iniciado"
# ... operaciones
log "INFO" "Script finalizado"
log "INFO" "=========================================="
```

### 7. Registrar todo lo importante
- Inicio/fin de operaciones
- Verificaciones previas
- Resultados de comandos críticos
- Errores con contexto
- Estadísticas finales

---

## 🎯 Preparación para el Proyecto Final

Con esta sesión completamos todas las herramientas necesarias para el proyecto final:

✅ Variables y parámetros  
✅ Condicionales  
✅ Bucles  
✅ tar  
✅ rsync  
✅ cron  
✅ **Logging profesional**  

**Próximas sesiones:**
- Sesión 7-8: Proyecto integrador (sistema de backup automatizado)
- Sesión 9: Examen

**El proyecto combinará TODO lo aprendido en un sistema completo de backup con:**
- Scripts con logging
- Backup con tar
- Sincronización con rsync
- Automatización con cron
- Análisis de logs

---

## 🔍 Comandos Útiles Adicionales

### Filtrar logs por fecha
```bash
# Ver logs de hoy
grep "$(date +%Y-%m-%d)" backup.log

# Ver logs de febrero
grep "2026-02" backup.log
```

### Estadísticas de logs
```bash
# Operaciones por hora
grep "14:" backup.log | wc -l

# Días diferentes en el log
cut -d' ' -f1 backup.log | sort -u | wc -l
```

### Buscar patrones específicos
```bash
# IPs en logs
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log

# Tamaños en logs
grep -oE '[0-9]+M|[0-9]+G' backup.log
```