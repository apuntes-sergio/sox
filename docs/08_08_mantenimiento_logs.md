---
title: Mantenimiento y control - Monitorización y logs
description:  Redes híbridas. Linux Server e integración básica con Windows. 
---

Bloque 5: Mantenimiento y control - Monitorización y logs

Finalizaremos con las herramientas de diagnóstico y resolución de problemas:

Gestión de logs con journald: Aprenderemos a usar journalctl para consultar y analizar los registros del sistema, diagnosticando problemas y monitorizando la actividad.

Monitorización del sistema: Utilizaremos herramientas como htop, iotop, netstat y comandos de análisis de rendimiento para vigilar el estado del servidor.

Análisis de espacio en disco: Herramientas como df, du y ncdu para controlar el uso del almacenamiento.

Resolución de incidencias: Metodología para diagnosticar y resolver problemas comunes en servidores Linux.


---

---
title: Mantenimiento y control - Monitorización y logs
description: Herramientas de diagnóstico, análisis de logs y resolución de problemas en Ubuntu Server
---

## Introducción

El mantenimiento adecuado de un servidor Linux requiere habilidades de monitorización, análisis de logs y diagnóstico de problemas. En esta unidad aprenderemos a usar las herramientas más importantes para:

- **Consultar y analizar logs del sistema** con `journalctl`
- **Monitorizar recursos en tiempo real** (CPU, memoria, red, disco)
- **Analizar el uso del espacio en disco**
- **Diagnosticar y resolver problemas** comunes en servidores

Un administrador competente debe ser capaz de:

1. Detectar problemas antes de que se vuelvan críticos
2. Investigar la causa raíz de fallos o rendimiento degradado
3. Interpretar logs y mensajes de error del sistema
4. Tomar medidas correctivas basadas en datos objetivos

!!!warning "Importancia de la monitorización"
    
    La monitorización proactiva puede prevenir:
    
    - Caídas del servidor por falta de recursos
    - Pérdida de datos por discos llenos
    - Violaciones de seguridad no detectadas
    - Degradación del rendimiento que afecta a usuarios
    
    Un servidor sin monitorización es un servidor en riesgo.

## Gestión de logs con journald

### ¿Qué es journald?

**systemd-journald** es el sistema de gestión de logs de `systemd`. Reemplaza al tradicional `syslog` y ofrece ventajas importantes:

- Almacenamiento binario estructurado (más eficiente que texto plano)
- Indexado para búsquedas rápidas
- Integración nativa con `systemd`
- Capacidad de filtrado avanzado
- Rotación automática de logs

Los logs se almacenan en `/var/log/journal/` (persistentes) o `/run/log/journal/` (volátiles, se pierden al reiniciar).

### Comando journalctl - Fundamentos

`journalctl` es la herramienta para consultar y analizar los logs del sistema.

#### Ver todos los logs

```bash
# Ver todos los logs (puede ser muy largo)
sudo journalctl

# Navegar:
# - Espacio: avanzar página
# - b: retroceder página
# - q: salir
# - /: buscar texto
# - n: siguiente coincidencia
# - N: coincidencia anterior
```

#### Ver logs en tiempo real

```bash
# Seguir los logs en tiempo real (como tail -f)
sudo journalctl -f

# Mostrar las últimas 20 líneas y seguir
sudo journalctl -n 20 -f
```

!!!tip "Equivalente a tail -f"
    
    `journalctl -f` es el equivalente de `tail -f /var/log/syslog` en sistemas tradicionales.

#### Ver logs desde el último arranque

```bash
# Solo logs desde el último arranque
sudo journalctl -b

# Logs del arranque anterior
sudo journalctl -b -1

# Logs de hace 2 arranques
sudo journalctl -b -2

# Listar todos los arranques registrados
sudo journalctl --list-boots
```

#### Filtrar por tiempo

```bash
# Logs desde hace 1 hora
sudo journalctl --since "1 hour ago"

# Logs desde hace 2 días
sudo journalctl --since "2 days ago"

# Logs de hoy
sudo journalctl --since today

# Logs de ayer
sudo journalctl --since yesterday --until today

# Logs entre dos fechas específicas
sudo journalctl --since "2026-02-01 00:00:00" --until "2026-02-03 23:59:59"

# Combinar con tiempo relativo
sudo journalctl --since "2026-02-01" --until "3 days ago"
```

#### Filtrar por servicio

```bash
# Ver logs de un servicio específico
sudo journalctl -u smbd

# Varios servicios a la vez
sudo journalctl -u smbd -u nmbd

# Logs de SSH
sudo journalctl -u ssh

# Logs de red (NetworkManager o systemd-networkd)
sudo journalctl -u NetworkManager
sudo journalctl -u systemd-networkd
```

!!!example "Ejemplo práctico: Diagnosticar problema de Samba"
    
    ```bash
    # Ver logs de Samba de la última hora
    sudo journalctl -u smbd --since "1 hour ago"
    
    # Seguir logs de Samba en tiempo real mientras pruebas conexión
    sudo journalctl -u smbd -u nmbd -f
    ```

#### Filtrar por prioridad

