## UD09: Monitorización y Planificación de Tareas

### Contextualización

Una vez que el servidor `SRVLINUXXXX` está instalado, integrado en el dominio `DOMXXX` y sirviendo recursos (UD07 y UD08), el enfoque del administrador pasa del despliegue al **mantenimiento predictivo y correctivo**. Esta unidad se centra en las técnicas para garantizar la disponibilidad, el rendimiento y la integridad de los datos de la empresa ISCASOX.

Dominar la monitorización implica no solo saber qué herramientas usar, sino también cómo interpretar los datos para diagnosticar cuellos de botella (CPU, memoria o disco). La planificación de tareas se aborda mediante **cron** y el **sistema de copias de seguridad robusto** con **rsync**, lo que convierte al servidor Linux en una plataforma de respaldo esencial para toda la infraestructura de la empresa, cumpliendo con el requisito de "ejecutar procedimientos de administración y mantenimiento en el software base".

---

### 1. Diagnóstico de Rendimiento y Análisis de Logs

#### 1.1. Herramientas de Monitorización de Rendimiento

El rendimiento de un servidor se mide por el uso de sus recursos principales: CPU, memoria y subsistema de E/S (Entrada/Salida) de disco. Un administrador debe poder identificar rápidamente qué recurso está saturado para diagnosticar un problema (un cuello de botella).

| Herramienta | Objetivo Principal | Interpretación Clave |
| :--- | :--- | :--- |
| **top / htop** | Visión general en tiempo real de los procesos y el uso de recursos. | **Load Average (Carga Media):** Mide el número de procesos en ejecución o esperando CPU/Disco. Un valor alto (superior al número de núcleos de CPU) indica saturación. |
| **vmstat** | Estadísticas de memoria virtual (VM) y actividad de disco. | **si/so (swap in/out):** Muestran la actividad de paginación. Un valor alto en `si` (swap in) y `so` (swap out) es una señal de que la memoria RAM está saturada y el sistema está usando intensivamente el espacio de intercambio (*swap*), lo que degrada gravemente el rendimiento. |
| **iostat** | Estadísticas detalladas de entrada/salida de disco. | **%util:** Indica el porcentaje de tiempo que el dispositivo está ocupado gestionando peticiones. Un valor cercano al 100% en el disco de datos (`/srv/datasox`) indica un cuello de botella en el subsistema de E/S. |
| **free** | Muestra el uso de la memoria RAM y *swap*. | Diferenciar entre **`used`** (utilizada) y **`free`** (libre) de la memoria, prestando atención al *buffer* y al *cache*, que contienen datos usados recientemente y no son memoria "perdida". |

El administrador debe utilizar estas herramientas para monitorizar el servidor `SRVLINUXXXX` mientras los clientes de ISCASOX acceden a los recursos de Samba, buscando patrones de uso elevado que puedan indicar la necesidad de aumentar recursos (RAM, CPU) o de optimizar el código de alguna aplicación.

#### 1.2. Gestión Centralizada de Logs con `journalctl`

En los sistemas operativos modernos basados en `Systemd` (como las versiones recientes de Ubuntu o Debian), los logs no se almacenan en ficheros de texto tradicionales en `/var/log` de forma dispersa, sino que se centralizan en el **Journal** del sistema.

El comando **`journalctl`** es la herramienta principal para consultar estos logs binarios de manera estructurada y eficiente. Esto es crucial en entornos de producción, ya que permite al administrador filtrar millones de entradas rápidamente.

* **Filtro por Servicio:** `journalctl -u smbd.service`: Muestra solo los logs del servicio Samba, esencial para diagnosticar problemas de acceso o autenticación con el AD.
* **Filtro por Tiempo:** `journalctl --since "2 hours ago"`: Permite revisar solo los eventos recientes tras un fallo.
* **Modo *Follow*:** `journalctl -f`: Muestra los logs en tiempo real (*tail*), útil durante la depuración de un servicio que se acaba de reiniciar.

Esta capacidad de análisis de *logs* permite al administrador del proyecto ISCASOX rastrear un error de autenticación de un usuario del AD, por ejemplo, hasta el momento exacto en que falló el *ticket* de Kerberos.

---

### 2. Automatización con `cron` y Copias de Seguridad Avanzadas con `rsync`

#### 2.1. Programación de Tareas con `cron`

En la administración de sistemas, muchas tareas deben ejecutarse de forma periódica y no supervisada: limpiezas de logs, monitorizaciones de estado, copias de seguridad o reinicios de servicios. El demonio **`cron`** se encarga de ejecutar comandos o *scripts* en momentos específicos definidos por el administrador.

* **`crontab`:** Es la tabla de cron de cada usuario (`crontab -e`). La sintaxis de cada línea es crucial y se compone de seis campos:
    $$\text{Minuto (0-59)} \quad \text{Hora (0-23)} \quad \text{Día del mes (1-31)} \quad \text{Mes (1-12)} \quad \text{Día de la semana (0-7)} \quad \text{Comando a ejecutar}$$
    *Ejemplo:* `0 3 * * 5 /usr/bin/find /tmp -type f -delete` (Elimina ficheros temporales todos los viernes a las 3:00 AM).

Para las tareas del sistema (como los *scripts* de monitorización y *backup* del proyecto ISCASOX), a menudo se utiliza el `crontab` del usuario `root` o los directorios específicos de Systemd (`/etc/cron.daily`, `/etc/cron.weekly`).

#### 2.2. rsync: El Estándar de Copias de Seguridad en Red

Aunque se pueden usar herramientas de copia simples (`cp` o `tar`), **rsync (remote sync)** es la herramienta de copia de seguridad preferida en entornos de red y servidores. Su eficiencia radica en el algoritmo de *delta-transfer*.

