## UD10: Servicios de Identidad y Almacenamiento Centralizado

### Contextualización

El perfil del administrador de sistemas se completa con la capacidad de integrar servicios diversos y de asegurar la resiliencia del almacenamiento. En la empresa ISCASOX, aunque el AD de Windows es el centro de identidad, esta unidad introduce al técnico en la comprensión de **LDAP** como protocolo base y en la explotación de *appliances* de software libre (como TrueNAS) para cubrir necesidades especializadas de almacenamiento (NAS/SAN).

El cierre de la unidad se centra en la **prueba de resiliencia del sistema de archivos LVM**. La capacidad de recuperar la estructura de almacenamiento del servidor `SRVLINUXXXX` tras un fallo (y restaurar los datos con las copias de seguridad de la UD09) es la métrica definitiva de un plan de continuidad de negocio funcional, cumpliendo con los procedimientos de mantenimiento en el software base.

---

### 1. Servicios de Directorio Abiertos y *Appliances* de Red

#### 1.1. LDAP: El Estándar de Servicios de Directorio

El **Lightweight Directory Access Protocol (LDAP)** es uno de los protocolos más importantes en la infraestructura de red. Su conocimiento es esencial porque es el lenguaje que utiliza el Active Directory (AD) de Windows Server para almacenar y exponer la información de usuarios y recursos. Un servidor LDAP no es necesariamente un controlador de dominio; es, ante todo, un repositorio jerárquico de información.

**Organización Jerárquica de LDAP:**
La información en un directorio LDAP se organiza en una estructura de árbol conocida como **DIT (Directory Information Tree)**, compuesta por:

1.  **Entradas (Entries):** Cada objeto (un usuario, un grupo, un equipo) es una entrada única.
2.  **Atributos (Attributes):** Son las propiedades asociadas a cada entrada (ej. el atributo `cn` para el nombre común, `mail` para el correo electrónico).

El entendimiento de LDAP permite al técnico de ISCASOX integrar aplicaciones de terceros (ej. un *software* de gestión) con la base de datos de usuarios, sin necesidad de interactuar directamente con el complejo AD. Esto se logra usando la herramienta de consulta **`ldapsearch`** para verificar si un usuario existe o a qué grupo pertenece.

#### 1.2. TrueNAS (FreeNAS): Plataforma Especializada en Almacenamiento

El crecimiento de los datos en ISCASOX exige soluciones dedicadas más allá de los recursos compartidos tradicionales de Windows Server y Samba. **TrueNAS (anteriormente FreeNAS)** es un sistema operativo especializado basado en FreeBSD que está diseñado exclusivamente para la gestión de almacenamiento, utilizando el avanzado sistema de archivos **ZFS**.

* **ZFS (Zettabyte File System):** Es la razón principal para usar TrueNAS. ZFS ofrece características que los sistemas de archivos tradicionales no tienen: verificación automática de integridad de datos (*checksumming*), *snapshots* (instantáneas) y gestión de grandes volúmenes de manera sencilla.
* **Tipos de Almacenamiento:**
    * **NAS (Network Attached Storage):** Proporciona almacenamiento a nivel de **archivo** (directorios y carpetas) a través de protocolos como SMB y NFS, ideal para compartir ficheros en red.
    * **SAN (Storage Area Network):** Proporciona almacenamiento a nivel de **bloque** (discos duros virtuales remotos) a través de protocolos como iSCSI o Fibre Channel. Es la opción preferida para alojar bases de datos o máquinas virtuales.

El despliegue de TrueNAS en el proyecto ISCASOX demuestra cómo el administrador puede implementar una solución de alto rendimiento y alta disponibilidad para cubrir las necesidades específicas de almacenamiento de un departamento, como la carpeta del **Taller de Reparación**.

---

### 2. Resiliencia Crítica: El Procedimiento de Recuperación LVM

La resiliencia de nuestro servidor `SRVLINUXXXX` se pone a prueba con la recuperación del volumen de datos de la empresa (`lv_datasox`). Si la estructura de los discos se corrompe (ej. un fallo en el disco que contiene los metadatos de LVM), los datos críticos son inaccesibles, incluso si el disco físico está intacto.

La clave de la recuperación es el uso de los **metadatos de LVM**, que son una pequeña base de datos que describe la estructura de los Volúmenes Físicos (PV), Grupos de Volúmenes (VG) y Volúmenes Lógicos (LV).

#### 2.1. El Rol de los Metadatos y Su Respaldo

LVM realiza automáticamente un *backup* de sus metadatos cada vez que se realiza un cambio estructural. Estos ficheros de texto se almacenan en:
* `/etc/lvm/backup/`: Contiene la copia más reciente de los metadatos.
* `/etc/lvm/archive/`: Contiene las copias históricas, permitiendo revertir cambios estructurales anteriores.

#### 2.2. Flujo de la Recuperación de Desastres LVM

El proceso de recuperación exige una secuencia precisa de comandos de la línea de comandos, que es crítica y no permite errores. La metodología real se centra en restaurar la estructura antes de que el sistema pueda acceder a los datos.