Los logs tienen niveles de prioridad (de menor a mayor gravedad):

- **0: emerg** - Sistema inutilizable
- **1: alert** - Se requiere acción inmediata
- **2: crit** - Condiciones críticas
- **3: err** - Errores
- **4: warning** - Advertencias
- **5: notice** - Eventos normales pero significativos
- **6: info** - Información
- **7: debug** - Mensajes de depuración

```bash
# Solo errores y niveles superiores (err, crit, alert, emerg)
sudo journalctl -p err

# Solo advertencias y superiores
sudo journalctl -p warning

# Errores de un servicio específico
sudo journalctl -u smbd -p err
```

!!!tip "Diagnosticar problemas rápidamente"
    
    ```bash
    # Ver todos los errores desde el último arranque
    sudo journalctl -b -p err
    
    # Ver errores de las últimas 24 horas
    sudo journalctl --since "24 hours ago" -p err
    ```

#### Filtrar por proceso o PID

```bash
# Logs de un ejecutable específico
sudo journalctl /usr/sbin/smbd

# Logs de un proceso por su PID
sudo journalctl _PID=1234

# Logs del kernel
sudo journalctl -k
# Equivalente a:
sudo journalctl _TRANSPORT=kernel
```

#### Búsqueda y grep

```bash
# Buscar texto específico (distingue mayúsculas)
sudo journalctl | grep "error"

# Buscar ignorando mayúsculas/minúsculas
sudo journalctl | grep -i "failed"

# Combinar filtros
sudo journalctl -u smbd --since today | grep -i "connection"

# Buscar múltiples patrones
sudo journalctl | grep -E "error|failed|denied"
```

#### Formato de salida

```bash
# Salida JSON (útil para scripts)
sudo journalctl -u smbd -o json

# Salida JSON con formato legible
sudo journalctl -u smbd -o json-pretty

# Solo el mensaje (sin metadatos)
sudo journalctl -u smbd -o cat

# Salida detallada (verbose)
sudo journalctl -u smbd -o verbose

# Formato corto (default)
sudo journalctl -u smbd -o short
```

### Ejemplos prácticos de diagnóstico

#### Diagnosticar fallo de arranque de un servicio

```bash
# Ver errores del servicio
sudo journalctl -u nombre_servicio -p err

# Ver todo el contexto del último intento de arranque
sudo journalctl -u nombre_servicio -b
```

#### Investigar problema de autenticación SSH

```bash
# Ver intentos de login SSH
sudo journalctl -u ssh | grep -i "authentication"

# Ver intentos fallidos
sudo journalctl -u ssh | grep -i "failed"

# Ver desde cuándo hay problemas
sudo journalctl -u ssh --since "1 hour ago" | grep -i "failed"
```

#### Analizar reinicio inesperado del servidor

```bash
# Ver logs del arranque anterior (cuando ocurrió el problema)
sudo journalctl -b -1

# Ver solo mensajes críticos del arranque anterior
sudo journalctl -b -1 -p crit

# Ver qué pasó antes del último shutdown
sudo journalctl -b -1 --until "shutdown"
```

### Gestión del tamaño de los logs

#### Ver espacio usado por los logs

```bash
# Ver cuánto espacio ocupan los logs
sudo journalctl --disk-usage
```

Salida típica:
```
Archived and active journals take up 384.0M in the file system.
```

#### Limpiar logs antiguos

```bash
# Eliminar logs más antiguos de 2 días
sudo journalctl --vacuum-time=2d

# Eliminar logs más antiguos de 1 semana
sudo journalctl --vacuum-time=7d

# Limitar el tamaño total a 500M
sudo journalctl --vacuum-size=500M

# Dejar solo los últimos 10 archivos de log
sudo journalctl --vacuum-files=10
```

!!!warning "Cuidado con la limpieza agresiva"
    
    No elimines logs demasiado recientes. Los logs antiguos pueden ser cruciales para investigar problemas que se manifestaron hace tiempo.

#### Configurar rotación automática

Editar `/etc/systemd/journald.conf`:

```bash
sudo nano /etc/systemd/journald.conf
```

Configuración recomendada:

```ini
[Journal]
# Almacenar en disco (persistente)
Storage=persistent

# Límite de tamaño total
SystemMaxUse=500M

# Tamaño máximo de archivos individuales
SystemMaxFileSize=50M

# Mantener logs de al menos 2 semanas
MaxRetentionSec=2week
```

Aplicar cambios:

```bash
sudo systemctl restart systemd-journald
```

### Exportar logs para análisis

```bash
# Exportar logs de un servicio a archivo
sudo journalctl -u smbd > samba_logs.txt

# Exportar logs de las últimas 24 horas
sudo journalctl --since "24 hours ago" > system_24h.log

# Exportar solo errores
sudo journalctl -p err --since today > errores_hoy.log
```

## Monitorización del sistema en tiempo real

### htop - Monitor interactivo de procesos

`htop` es una versión mejorada de `top` con interfaz visual más amigable.

#### Instalar htop

```bash
sudo apt update
sudo apt install htop
```

#### Usar htop

```bash
# Ejecutar htop
htop
```

