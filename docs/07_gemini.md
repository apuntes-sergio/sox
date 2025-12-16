## UD07: Linux Server Esencial y Administración Remota

### Contextualización

Tras haber establecido el Directorio Activo (AD) de la empresa ISCASOX bajo el dominio `DOMXXX` y haber configurado la base de la red con Windows Server, el siguiente paso es integrar un servidor basado en software libre. Este nuevo servidor, denominado provisionalmente `SRVLINUXXXX`, asumirá roles de servidor de archivos y otras funciones de infraestructura. Para que un servidor GNU/Linux pueda gestionar datos de la empresa de manera segura y eficiente, es imprescindible dominar dos aspectos fundamentales: la **gestión flexible del almacenamiento** mediante LVM (Logical Volume Management) y la **aplicación de políticas de acceso complejas** mediante ACLs (Access Control Lists). Esta unidad nos prepara para alojar servicios robustos y para la posterior integración con el entorno Windows, tal como exige la formación del Técnico en Sistemas Microinformáticos y Redes.

---

### 1. Gestión del Sistema de Archivos y LVM

#### 1.1. La Jerarquía del Sistema de Ficheros (FHS) en Servidores

La Jerarquía del Sistema de Ficheros (FHS) es un estándar que organiza el sistema de directorios en GNU/Linux, estableciendo dónde se deben encontrar los ficheros y directorios para garantizar la coherencia y la facilidad de administración. En un servidor de red como el que estamos desplegando, es vital comprender las funciones de los directorios clave para poder planificar la gestión de almacenamiento, especialmente en lo relativo a datos que deben crecer o ser respaldados por separado.

| Directorio | Función Clave en Servidor |
| :--- | :--- |
| `/etc` | Contiene los ficheros de configuración estáticos de todo el sistema y de todos los servicios. |
| `/var` | Aloja datos variables, como los ficheros de *log* de los servicios (`/var/log`), el contenido de servidores web y las colas de impresión. Este directorio tiende a crecer y a menudo se aísla en una partición separada. |
| `/home` | Contiene los directorios personales de los usuarios. Aunque en el proyecto ISCASOX los perfiles de usuario se gestionarán mayormente en el AD, este es el lugar para los administradores locales del sistema. |
| `/srv` | Es el directorio recomendado por la FHS para almacenar los datos de los servicios que sirve el sistema. Por ejemplo, los directorios que serán compartidos a través de Samba o NFS. **En nuestro proyecto, esta es la ubicación ideal para los datos de ISCASOX.** |

El administrador debe decidir qué directorios críticos deben residir en volúmenes separados para permitir copias de seguridad selectivas o para evitar que el crecimiento de *logs* (en `/var`) sature el sistema operativo (en `/`).

#### 1.2. LVM: Logical Volume Management

La gestión tradicional de particiones obliga a definir tamaños fijos al instalar el sistema. Si una partición se llena, la única solución es redimensionar el disco, lo cual históricamente requería desmontar la partición y, a menudo, implicaba tiempo de inactividad.

El **Logical Volume Management (LVM)** es una capa de abstracción que se interpone entre el sistema operativo y los discos físicos. El propósito principal de LVM es proporcionar una gestión de almacenamiento flexible que permita al administrador extender o reducir volúmenes de datos **incluso mientras están en uso**, minimizando el tiempo de inactividad.

**Componentes de LVM y su Función Explicativa:**

1.  **Volumen Físico (PV - Physical Volume):** Es la unidad base de LVM. Se trata de un disco duro físico o una partición tradicional que se ha marcado como disponible para LVM. Para crear un PV se utiliza el comando `pvcreate`.
2.  **Grupo de Volúmenes (VG - Volume Group):** Actúa como un *pool* de almacenamiento unificado. Uno o varios PVs se agrupan para formar un VG, creando un espacio total de almacenamiento. Este grupo es la fuente de donde se "extraen" los volúmenes lógicos. El comando para crearlo es `vgcreate`.
3.  **Volumen Lógico (LV - Logical Volume):** Son las particiones lógicas que el sistema operativo reconoce y que se montan. Estos LVs se crean dentro de un VG, y su tamaño puede ser modificado a demanda. Son la clave de la flexibilidad de LVM, ya que el espacio no utilizado del VG se puede asignar a cualquier LV que lo necesite. Se crean con el comando `lvcreate`.

