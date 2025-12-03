---
title: Monitorización Windows Server
description:  Gestión del dominio en Windows Server. Monitorización Windows Server
---


## Monitorización Windows Server

La monitorización es un proceso proactivo y fundamental en la administración de servidores, cuyo objetivo es evaluar la salud, el rendimiento y la seguridad del sistema en tiempo real y a lo largo del tiempo. Permite a los administradores identificar cuellos de botella (*bottlenecks*), prever fallos de *hardware* o *software*, y diagnosticar problemas de rendimiento antes de que afecten a los usuarios. Windows Server integra varias herramientas esenciales para la monitorización detallada.

### Visor de Eventos (*Event Viewer*)

El Visor de Eventos es la herramienta central para analizar la estabilidad del sistema, los fallos de *hardware* y *software*, y las actividades de seguridad. Centraliza todos los mensajes generados por el sistema operativo, las aplicaciones y los servicios.

* **Estructura de Registros:** Los eventos se organizan en registros (*logs*) principales que facilitan la localización de la información:
    * **Sistema:** Registra eventos de los componentes centrales de Windows, como *drivers* y *hardware*.
    * **Aplicación:** Almacena eventos generados por aplicaciones instaladas (por ejemplo, fallos de un servicio).
    * **Seguridad:** Contiene registros de eventos de auditoría (inicios de sesión exitosos o fallidos, cambios en políticas de seguridad, accesos a archivos).
    * **Servicios Específicos:** Existen registros dedicados a Active Directory, DNS, y otros roles instalados.
* **Niveles de Evento:** Cada evento se clasifica según su severidad, lo que ayuda al administrador a priorizar la atención:
    * **Error:** Indica un problema que ha causado la pérdida de funcionalidad o un fallo.
    * **Advertencia (*Warning*):** Sugiere un problema potencial que no es crítico, pero que podría causar un fallo en el futuro.
    * **Información:** Registra la operación exitosa de un programa o servicio (por ejemplo, el inicio de un servicio).
    * **Auditoría (Correcta/Incorrecta):** Se aplica a los eventos de seguridad para rastrear accesos y operaciones.
* **Uso Avanzado:** Los administradores utilizan **Filtros** y **Vistas Personalizadas** para buscar patrones de eventos específicos, diagnosticar fallos repetitivos, o identificar intentos de acceso no autorizados.

### Monitor de Rendimiento (*Performance Monitor*)

El Monitor de Rendimiento es una herramienta de diagnóstico en tiempo real y a largo plazo. Su función es recolectar y presentar datos sobre el uso de recursos del sistema, lo cual es fundamental para el ajuste (*tuning*) del servidor.

* **Contadores de Rendimiento:** La herramienta opera a través de **contadores**, que son métricas específicas asociadas a **objetos** del sistema (como la CPU, la memoria o el disco).
* **Contadores Críticos para Diagnóstico:** Para identificar cuellos de botella de *hardware*, se monitorizan contadores clave:
    * **Procesador:** `% Processor Time` (Uso de CPU). Un valor consistentemente alto indica un cuello de botella en la CPU.
    * **Memoria:** `Available Mbytes` (Memoria disponible). Un valor bajo sugiere la necesidad de más RAM o de identificar fugas de memoria.
    * **Disco Físico:** `% Disk Time` (Tiempo de disco en uso). Un valor alto indica que el disco es un cuello de botella.
    * **Interfaz de Red:** `Bytes Total/sec` (Tráfico total). Se usa para verificar la capacidad de la red.
* **Registros de Datos (*Data Collector Sets - DCS*):** Para la monitorización histórica, el Monitor de Rendimiento permite configurar conjuntos de recolectores de datos que graban las métricas de rendimiento durante periodos largos, facilitando el análisis de tendencias y la identificación de problemas intermitentes.


### Monitor de Recursos (*Resource Monitor*)

El Monitor de Recursos es una utilidad de diagnóstico avanzada que complementa al Administrador de Tareas y al Monitor de Rendimiento. Su función principal es proporcionar una visión inmediata y granular del uso de los recursos del sistema, **vinculando directamente el consumo de CPU, memoria, disco y red al proceso o servicio responsable**. Esto lo convierte en una herramienta esencial para el diagnóstico rápido de problemas de rendimiento en tiempo real.

* **Ejecución:** El Monitor de Recursos se lanza fácilmente desde el **Administrador de Tareas** (pestaña Rendimiento) o buscándolo directamente en el menú de inicio de Windows Server.
* **Propósito:** Permite a los administradores identificar de manera eficiente qué procesos específicos están causando cuellos de botella (por ejemplo, qué archivo está bloqueando la actividad del disco o qué servicio está saturando la CPU).