<figure markdown="span" align="center">
  ![htop](./imgs/ubuntu/monitor/htop.png){ width="90%"}
  <figcaption>Interfaz de htop mostrando procesos del sistema</figcaption>
</figure>

**Interpretación de la pantalla:**

**Zona superior (barras de recursos):**

- **CPU**: Una barra por cada núcleo/hilo del procesador
  - Verde: procesos normales
  - Rojo: procesos del kernel
  - Azul: procesos de baja prioridad
  - Cyan: procesos de virtualización
  
- **Memoria (Mem)**:
  - Verde: memoria usada
  - Azul: buffers
  - Amarillo: caché
  
- **Swap**: Memoria de intercambio (disco usado como RAM)
  - Si está alto, el sistema necesita más RAM

**Columnas importantes:**

- **PID**: ID del proceso
- **USER**: Usuario que ejecuta el proceso
- **PRI**: Prioridad
- **NI**: Nice (valor de prioridad ajustable)
- **VIRT**: Memoria virtual usada
- **RES**: Memoria RAM física usada
- **SHR**: Memoria compartida
- **S**: Estado (R=ejecutando, S=durmiendo, Z=zombie)
- **CPU%**: Porcentaje de CPU usado
- **MEM%**: Porcentaje de memoria usado
- **TIME+**: Tiempo total de CPU usado
- **Command**: Comando/programa

**Teclas útiles:**

- **F1**: Ayuda
- **F2**: Configuración
- **F3**: Buscar proceso (/)
- **F4**: Filtrar procesos
- **F5**: Vista de árbol (jerarquía de procesos)
- **F6**: Ordenar por columna
- **F9**: Matar proceso (kill)
- **F10**: Salir
- **Espacio**: Marcar proceso
- **U**: Mostrar procesos de un usuario específico
- **t**: Vista de árbol
- **H**: Ocultar/mostrar hilos
- **K**: Enviar señal a proceso
- **I**: Invertir orden
- **s**: Rastrear llamadas del sistema (strace)
- **l**: Ver archivos abiertos (lsof)

!!!example "Casos de uso de htop"
    
    **Encontrar proceso que consume mucha CPU:**
    
    1. Presiona `F6` y selecciona `CPU%` para ordenar por uso de CPU
    2. Los procesos con mayor consumo estarán arriba
    
    **Matar un proceso que no responde:**
    
    1. Busca el proceso con `F3` o navega con las flechas
    2. Presiona `F9` para enviar señal de terminación
    3. Selecciona `SIGTERM` (15) primero (terminación limpia)
    4. Si no funciona, usa `SIGKILL` (9) (terminación forzada)
    
    **Ver qué servicios están consumiendo memoria:**
    
    1. Presiona `F6` y ordena por `MEM%`
    2. Los procesos con mayor uso de RAM estarán arriba

### Comandos de monitorización básica

#### free - Ver uso de memoria

```bash
# Ver uso de memoria en MB
free -m

# Ver en GB
free -g

# Ver en formato legible (KB, MB, GB)
free -h

# Actualizar cada 2 segundos
free -h -s 2
```

Salida típica:
```
              total        used        free      shared  buff/cache   available
Mem:           7.7G        2.1G        3.2G        156M        2.4G        5.2G
Swap:          2.0G          0B        2.0G
```

**Interpretación:**

- **total**: RAM total instalada
- **used**: RAM usada por procesos
- **free**: RAM completamente libre
- **shared**: RAM compartida entre procesos
- **buff/cache**: RAM usada para buffers y caché (se libera automáticamente si se necesita)
- **available**: RAM disponible para nuevas aplicaciones (incluye parte del caché)
- **Swap**: Espacio de intercambio en disco

!!!warning "Cuándo preocuparse por la memoria"
    
    - **Swap usado constantemente**: El sistema está sin RAM y va lento
    - **Available muy bajo**: Poco margen para nuevos procesos
    - **Used cerca del 100%**: Normal si buff/cache es alto, problemático si no

#### uptime - Tiempo de funcionamiento y carga

```bash
uptime
```

Salida típica:
```
16:45:32 up 3 days, 2:15,  2 users,  load average: 0.52, 0.58, 0.59
```

**Interpretación:**

- **16:45:32**: Hora actual
- **up 3 days, 2:15**: El servidor lleva 3 días y 2 horas encendido
- **2 users**: Usuarios conectados
- **load average: 0.52, 0.58, 0.59**: Carga del sistema en 1, 5 y 15 minutos

**¿Qué significa la carga (load average)?**

- Representa el número de procesos esperando CPU
- En un sistema con 4 núcleos:
  - Load < 4: Sistema saludable
  - Load ≈ 4: Sistema al límite
  - Load > 4: Sistema sobrecargado (procesos esperando)

!!!tip "Regla general"
    
    Si tu load average es consistentemente mayor que el número de núcleos de CPU, el sistema está sobrecargado.
    
    Ver número de núcleos:
    ```bash
    nproc
    ```

#### vmstat - Estadísticas de memoria virtual