**Pasos para el Redimensionamiento Explicados:**
Si el Volumen Lógico `lv_datasox` necesita más espacio, el administrador solo tiene que:
1.  **Extender el LV:** Utilizar el comando `lvextend` para añadir más espacio del VG al LV.
2.  **Redimensionar el Sistema de Archivos:** El sistema de archivos (ej. ext4) debe ser informado del cambio de tamaño, lo que se realiza con el comando `resize2fs` (o equivalente para otros sistemas de archivos).

Este procedimiento garantiza que el servidor `SRVLINUXXXX` de ISCASOX pueda adaptarse a las necesidades de almacenamiento futuras de la empresa sin requerir una reinstalación o periodos largos de mantenimiento.

---

### 2. ACLs y Administración Segura con SSH

#### 2.1. Listas de Control de Acceso (ACLs)

En el proyecto ISCASOX, los permisos de los recursos compartidos deben ser granulares. El sistema de permisos tradicional de Linux (`chmod`) es insuficiente cuando se necesita otorgar acceso de escritura a un usuario específico y acceso de solo lectura a un grupo distinto, sobre un directorio cuya propiedad pertenece a un tercer usuario de sistema (como podría ser el usuario de Samba).

Las **ACLs (Access Control Lists)** son una capa de permisos más avanzada que permite definir reglas de acceso a nivel de usuario o grupo de forma específica, sobrepasando las limitaciones del modelo tradicional. Su uso es esencial en la administración de servidores para implementar políticas de seguridad detalladas.

**Mecanismos de Control de ACLs:**

1.  **Definición de Permisos (setfacl):** Permite añadir (`-m`) o eliminar (`-x`) reglas específicas. Por ejemplo, podemos indicar que el grupo de **Comercial** (creado en el AD, pero reconocido en Linux, como veremos en la UD08) tenga permisos de solo lectura (`r-x`) sobre una carpeta, mientras que el propietario del directorio mantiene sus permisos originales.
    * Ejemplo: `setfacl -m g:nombre_grupo:r-x /ruta`
2.  **Consulta de Permisos (getfacl):** Este comando muestra tanto los permisos estándar (propietario, grupo, otros) como la lista de reglas de ACLs que se han aplicado al fichero o directorio, incluyendo la información de la máscara.
3.  **La Máscara (mask):** Cuando se aplica una ACL, el permiso efectivo máximo que cualquier usuario o grupo puede tener sobre ese objeto está limitado por la máscara. Si a un grupo le otorgamos `rwx` pero la máscara es `r-x`, el grupo solo podrá leer y ejecutar, no escribir. El administrador debe verificar y gestionar la máscara para asegurar que la intención de seguridad se cumple.

El uso de ACLs en `SRVLINUXXXX` nos permite implementar la política de que el equipo de Desarrollo Web y el de Comercial tengan accesos distintos y simultáneos a un mismo recurso, sin interferir en los permisos del propietario (el administrador de sistema).

#### 2.2. Administración Segura con SSH

El servidor Linux se administra casi exclusivamente de forma remota, generalmente a través de la terminal, utilizando el protocolo **Secure Shell (SSH)**. En un entorno de red profesional, la seguridad de este canal de administración es crítica.

**Autenticación por Claves SSH:**

El método más seguro y recomendado es el uso de pares de claves asimétricas:

* **Clave Pública:** Es la llave que se deposita en el servidor (`SRVLINUXXXX`) y se hace pública.
* **Clave Privada:** Es la llave que solo posee el administrador en su equipo local. Es un fichero que debe estar cifrado y protegido con una frase de paso (`passphrase`).

Cuando el administrador intenta conectarse, el servidor desafía al cliente pidiéndole que demuestre que posee la clave privada, un proceso que es imposible de interceptar o de adivinar mediante fuerza bruta (a diferencia de una contraseña).

**Refuerzo de la Configuración de SSH (*Hardening*):**

Para proteger el servidor `SRVLINUXXXX` de posibles ataques por fuerza bruta o *login* no autorizado, se realizan modificaciones en el fichero de configuración principal `/etc/ssh/sshd_config`:

1.  **Deshabilitar la Autenticación por Contraseña (`PasswordAuthentication no`):** Una vez que la autenticación por clave funciona, se deshabilita el *login* por contraseña, eliminando la vía más común de ataque.
2.  **Restricción de Acceso Root (`PermitRootLogin no`):** Se prohíbe el acceso directo al usuario `root` por SSH. El administrador debe iniciar sesión con una cuenta de usuario normal y luego elevar privilegios con `sudo`, lo que proporciona un mejor registro de auditoría de las acciones críticas.

Mediante estas técnicas, aseguramos que la administración del servidor de ISCASOX sea segura y auditable.

---

