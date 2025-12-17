---
title: Copias de seguridad
description:  Gestión del dominio en Windows Server. Copias de seguridad
---

## 7. Copia de Seguridad de Datos con Windows Server Backup 💾

La copia de seguridad es el pilar de la **continuidad del negocio** y la **recuperación ante desastres**. La planificación de estas tareas se rige por el **Objetivo de Punto de Recuperación (RPO)**, que define la pérdida máxima de datos aceptable, y el **Objetivo de Tiempo de Recuperación (RTO)**, que establece el tiempo máximo para restaurar el servicio. En Windows Server, la herramienta integrada **Windows Server Backup (WSB)** permite a los administradores crear copias de seguridad completas y parciales del sistema y de los datos.

Windows Server Backup es una **característica** que debe instalarse en el servidor a través del Administrador del Servidor. A diferencia de las herramientas de terceros, WSB está diseñada para realizar copias de seguridad basadas en imágenes a nivel de bloque, lo que facilita una restauración rápida y completa del sistema operativo o del *hardware* (Bare Metal Recovery).

### Instalación y Ejecución de WSB

Windows Server Backup es una herramienta integrada en el sistema operativo, pero no se instala de forma predeterminada. Su acceso y utilización es posible tanto a través de una consola gráfica como mediante la línea de comandos.
    

* **Instalación:** WSB es una **Característica** (*Feature*), no un Rol de servidor. Debe añadirse al sistema operativo utilizando el **Asistente para agregar roles y características** (*Add Roles and Features Wizard*) en el **Administrador del Servidor** antes de que pueda ser utilizada.

<figure markdown="span" align="center">
    ![](./imgs/adicional/wbadmin-01.png){ width="90%" }
    <figcaption>Agregar características de copias de seguridad</figcaption>
</figure>

* **Ejecución Gráfica (GUI):** Una vez instalada, la consola de WSB se puede iniciar de dos maneras principales:
    1.  Desde el **Administrador del Servidor**, navegando al menú **Herramientas** (*Tools*) y seleccionando **Windows Server Backup**.
    2.  Buscando directamente **Windows Server Backup** en el menú de inicio del servidor.
* **Ejecución por Línea de Comandos (CLI):** Para tareas de automatización y la ejecución precisa de copias de seguridad desde *scripts* o el terminal, se utiliza el comando **`wbadmin`**. Esta utilidad permite configurar copias de seguridad programadas, realizar copias manuales y gestionar procesos de restauración de forma eficiente.

### Tipos de Copias de Seguridad

WSB soporta diferentes tipos de copias de seguridad según el objetivo de recuperación:

* **Servidor Completo (*Full Server*):** 💻
    * Crea una imagen de todo el servidor, incluyendo todos los datos, aplicaciones, y el estado del sistema operativo. Es la opción ideal para la recuperación completa ante fallos de *hardware*.
* **Elementos Personalizados (*Custom*):**
    * Permite seleccionar volúmenes específicos, carpetas o archivos.
    * Incluye la opción de **Copia de Seguridad del Estado del Sistema (*System State*)**, que es fundamental para los Controladores de Dominio.
* **Copia de Seguridad del Estado del Sistema (System State):**
    * Contiene la información crítica para la identidad del servidor: la base de datos de Active Directory, el registro del sistema, los archivos de inicio y la configuración de los servicios. Es esencial para restaurar el dominio en caso de fallos lógicos.

### Planificación y Destino de las Copias

La configuración de WSB se centra en la automatización de las copias para cumplir con el RPO.

* **Programación (*Scheduled Backup*):**
    * Permite configurar copias de seguridad que se ejecutan automáticamente a diario o varias veces al día. WSB gestiona el almacenamiento y el ciclo de vida de estas copias.
* **Copia de Seguridad Única (*One-time Backup*):**
    * Utilizada para tareas específicas o antes de realizar cambios importantes en la configuración del sistema.

#### Destino Recomendado

WSB está optimizado para almacenar copias de seguridad en:

* **Discos Duros Locales Dedicados:** El destino más rápido y recomendado.
* **Carpetas Compartidas en Red:** Es una opción viable, pero puede tener limitaciones de rendimiento y debe ser monitoreada.