```bash
# Mostrar estadísticas cada 2 segundos
vmstat 2

# Mostrar 5 muestras con intervalo de 2 segundos
vmstat 2 5
```

Salida típica:
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 3247680 245760 2458624    0    0     5    12   89  156  2  1 97  0  0
```

**Columnas importantes:**

**Procesos:**
- **r**: Procesos ejecutándose o esperando CPU
- **b**: Procesos bloqueados (esperando I/O)

**Memoria:**
- **swpd**: Memoria swap usada
- **free**: Memoria libre
- **buff**: Memoria en buffers
- **cache**: Memoria en caché

**Swap:**
- **si**: Memoria intercambiada desde disco (swap in)
- **so**: Memoria intercambiada a disco (swap out)

**I/O:**
- **bi**: Bloques leídos desde dispositivos (KB/s)
- **bo**: Bloques escritos a dispositivos (KB/s)

**CPU:**
- **us**: Tiempo en modo usuario (%)
- **sy**: Tiempo en modo kernel (%)
- **id**: Tiempo inactivo (%)
- **wa**: Tiempo esperando I/O (%)

!!!warning "Señales de problemas"
    
    - **si/so altos**: Sistema usando swap constantemente (necesita más RAM)
    - **wa alto**: Procesos esperando mucho por disco (disco lento o saturado)
    - **b alto**: Muchos procesos bloqueados esperando I/O

### iotop - Monitor de I/O de disco

`iotop` muestra qué procesos están usando el disco (lectura/escritura).

#### Instalar iotop

```bash
sudo apt install iotop
```

#### Usar iotop

```bash
# Ejecutar iotop
sudo iotop

# Solo mostrar procesos con actividad de I/O
sudo iotop -o

# Modo acumulativo (total desde inicio)
sudo iotop -a
```

<figure markdown="span" align="center">
  ![iotop](./imgs/ubuntu/monitor/iotop.png){ width="90%"}
  <figcaption>iotop mostrando uso de disco por proceso</figcaption>
</figure>

**Columnas importantes:**

- **TID**: Thread ID
- **PRIO**: Prioridad
- **USER**: Usuario
- **DISK READ**: Velocidad de lectura (KB/s o MB/s)
- **DISK WRITE**: Velocidad de escritura
- **SWAPIN**: Porcentaje de tiempo esperando swap
- **IO**: Porcentaje de tiempo esperando I/O
- **COMMAND**: Proceso/comando

**Teclas útiles:**

- **o**: Mostrar solo procesos con I/O activo
- **p**: Alternar entre procesos e hilos
- **a**: Modo acumulativo
- **q**: Salir
- **r**: Invertir orden
- **flechas**: Navegar

!!!example "Caso de uso: Disco lento"
    
    Si el servidor va lento y sospechas del disco:
    
    1. Ejecuta `sudo iotop -o`
    2. Observa qué proceso está escribiendo/leyendo mucho
    3. Decide si es normal o hay que investigar más

### iostat - Estadísticas de CPU y disco

```bash
# Instalar (parte del paquete sysstat)
sudo apt install sysstat

# Mostrar estadísticas de CPU y disco
iostat

# Actualizar cada 2 segundos
iostat 2

# Mostrar solo estadísticas de disco extendidas
iostat -x

# Mostrar en MB/s
iostat -m 2
```

Salida típica (extendida):
```
Device            r/s     w/s     rMB/s     wMB/s   %util
sda              2.14    5.67      0.08      0.15    1.23
```

**Columnas importantes:**

- **r/s**: Lecturas por segundo
- **w/s**: Escrituras por segundo
- **rMB/s**: MB leídos por segundo
- **wMB/s**: MB escritos por segundo
- **%util**: Porcentaje de utilización del disco

!!!warning "Disco saturado"
    
    Si **%util** está consistentemente cerca del 100%, el disco es un cuello de botella.

### Monitorización de red

#### ss - Ver conexiones de red

`ss` es el reemplazo moderno de `netstat`.

```bash
# Ver todas las conexiones TCP
ss -t

# Ver solo conexiones establecidas
ss -t state established

# Ver puertos en escucha
ss -tln

# Ver conexiones con información de procesos
sudo ss -tlnp

# Ver todas las conexiones (TCP + UDP)
ss -tuna
```

**Opciones:**

- **-t**: TCP
- **-u**: UDP
- **-l**: Solo conexiones en escucha (listening)
- **-n**: Mostrar números de puerto (no resolver nombres)
- **-p**: Mostrar proceso asociado
- **-a**: Todas las conexiones

Salida típica de `sudo ss -tlnp`:
```
State    Recv-Q   Send-Q   Local Address:Port   Peer Address:Port   Process
LISTEN   0        50       0.0.0.0:445           0.0.0.0:*           users:(("smbd",pid=1568,fd=30))
LISTEN   0        128      0.0.0.0:22            0.0.0.0:*           users:(("sshd",pid=745,fd=3))
```

!!!example "Verificar si un servicio está escuchando"
    
    ```bash
    # Ver si Samba está escuchando en puerto 445
    sudo ss -tlnp | grep :445
    
    # Ver si SSH está escuchando en puerto 22
    sudo ss -tlnp | grep :22
    
    # Ver todas las conexiones activas a Samba
    sudo ss -tnp | grep smbd
    ```

#### nethogs - Monitor de ancho de banda por proceso

`nethogs` muestra qué procesos están usando la red y cuánto ancho de banda consumen.

```bash
# Instalar
sudo apt install nethogs