* **Transferencia Incremental / Diferencial:** En lugar de copiar todos los archivos, `rsync` compara los archivos de origen y destino y solo transfiere las **diferencias a nivel de bloque** dentro de los archivos modificados. Esto reduce drásticamente el ancho de banda y el tiempo necesarios para las copias de seguridad diarias.
* **Copias Remotas:** `rsync` puede copiar datos directamente a través de SSH. Esto significa que la copia de seguridad de `SRVLINUXXXX` se puede realizar de forma segura a cualquier otro servidor de la red o a la unidad de almacenamiento central sin necesidad de montar recursos de Samba previamente, utilizando la sintaxis:
    $$\text{rsync -avz /origen/ } \text{usuario@destino:/ruta/destino}$$
* **Opciones Comunes:**
    * `-a` (archive): Conserva permisos, propietarios, grupos y marcas de tiempo, crucial para un servidor de archivos.
    * `-v` (verbose): Muestra la actividad.
    * `-z` (compress): Comprime los datos durante la transferencia (útil en red).
    * `--delete`: Elimina del destino los archivos que han sido borrados del origen (mantiene la sincronización espejo).

La implementación de `rsync` garantiza que las copias de seguridad de la información crítica de ISCASOX sean rápidas, seguras (vía SSH) y eficientes, manteniendo la integridad de los metadatos de los archivos (permisos, fechas). 

---

### 3. Enunciado de la Práctica Detallada para el Proyecto ISCASOX

#### UD09.01: Monitorización y Copias de Seguridad con `cron` y `rsync`

**Módulo Profesional:** Sistemas Informáticos en Red (0224)
**Unidad Didáctica:** UD09 - Monitorización y Planificación de Tareas
**Duración:** 2 Semanas (12 horas)

**Objetivo General:** Implementar un sistema de monitorización automatizada y un plan de copias de seguridad incrementales con `rsync` en el servidor `SRVLINUXXXX` para proteger los datos de la empresa.

**Contexto del Proyecto:**
* **Servidor Monitorizado:** `SRVLINUXXXX` (Linux).
* **Datos Críticos a Respaldar:** La carpeta `Datos_Linux` y `Recursos_Jefes` (creadas en UD08) en el volumen `/srv/datasox`.
* **Servicios Críticos a Monitorizar:** El servicio de Samba (`smbd.service`).

**Procedimiento Detallado de las Tareas:**

1.  **Diagnóstico de Rendimiento (Observación):**
    * Utilice las herramientas **`htop`** y **`vmstat`** mientras se realiza una copia intensiva de ficheros grandes a los recursos compartidos de Samba.
    * Identifique y documente los valores críticos (ej. `Load Average` y actividad de *swap*).
    * Utilice `journalctl` para filtrar los logs de Samba (`-u smbd.service`) durante el proceso de copia.

2.  **Automatización de Monitorización con `cron`:**
    * **Crear el Script de Monitorización (Bash):** Cree un *script* llamado `monitor_sistema.sh` que realice dos acciones críticas:
        * Verificar si el servicio `smbd` está activo.
        * Comprobar el porcentaje de uso del volumen `/srv/datasox`.
        * Si el uso de disco supera el 80% o el servicio Samba está inactivo, debe registrar una **alerta** en un fichero de *log* propio (`/var/log/monitor_ISCASOX.log`).
    * **Programar la Tarea:** Edite el `crontab` de *root* para que el *script* `monitor_sistema.sh` se ejecute automáticamente cada hora (`H * * * *`).

3.  **Implementación de Copias de Seguridad Eficientes con `rsync`:**
    * **Crear el Script de Backup:** Cree un *script* llamado `backup_datos_linux.sh`. Este *script* debe utilizar **`rsync`** con las opciones `-avz` para realizar una copia de seguridad de la carpeta `Recursos_Jefes` y `Documentacion_Comun` a una ruta de destino separada (ej. un disco externo simulado en `/mnt/backup`).
    * **Simular un Backup Incremental:** Demuestre la eficiencia de `rsync`. Ejecute el *script* una vez, luego modifique un fichero grande en el origen y vuelva a ejecutarlo. Documente cómo `rsync` solo transfiere los bloques modificados.
    * **Programar el Backup:** Edite el `crontab` para que el *script* de *backup* se ejecute semanalmente (ej. Sábados a las 4:00 AM).

4.  **Desafío Adicional (Integración de Backup Cruzado):**
    * El administrador de ISCASOX necesita respaldar la carpeta **`Empresa`** del Windows Server principal (`\\SRVXXX\Empresa`) desde el servidor Linux. Documente teóricamente o, si es posible, realice la configuración para que el servidor Linux pueda realizar una copia de seguridad remota de esa carpeta Windows utilizando un método de montaje compatible con `rsync` (ej. montando la compartición SMB/CIFS del Windows Server en el Linux, o utilizando la conexión SSH/rsync si el Windows Server tuviera ese servicio).

**Entregable (Memoria Técnica):**

1.  **Diagnóstico:** Interpretación de los datos obtenidos con `htop` y `vmstat` durante el acceso a Samba (identificando el cuello de botella más probable).
2.  **Automatización:** Copia del código del *script* `monitor_sistema.sh` y captura de la entrada en el `crontab` de *root*.
3.  **Backup `rsync`:** Copia del código del *script* `backup_datos_linux.sh`. Capturas de la salida de `rsync` que demuestren que en la segunda ejecución solo se transfieren los cambios (la eficiencia incremental).
4.  **Plan de Contingencia:** Detalle del proceso de *backup* cruzado (Tarea 4).