### Vistas y Métricas Clave

El Monitor de Recursos organiza la información de rendimiento en cinco pestañas principales, cada una ofreciendo un análisis detallado del recurso correspondiente:

| Pestaña | Información Principal que Proporciona | Importancia para el Diagnóstico |
| :--- | :--- | :--- |
| **CPU** | Muestra el uso individual del procesador por cada proceso y servicio activo. Permite suspender procesos que consumen excesivos ciclos de CPU. | Identificar fugas de código o procesos descontrolados que afectan el rendimiento. |
| **Memoria** | Muestra cómo se utiliza la memoria física (bytes en uso, en espera, y libres). Detalla qué procesos están asignando la mayor cantidad de memoria de trabajo (*Working Set*). | Diagnosticar la necesidad de más RAM o identificar aplicaciones con gestión ineficiente de memoria. |
| **Disco** | Muestra la actividad de lectura y escritura (bytes/segundo) de cada volumen. **Lo más valioso:** Indica qué procesos están accediendo a qué archivos específicos en tiempo real. | Determinar qué aplicación está causando latencia o congestión en el subsistema de disco. |
| **Red** | Muestra la actividad de red (envío y recepción) de cada proceso. Detalla qué puertos y direcciones IP están utilizando las conexiones activas. | Identificar procesos que consumen excesivo ancho de banda o detectar actividad de red no deseada. |

La principal ventaja del Monitor de Recursos reside en su capacidad para **filtrar y ordenar** los datos en vivo, permitiendo al administrador aislar y analizar el impacto de un proceso individual en los cuatro recursos fundamentales del servidor de forma simultánea.

## Auditorías Windows Server

La auditoría de seguridad es un proceso fundamental en la administración de Windows Server, cuyo objetivo es rastrear y registrar las acciones de los usuarios y del sistema operativo. Esto permite la **rendición de cuentas**, la **detección de intrusiones** y el análisis forense tras un incidente de seguridad, garantizando el cumplimiento de las políticas internas y normativas externas.

### Configuración de la Directiva de Auditoría

La configuración de las auditorías se establece principalmente a través de la **Directiva de Grupo (Group Policy)**, lo que permite aplicarla de manera centralizada a Controladores de Dominio y servidores miembros específicos.

* **Implementación:** Se utiliza la consola de **Administración de Directiva de Grupo (GPMC)** para editar la directiva de seguridad que aplica a la Unidad Organizativa (OU) que contiene los servidores.
* **Auditoría Avanzada:** Se recomienda utilizar la **Configuración de Directiva de Auditoría Avanzada**, que ofrece un control más granular sobre los eventos a registrar que las categorías de auditoría básicas.

### Categorías de Eventos Clave

Para un entorno de dominio seguro, hay categorías de auditoría que son prioritarias para el rastreo, ya que revelan la actividad de los administradores y los usuarios:

* **Gestión de Cuentas (*Account Management*):**
    * **Propósito:** Rastrea cuando se crean, modifican, eliminan o se realizan cambios de estado (deshabilitar/habilitar) en cuentas de usuario y grupos de seguridad.
    * **Importancia:** Es esencial para monitorizar la actividad de los administradores y detectar la creación de cuentas no autorizadas.
* **Inicio/Cierre de Sesión (*Account Logon/Logoff*):**
    * **Propósito:** Registra los intentos de inicio de sesión en el dominio o en un servidor local.
    * **Importancia:** Ayuda a identificar patrones de actividad inusuales y a detectar ataques de fuerza bruta (registrando los fallos de inicio de sesión).
* **Acceso a Objetos (*Object Access*):**
    * **Propósito:** Rastrea los intentos de acceder, leer, escribir o modificar recursos específicos del sistema (archivos, carpetas, claves de registro).
    * **Configuración:** Requiere la configuración de **Listas de Control de Acceso del Sistema (SACL)** en los objetos individuales que se desean auditar.

### Tipos de Auditoría y Revisión

Al configurar una directiva, se especifica si se debe auditar el éxito, el fallo, o ambos resultados de una acción.

* **Auditoría Correcta (*Success*):** Registra una entrada cuando la acción del usuario o sistema es permitida. Útil para rastrear quién accedió a recursos críticos.
* **Auditoría Incorrecta (*Failure*):** Registra una entrada cuando la acción es denegada. Vital para detectar intentos de acceso no autorizados.
* **Localización de Eventos:** Los resultados de las auditorías se encuentran en el **Visor de Eventos**, específicamente en el **Registro de Seguridad**. El análisis de este registro es la tarea diaria que cierra el ciclo de auditoría.