# Ejecutar (especificar interfaz de red)
sudo nethogs eth0

# Todas las interfaces
sudo nethogs
```

<figure markdown="span" align="center">
  ![nethogs](./imgs/ubuntu/monitor/nethogs.png){ width="80%"}
  <figcaption>nethogs mostrando uso de red por proceso</figcaption>
</figure>

Muestra en tiempo real:

- Proceso/programa
- Velocidad de envío (sent)
- Velocidad de recepción (received)

#### iftop - Monitor de tráfico de red

```bash
# Instalar
sudo apt install iftop

# Ejecutar
sudo iftop

# Especificar interfaz
sudo iftop -i eth0
```

Muestra conexiones de red en tiempo real con gráficos de barras del ancho de banda.

#### nload - Visualizar tráfico de red

```bash
# Instalar
sudo apt install nload

# Ejecutar
nload

# Especificar interfaz
nload eth0
```

Muestra gráficos ASCII del tráfico entrante y saliente en tiempo real.

## Análisis de espacio en disco

### df - Ver uso de sistemas de archivos

```bash
# Ver uso de disco en formato legible
df -h

# Ver uso en MB
df -m

# Ver solo sistemas de archivos locales (excluir montajes de red)
df -hl

# Ver información de inodos
df -i
```

Salida típica de `df -h`:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  8.5G   11G  45% /
/dev/sdb1        50G   25G   23G  52% /srv
tmpfs           3.9G  1.2M  3.9G   1% /run
```

**Columnas:**

- **Filesystem**: Dispositivo o sistema de archivos
- **Size**: Tamaño total
- **Used**: Espacio usado
- **Avail**: Espacio disponible
- **Use%**: Porcentaje usado
- **Mounted on**: Punto de montaje

!!!warning "Disco casi lleno"
    
    - **Use% > 80%**: Empezar a preocuparse
    - **Use% > 90%**: Problema inminente
    - **Use% = 100%**: Sistema puede volverse inestable
    
    Algunos servicios (como bases de datos) fallan si no hay espacio en disco.

### du - Analizar uso de espacio por directorio

```bash
# Ver tamaño de un directorio
du -sh /srv/compartido

# Ver tamaño de todos los subdirectorios
du -h /srv/compartido

# Limitar profundidad a 1 nivel
du -h --max-depth=1 /srv

# Ordenar por tamaño (mayor primero)
du -h /srv | sort -rh | head -n 10

# Ver tamaño de directorios en el nivel actual
du -sh *

# Excluir ciertos tipos de archivo
du -sh --exclude='*.log' /var
```

!!!example "Encontrar qué está llenando el disco"
    
    ```bash
    # Ver qué carpetas en /var ocupan más espacio
    sudo du -h --max-depth=1 /var | sort -rh
    
    # Ver los 10 archivos más grandes en /var/log
    sudo find /var/log -type f -exec du -h {} + | sort -rh | head -n 10
    
    # Buscar archivos mayores de 100MB en todo el sistema
    sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
    ```

### ncdu - Navegador interactivo de uso de disco

`ncdu` (NCurses Disk Usage) es una herramienta interactiva muy útil para explorar el uso de disco.

#### Instalar ncdu

```bash
sudo apt install ncdu
```

#### Usar ncdu

```bash
# Analizar directorio actual
ncdu

# Analizar un directorio específico
ncdu /srv

# Analizar sistema completo (puede tardar)
sudo ncdu /
```

<figure markdown="span" align="center">
  ![ncdu](./imgs/ubuntu/monitor/ncdu.png){ width="85%"}
  <figcaption>ncdu mostrando uso de disco de forma interactiva</figcaption>
</figure>

**Navegación:**

- **Flechas arriba/abajo**: Navegar por directorios y archivos
- **Enter**: Entrar en un directorio
- **Flecha izquierda**: Subir un nivel
- **d**: Eliminar archivo/directorio seleccionado (con confirmación)
- **n**: Ordenar por nombre
- **s**: Ordenar por tamaño
- **g**: Mostrar/ocultar gráfico de barras
- **q**: Salir

!!!tip "Uso práctico de ncdu"
    
    `ncdu` es perfecto para investigar qué está ocupando espacio:
    
    1. Ejecuta `sudo ncdu /`
    2. Espera a que termine el análisis
    3. Navega a las carpetas más grandes
    4. Identifica archivos o logs innecesarios
    5. Elimina lo que no necesites (con cuidado)

### Limpiar espacio en disco

#### Limpiar paquetes no necesarios

```bash
# Eliminar paquetes que ya no se necesitan
sudo apt autoremove

# Limpiar caché de paquetes descargados
sudo apt clean

# Limpiar solo paquetes antiguos
sudo apt autoclean
```

#### Limpiar logs del sistema