| Paso | Acción de Contingencia | Comando Clave y Función |
| :--- | :--- | :--- |
| **Paso 1: Localización del Fallo** | Se detecta que el volumen crítico no está montado y que los comandos LVM (`lvs`) no lo reconocen. | `vgdisplay` o `vgs` (Muestra la ausencia de la estructura). |
| **Paso 2: Obtención del Backup** | Localizar el fichero de *backup* más reciente y correcto en `/etc/lvm/backup/` para el VG de ISCASOX (`vg_ISCASOX`). | `ls /etc/lvm/backup/` (Listar los backups). |
| **Paso 3: Restauración de Metadatos** | **El paso más crítico.** Se utiliza el fichero de backup para reconstruir la estructura de LVM en los discos. | `vgcfgrestore -f /path/to/backup/file vg_ISCASOX` (Restaura la definición de la estructura). |
| **Paso 4: Activación y Montaje** | Se activa el Volume Group y se monta el Logical Volume restaurado en el punto de montaje original (`/srv/datasox`). | `vgchange -ay vg_ISCASOX` (Activa el volumen). `mount /dev/vg_ISCASOX/lv_datasox /srv/datasox` (Montar el volumen). |
| **Paso 5: Restauración de Archivos** | Una vez que la estructura está accesible, se utiliza la herramienta de copia (`rsync` de la UD09) para restaurar los ficheros perdidos o borrados desde la copia de seguridad. | `rsync -av /mnt/backup/ /srv/datasox/` (Restauración de ficheros). |

Dominar esta secuencia demuestra la capacidad del técnico para gestionar la resiliencia en el nivel más bajo del sistema operativo. 

---

### 3. Enunciado de la Práctica Detallada para el Proyecto ISCASOX

#### UD10.01: Servicios de Identidad, Almacenamiento y Resiliencia Crítica

**Módulo Profesional:** Sistemas Informáticos en Red (0224)
**Unidad Didáctica:** UD10 - Servicios de Identidad y Almacenamiento Centralizado
**Duración:** 2 Semanas (12 horas)

**Objetivo General:** Explorar soluciones de almacenamiento especializado y demostrar, mediante una prueba de contingencia documentada, la capacidad de recuperar el sistema de archivos del servidor Linux (`SRVLINUXXXX`).

**Contexto del Proyecto:**
* **Servidor Central:** `SRVLINUXXXX` (Linux) con volumen de datos `/srv/datasox` (montado con LVM).
* **Almacenamiento Adicional:** Se requiere una solución NAS/SAN.
* **Datos Críticos:** La estructura del volumen `vg_ISCASOX` y sus datos.

**Procedimiento Detallado de las Tareas:**

1.  **Despliegue del Servicio de Almacenamiento Especializado (TrueNAS):**
    * Instale una máquina virtual (VM) con **TrueNAS CORE o FreeNAS** en la red de la empresa ISCASOX.
    * Configure su acceso de red y la interfaz web.
    * **Creación de Pool y Volumen:** Añada discos virtuales a la VM y cree un *Pool* de almacenamiento ZFS y un volumen dentro de él.
    * **Configurar Recurso Compartido:** Configure un recurso compartido de tipo **SMB** (para compatibilidad con Windows/Samba) llamado `Almacenamiento_Taller`.
    * **Integración de Identidad:** Configure TrueNAS para que utilice el Directorio Activo (`DOMXXX.LOCAL`) como fuente de autenticación.
    * **Asignación de Acceso:** Configure el recurso `Almacenamiento_Taller` para que solo el grupo **`gTallerXXX`** (o el grupo más relevante para reparación) del AD pueda acceder.
    * **Verificación:** Desde un cliente Windows o Linux, acceda al recurso con un usuario del grupo y compruebe el acceso.

2.  **Preparación para la Contingencia LVM:**
    * **Localización del Backup:** Identifique y documente la ruta del fichero de *backup* de metadatos más reciente para el VG `vg_ISCASOX` en `/etc/lvm/backup/`.
    * **Copia de Seguridad Adicional:** Realice una copia manual de ese fichero de *backup* y guárdela fuera de `/etc/lvm/` para simular un *backup* externo.

3.  **Simulación y Recuperación de Desastres LVM:**
    * **Simular Fallo Crítico (CLI):** Desmonte el volumen `/srv/datasox`. Ejecute el comando que **elimina la estructura del Volume Group** (`vgremove`). Verifique con `vgs` y `lvs` que la estructura ha desaparecido (¡los datos son inaccesibles!).
    * **Restauración de la Estructura (CLI):** Utilice el comando **`vgcfgrestore`** y el fichero de *backup* del paso 2 para reconstruir el Volume Group.
    * **Verificación de la Estructura:** Active el Volume Group (`vgchange`) y compruebe que el Logical Volume aparece con los mismos parámetros.
    * **Montaje Final:** Vuelva a montar el volumen lógico en `/srv/datasox`.

4.  **Resiliencia Total (Restauración de Archivos):**
    * Confirme que los datos de la empresa están accesibles en `/srv/datasox`.
    * **Simular Borrado:** Borre deliberadamente un fichero importante de la carpeta de la empresa.
    * **Restauración desde Backup:** Utilice la herramienta **`rsync`** (o el script de la UD09) para restaurar **solo ese fichero borrado** desde la copia de seguridad al volumen `/srv/datasox` recién recuperado.

**Entregable (Memoria Técnica):**

1.  **TrueNAS/Servicio Auxiliar:** Capturas de la interfaz web de TrueNAS mostrando el *pool* de almacenamiento, el recurso compartido y la configuración de acceso utilizando el grupo de seguridad de `DOMXXX`.
2.  **Preparación LVM:** Captura de la ruta y el nombre del fichero de *backup* de metadatos utilizado para la restauración.
3.  **Procedimiento de Recuperación:** **Documentación paso a paso de los comandos CLI** que demuestran la **eliminación del VG** y el éxito del comando **`vgcfgrestore`**.
4.  **Resiliencia Comprobada:** Capturas finales que muestren el volumen `/srv/datasox` montado, accesible, y el fichero crítico restaurado de la copia de seguridad.