### 3. Enunciado de la Práctica Detallada para el Proyecto ISCASOX

#### UD07.01: Instalación y Bases del Servidor Linux ISCASOX

**Módulo Profesional:** Sistemas Informáticos en Red (0224)
**Unidad Didáctica:** UD07 - Linux Server Esencial y Administración Remota
**Duración:** 2 Semanas (12 horas)

**Objetivo General:** Desplegar el servidor de archivos Linux de la empresa ISCASOX, planificar su almacenamiento con LVM y aplicar políticas de permisos avanzados con ACLs, configurando un acceso remoto seguro.

**Contexto del Proyecto (Empresa ISCASOX - Dominio DOMXXX):**
* **Servidor a Configurar:** `SRVLINUXXXX` (VM Linux Server).
* **Recurso a Proteger:** Se requiere un volumen de datos flexible para la información de los proyectos de la empresa.
* **Grupos de Seguridad a Usar (Creados en el AD):**
    * `gDesarrolloXXX` (Grupo de Programadores Web - **Necesitan Control Total**).
    * `gComercialXXX` (Grupo de Vendedores - **Solo Necesitan Lectura**).

**Procedimiento Detallado de las Tareas:**

1.  **Preparación del Servidor y Particionado LVM:**
    * Instale su sistema operativo servidor Linux (se recomienda una distribución LTS sin entorno gráfico, como Debian o Ubuntu Server).
    * Configure la interfaz de red para que tenga una **IP estática coherente** con el esquema de red de `DOMXXX` (ej. 192.168.1.X) y apunte al Windows Server como DNS principal.
    * Añada un segundo disco duro virtual (mínimo 50 GB) a la máquina virtual.
    * **Implemente LVM:** Convierta el segundo disco en un **Volumen Físico (PV)**.
    * **Cree un Grupo de Volúmenes (VG)** llamado `vg_ISCASOX`.
    * **Cree un Volumen Lógico (LV)** de 30 GB llamado `lv_datasox` dentro del VG, formatéelo con `ext4`.
    * **Cree el punto de montaje** `/srv/datasox` y monte allí el LV, asegurando que el montaje se realice automáticamente al reiniciar.

2.  **Gestión de Espacio y Flexibilidad (LVM):**
    * Simule que la carpeta de datos se llena. Utilice los comandos de LVM para **extender el `lv_datasox`** en 10 GB de espacio libre que quede en el VG.
    * Asegure que el sistema de archivos (ext4) refleje el nuevo tamaño sin perder datos.

3.  **Implementación de ACLs para la Empresa ISCASOX:**
    * En el volumen recién creado, **cree la siguiente estructura de directorios** para la empresa: `/srv/datasox/Proyectos/Proyecto_Web`.
    * **Instale** las utilidades necesarias para usar ACLs (`acl`).
    * **Aplique la política de acceso mediante `setfacl`:**
        * Otorgue permisos de **lectura, escritura y ejecución (rwx)** al grupo `gDesarrolloXXX` sobre la carpeta `Proyecto_Web`.
        * Otorgue permisos de **lectura y ejecución (r-x)** al grupo `gComercialXXX` sobre la misma carpeta.
    * **Verifique** los permisos efectivos del directorio y la máscara.

4.  **Refuerzo de la Seguridad Remota (SSH):**
    * Desde su equipo cliente, **genere un par de claves SSH** (clave pública y privada).
    * **Copie la clave pública** del administrador al servidor `SRVLINUXXXX`.
    * **Modifique el fichero de configuración de SSH** (`/etc/ssh/sshd_config`) para:
        * Deshabilitar la autenticación por contraseña (`PasswordAuthentication no`).
        * Deshabilitar el acceso directo como `root` (`PermitRootLogin no`).
    * Reinicie el servicio SSH para aplicar los cambios.
    * **Compruebe** que solo puede acceder con la clave privada.

**Entregable (Memoria Técnica):**

1.  **Plan de Almacenamiento:** Esquema detallado del LVM (salida de `pvs`, `vgs`, `lvs` y `df -h`).
2.  **Comprobación de Flexibilidad:** Capturas de pantalla que muestren la extensión exitosa del volumen lógico.
3.  **Implementación de ACLs:** Salida del comando **`getfacl /srv/datasox/Proyectos/Proyecto_Web`** que confirme las políticas de acceso de `gDesarrolloXXX` y `gComercialXXX`.
4.  **Seguridad SSH:** Captura de la parte modificada de `/etc/ssh/sshd_config` y una captura que demuestre el acceso exitoso con la clave y el fallo al intentar acceder con contraseña.