```bash
# Ver tamaño de logs
sudo journalctl --disk-usage

# Limpiar logs más antiguos de 3 días
sudo journalctl --vacuum-time=3d

# Limitar tamaño total a 500M
sudo journalctl --vacuum-size=500M
```

#### Buscar y eliminar archivos grandes innecesarios

```bash
# Buscar archivos mayores de 500MB
sudo find / -type f -size +500M -exec ls -lh {} \; 2>/dev/null

# Buscar logs antiguos (mayores de 30 días)
sudo find /var/log -type f -name "*.log" -mtime +30

# Eliminar logs comprimidos antiguos
sudo find /var/log -type f -name "*.gz" -mtime +30 -delete

# Buscar archivos core dump (pueden ser grandes)
sudo find / -type f -name "core" -size +100M 2>/dev/null
```

!!!danger "Precaución al eliminar archivos"
    
    Antes de eliminar archivos grandes:
    
    1. Verifica qué son y para qué sirven
    2. Asegúrate de que no son críticos para el sistema
    3. Haz backup si hay dudas
    4. Prueba el sistema después de eliminar

## Resolución de incidencias - Metodología

### Proceso sistemático de diagnóstico

Cuando surge un problema en un servidor, sigue esta metodología:

#### 1. Identificar el síntoma

- ¿Qué está fallando exactamente?
- ¿Cuándo empezó el problema?
- ¿Ha cambiado algo recientemente?
- ¿Afecta a todos los usuarios o solo a algunos?

#### 2. Recopilar información

```bash
# Estado general del sistema
uptime
free -h
df -h

# Revisar logs recientes
sudo journalctl -p err --since "1 hour ago"
sudo journalctl -b -p warning

# Ver procesos problemáticos
htop
```

#### 3. Aislar la causa

- ¿Es un problema de recursos (CPU, RAM, disco)?
- ¿Es un problema de red?
- ¿Es un problema de configuración?
- ¿Es un problema de permisos?

#### 4. Formular hipótesis

Basándote en la información, plantea posibles causas:

- "El disco está lleno, por eso el servicio no puede escribir logs"
- "El servicio no arranca porque hay un error de sintaxis en la configuración"
- "La conexión falla porque el firewall está bloqueando el puerto"

#### 5. Probar la hipótesis

Realiza pruebas para confirmar o descartar:

```bash
# ¿Hay espacio en disco?
df -h

# ¿El servicio está corriendo?
sudo systemctl status nombre_servicio

# ¿Hay errores en la configuración?
testparm  # Para Samba
nginx -t   # Para Nginx
apache2ctl -t  # Para Apache

# ¿El puerto está escuchando?
sudo ss -tlnp | grep :puerto
```

#### 6. Aplicar solución

Una vez identificada la causa, aplica la corrección:

- Liberar espacio en disco
- Corregir configuración
- Reiniciar servicio
- Ajustar permisos

#### 7. Verificar la solución

```bash
# Verificar que el servicio arrancó
sudo systemctl status nombre_servicio

# Ver logs para confirmar que funciona
sudo journalctl -u nombre_servicio -f

# Probar desde un cliente
```

#### 8. Documentar

Registra:

- Qué falló
- Cuándo ocurrió
- Cómo se diagnosticó
- Qué solución se aplicó
- Cómo evitarlo en el futuro

### Problemas comunes y soluciones

#### Servicio no arranca

**Síntoma:** `sudo systemctl start servicio` falla

**Diagnóstico:**

```bash
# Ver estado detallado
sudo systemctl status servicio

# Ver logs del servicio
sudo journalctl -u servicio -n 50

# Ver errores específicos
sudo journalctl -u servicio -p err
```

**Causas comunes:**

1. **Error de configuración**
   ```bash
   # Verificar sintaxis (ejemplo con Samba)
   testparm
   
   # Corregir y reintentar
   sudo systemctl restart smbd
   ```

2. **Puerto ya en uso**
   ```bash
   # Ver qué está usando el puerto
   sudo ss -tlnp | grep :445
   
   # Detener el proceso conflictivo o cambiar puerto
   ```

3. **Permisos incorrectos**
   ```bash
   # Verificar permisos de archivos de configuración
   ls -l /etc/samba/smb.conf
   
   # Corregir permisos
   sudo chmod 644 /etc/samba/smb.conf
   ```

4. **Dependencias no satisfechas**
   ```bash
   # Ver qué servicios necesita
   systemctl list-dependencies servicio
   
   # Iniciar servicios dependientes
   sudo systemctl start servicio_dependiente
   ```

#### Sistema muy lento

**Síntoma:** El servidor responde con mucha latencia

**Diagnóstico:**

```bash
# Ver carga del sistema
uptime

# Ver uso de CPU y memoria
htop

# Ver uso de disco
sudo iotop -o

# Ver si está usando swap
free -h
```

**Causas comunes:**

1. **CPU saturada**
   ```bash
   # En htop, presiona F6 y ordena por CPU%
   # Identifica el proceso problemático
   
   # Si es un proceso legítimo, considera:
   # - Limitar su uso de CPU (nice/renice)
   # - Mejorar hardware (más CPUs)
   # - Optimizar la aplicación
   ```

