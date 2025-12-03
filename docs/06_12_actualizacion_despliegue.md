---
title: WSUS & WDS
description:  Gestión del dominio en Windows Server. 
---


## 12. Windows Update Services (WSUS)

Windows Server Update Services (WSUS) es un rol de servidor que permite a los administradores gestionar de forma centralizada la distribución de las actualizaciones y parches de Microsoft a los equipos cliente y servidores de la red local. Su implementación es esencial para mantener los sistemas operativos parcheados de manera uniforme y eficiente, reduciendo los riesgos de seguridad derivados de *software* obsoleto.

### Beneficio Operativo Principal

El principal beneficio de utilizar WSUS radica en el **control de las actualizaciones** y la **optimización del tráfico de red**.

* **Ahorro de Ancho de Banda:** El servidor WSUS descarga las actualizaciones de Microsoft Update **una sola vez**. Los equipos cliente y servidores de la red local las descargan directamente del servidor WSUS. Esto evita que cada máquina individual consuma ancho de banda externo, optimizando significativamente la conexión a Internet.
* **Control y Aprobación:** Los administradores tienen la capacidad de **aprobar o denegar** las actualizaciones y parches. Esto permite probar las actualizaciones en un pequeño grupo de equipos antes de desplegarlas al entorno de producción completo, previniendo fallos críticos.
* **Segmentación:** Las actualizaciones pueden dirigirse a grupos de equipos específicos (*Computer Groups*), permitiendo un despliegue gradual y controlado.

### Funcionamiento y Despliegue en Clientes

El funcionamiento de WSUS se basa en la sincronización y la definición de políticas de destino.

* **Sincronización:** El servidor WSUS se conecta periódicamente a Microsoft Update para obtener las listas de nuevas actualizaciones.
* **Despliegue con Directiva de Grupo (GPO):** La configuración de los equipos clientes para que utilicen el servidor WSUS, en lugar de Microsoft Update, se realiza mediante una Directiva de Grupo (GPO) aplicada a la Unidad Organizativa (OU) que contiene los equipos. Esta GPO define la URL del servidor WSUS y la frecuencia de la búsqueda de actualizaciones.

---

## 13. Windows Deployment Services (WDS)

Windows Deployment Services (WDS) es un rol de servidor que permite a los administradores desplegar sistemas operativos Windows de forma remota a través de la red. Esto elimina la necesidad de utilizar medios físicos (DVD o USB) para cada instalación, permitiendo la instalación centralizada y simultánea de sistemas operativos en múltiples equipos.

### Prerrequisitos de Infraestructura

Para que WDS funcione correctamente, se requiere la existencia y correcta configuración de otros servicios de infraestructura de red:

* **Servicios de Dominio de Active Directory (AD DS):** WDS debe ser un miembro o un Controlador de Dominio para acceder a la configuración de la red.
* **Servidor DNS y DHCP:** El servicio **DHCP** es esencial para asignar direcciones IP a los nuevos clientes que se conectan a la red, y el servicio **DNS** debe estar operativo para la resolución de nombres.
* **Volumen NTFS:** Se requiere un volumen con el sistema de archivos NTFS para almacenar las imágenes de instalación y arranque.

### Componentes de Despliegue

WDS utiliza dos tipos principales de archivos de imagen, ambos en formato **.WIM (Windows Imaging Format)**:

* **Imágenes de Arranque (*Boot Images*):** 💻
    * Son archivos pequeños que el cliente descarga a través de la red (mediante **PXE**) para iniciar un entorno de preinstalación básico. Su única función es preparar el equipo para recibir la imagen de instalación real.
* **Imágenes de Instalación (*Install Images*):** 💾
    * Son los archivos que contienen el sistema operativo completo (Windows 10, Windows 11 o Windows Server) que se instalará en el disco duro del cliente.

### Proceso de Instalación PXE

El despliegue de WDS se basa en el estándar **PXE (Preboot Execution Environment)**:

1.  **Arranque de Red:** El equipo cliente se configura para arrancar desde la red.
2.  **Solicitud PXE:** El cliente realiza una solicitud al servidor DHCP. El servidor DHCP o el servidor WDS (dependiendo de la configuración) dirige al cliente al servidor WDS.
3.  **Descarga de Imagen de Arranque:** El cliente descarga el archivo de imagen de arranque (*Boot Image*) y carga el entorno de preinstalación, donde el usuario puede seleccionar la imagen de sistema operativo deseada para su instalación.

---

Con esto, se completa la documentación de los trece temas clave para la guía del alumno.