2. **Memoria insuficiente (usando swap)**
   ```bash
   # Ver uso de swap
   free -h
   
   # Si swap está alto, el sistema necesita más RAM
   # Soluciones:
   # - Cerrar servicios innecesarios
   # - Optimizar configuración de servicios
   # - Añadir más RAM física
   ```

3. **Disco lento**
   ```bash
   # Ver qué proceso está saturando el disco
   sudo iotop -o
   
   # Verificar con iostat
   iostat -x 2
   
   # Soluciones:
   # - Mover operaciones intensivas a horas valle
   # - Usar SSD en lugar de HDD
   # - Optimizar bases de datos (índices, consultas)
   ```

#### Disco lleno

**Síntoma:** Errores de "No space left on device"

**Diagnóstico:**

```bash
# Ver qué filesystem está lleno
df -h

# Encontrar qué está ocupando espacio
sudo ncdu /

# Buscar archivos grandes
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

**Soluciones:**

1. **Limpiar logs**
   ```bash
   # Limpiar logs del sistema
   sudo journalctl --vacuum-size=500M
   
   # Eliminar logs antiguos
   sudo find /var/log -type f -name "*.log" -mtime +30 -delete
   ```

2. **Limpiar paquetes**
   ```bash
   sudo apt autoremove
   sudo apt clean
   ```

3. **Eliminar archivos temporales**
   ```bash
   # Limpiar /tmp (con cuidado)
   sudo rm -rf /tmp/*
   
   # Limpiar cachés de aplicaciones
   ```

4. **Aumentar espacio (LVM)**
   ```bash
   # Si usas LVM, puedes extender volúmenes
   sudo lvextend -L +10G /dev/mapper/ubuntu--vg-lv_datos
   sudo resize2fs /dev/mapper/ubuntu--vg-lv_datos
   ```

#### Problema de red/conectividad

**Síntoma:** No hay conexión a internet o entre máquinas

**Diagnóstico:**

```bash
# Verificar interfaz de red
ip addr

# Ver rutas
ip route

# Verificar DNS
cat /etc/resolv.conf

# Hacer ping al gateway
ping 192.168.100.1

# Hacer ping a internet
ping 8.8.8.8

# Probar DNS
ping google.com
```

**Causas comunes:**

1. **Interfaz de red caída**
   ```bash
   # Ver estado de interfaces
   ip link
   
   # Levantar interfaz
   sudo ip link set eth0 up
   
   # Reiniciar NetworkManager o systemd-networkd
   sudo systemctl restart NetworkManager
   ```

2. **Configuración IP incorrecta**
   ```bash
   # Verificar configuración de Netplan
   cat /etc/netplan/*.yaml
   
   # Aplicar configuración
   sudo netplan apply
   ```

3. **Firewall bloqueando**
   ```bash
   # Ver reglas de firewall
   sudo ufw status verbose
   
   # Permitir tráfico si es necesario
   sudo ufw allow 22/tcp
   ```

4. **Problema de DNS**
   ```bash
   # Si ping a IP funciona pero a nombre no:
   # Verificar /etc/resolv.conf
   cat /etc/resolv.conf
   
   # Agregar DNS manualmente (temporalmente)
   echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
   ```

#### Permisos denegados

**Síntoma:** Errores de "Permission denied"

**Diagnóstico:**

```bash
# Ver permisos del archivo/carpeta
ls -l archivo_o_carpeta

# Ver propietario
ls -ld /srv/compartido/prueba

# Ver qué usuario está ejecutando el proceso
ps aux | grep nombre_proceso
```

**Soluciones:**

1. **Ajustar permisos**
   ```bash
   # Dar permisos de escritura
   sudo chmod 775 /srv/compartido/prueba
   
   # Cambiar propietario
   sudo chown usuario:grupo /srv/compartido/prueba
   
   # Aplicar recursivamente
   sudo chown -R usuario:grupo /srv/compartido/prueba
   sudo chmod -R 775 /srv/compartido/prueba
   ```

2. **Para Samba, verificar force user**
   ```bash
   # En smb.conf, asegúrate de tener:
   force user = usuario_correcto
   
   # Y que la carpeta pertenezca a ese usuario
   ```

### Scripts útiles para monitorización

#### Script de chequeo general del sistema

Crear `/usr/local/bin/check-system.sh`:

```bash
#!/bin/bash

echo "=== ESTADO DEL SISTEMA ==="
echo

echo "Fecha: $(date)"
echo "Uptime: $(uptime -p)"
echo

echo "=== USO DE CPU Y MEMORIA ==="
uptime
free -h
echo

echo "=== USO DE DISCO ==="
df -h | grep -E '^/dev/'
echo

echo "=== SERVICIOS CRÍTICOS ==="
systemctl is-active smbd && echo "Samba: OK" || echo "Samba: FAILED"
systemctl is-active ssh && echo "SSH: OK" || echo "SSH: FAILED"
systemctl is-active systemd-networkd && echo "Network: OK" || echo "Network: FAILED"
echo

echo "=== ERRORES RECIENTES ==="
journalctl -p err --since "1 hour ago" --no-pager | tail -n 10
echo

echo "=== TOP 5 PROCESOS POR CPU ==="
ps aux --sort=-%cpu | head -n 6
echo

echo "=== TOP 5 PROCESOS POR MEMORIA ==="
ps aux --sort=-%mem | head -n 6
```

Hacer ejecutable:

```bash
sudo chmod +x /usr/local/bin/check-system.sh
```

Ejecutar:

```bash
sudo /usr/local/bin/check-system.sh
```

#### Script para alertar si disco está lleno

Crear `/usr/local/bin/check-disk-space.sh`:

```bash
#!/bin/bash

THRESHOLD=80  # Porcentaje de alerta

df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output;
do
  usage=$(echo $output | awk '{ print $1}' | sed 's/%//g')
  partition=$(echo $output | awk '{ print $2 }')
  
  if [ $usage -ge $THRESHOLD ]; then
    echo "ALERTA: La partición \"$partition\" está al $usage%"
  fi
done
```

Hacer ejecutable y programar con cron:

```bash
sudo chmod +x /usr/local/bin/check-disk-space.sh

# Editar crontab
crontab -e

# Agregar línea para ejecutar cada hora
0 * * * * /usr/local/bin/check-disk-space.sh
```

## Herramientas adicionales

### dmesg - Mensajes del kernel

```bash
# Ver mensajes del kernel (hardware, drivers)
dmesg

# Ver mensajes recientes
dmesg | tail -n 50

# Buscar errores
dmesg | grep -i error

# Ver con marcas de tiempo legibles
dmesg -T

# Seguir en tiempo real
dmesg -w
```

Útil para diagnosticar problemas de hardware, discos, red a nivel bajo.

### lsof - Ver archivos abiertos

```bash
# Ver todos los archivos abiertos
sudo lsof

# Ver qué proceso está usando un archivo específico
sudo lsof /srv/compartido/prueba/archivo.txt

# Ver qué está usando un puerto
sudo lsof -i :445

# Ver archivos abiertos por un proceso
sudo lsof -p PID

# Ver archivos abiertos por un usuario
sudo lsof -u nombre_usuario
```

!!!example "Caso de uso: No puedo desmontar un dispositivo"
    
    ```bash
    # Ver qué proceso está usando el dispositivo
    sudo lsof /mnt/disco_externo
    
    # Cerrar los procesos que lo usan
    # Luego desmontar
    sudo umount /mnt/disco_externo
    ```

### strace - Rastrear llamadas del sistema

```bash
# Rastrear un comando
strace ls

# Rastrear un proceso en ejecución
sudo strace -p PID

# Guardar output en archivo
strace -o output.txt comando

# Solo mostrar llamadas de archivo
strace -e trace=file comando
```

Herramienta avanzada para debugging de bajo nivel.

## Resumen de comandos clave

| Tarea | Comando |
|-------|---------|
| Ver logs del sistema | `sudo journalctl` |
| Seguir logs en tiempo real | `sudo journalctl -f` |
| Ver logs de un servicio | `sudo journalctl -u servicio` |
| Ver errores recientes | `sudo journalctl -p err --since "1 hour ago"` |
| Monitor de procesos interactivo | `htop` |
| Ver uso de memoria | `free -h` |
| Ver uso de disco | `df -h` |
| Analizar uso de carpeta | `du -sh /ruta` |
| Navegador de uso de disco | `ncdu /ruta` |
| Monitor de I/O de disco | `sudo iotop -o` |
| Ver conexiones de red | `sudo ss -tlnp` |
| Monitor de ancho de banda | `sudo nethogs` |
| Ver servicios activos | `systemctl list-units --type=service --state=running` |
| Ver carga del sistema | `uptime` |

## Conclusión

La monitorización y el análisis de logs son habilidades fundamentales para cualquier administrador de sistemas. Un buen administrador:

✅ **Monitoriza proactivamente** - No espera a que haya problemas
✅ **Interpreta logs correctamente** - Sabe distinguir mensajes normales de problemas reales  
✅ **Diagnostica sistemáticamente** - Sigue una metodología lógica, no adivina
✅ **Documenta incidencias** - Aprende de los problemas para evitarlos en el futuro
✅ **Automatiza chequeos** - Usa scripts y alertas para detectar problemas temprano

Las herramientas que hemos visto te permitirán:

- Detectar problemas de rendimiento antes de que afecten a usuarios
- Investigar la causa raíz de fallos y errores
- Mantener el servidor saludable y optimizado
- Responder rápidamente cuando surjan incidencias

**Práctica regularmente** con estas herramientas. Cuanto más familiarizado estés con el comportamiento normal de tu servidor, más fácil será detectar anomalías.

!!!tip "Siguientes pasos"
    
    1. Configura alertas automáticas para disco lleno y servicios caídos
    2. Establece una rutina de revisión semanal de logs y recursos
    3. Documenta los valores normales de tu servidor (CPU, RAM, carga)
    4. Crea scripts personalizados para tus necesidades específicas
    5. Aprende a usar herramientas de monitorización más avanzadas (Prometheus, Grafana)