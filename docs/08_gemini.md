## UD08: Servidor de Archivos Heterogéneo (Samba)

### Contextualización

La administración de sistemas en red rara vez se limita a un único fabricante o sistema operativo. El currículo exige que el técnico sea competente en la **integración de sistemas operativos en red libres y propietarios**. Dado que la empresa ISCASOX ya utiliza Windows Server con Active Directory (AD) para la gestión de identidades (`DOMXXX`), el reto de esta unidad es convertir el nuevo servidor Linux (`SRVLINUXXXX`) en un miembro funcional de ese dominio.

El protocolo que permite esta integración es **SMB/CIFS (Server Message Block / Common Internet File System)**, y en el mundo del software libre, la implementación estándar de este protocolo es **Samba**. El objetivo no es reemplazar el AD, sino extender su funcionalidad, permitiendo que `SRVLINUXXXX` ofrezca recursos de archivo a todos los clientes (Windows y Ubuntu) y, fundamentalmente, **autenticando a los usuarios contra el Directorio Activo ya existente**, evitando la duplicidad de cuentas de usuario.

---

### 1. Conceptos Fundamentales de la Integración (AD, Kerberos, LDAP)

Para que el servidor Linux y el AD de Windows puedan comunicarse, ambos deben "hablar" los mismos protocolos de seguridad y directorio:

#### 1.1. El Protocolo SMB/CIFS y Samba

* **SMB/CIFS:** Es el protocolo de red que utilizan los sistemas Windows para compartir archivos, impresoras y para la comunicación entre los nodos de red. Históricamente, Microsoft desarrolló diferentes versiones (CIFS fue un término inicial, SMB es el actual).
* **Samba:** Es la implementación de software libre (GNU/Linux) que permite a un sistema operativo que no es Windows entender y participar plenamente en las redes que utilizan el protocolo SMB/CIFS. Samba puede funcionar de tres modos principales:
    1.  **Standalone Server (Servidor Independiente):** Solo utiliza cuentas de usuario locales.
    2.  **Domain Member (Miembro de Dominio):** Se une a un dominio existente (nuestro caso con el AD de ISCASOX), delegando la autenticación al Controlador de Dominio.
    3.  **Domain Controller (Controlador de Dominio):** Actúa como un servidor de directorio (una alternativa al AD). Este modo queda fuera del alcance de esta práctica.

#### 1.2. Protocolos de Autenticación Centralizada

La clave para que Samba funcione como miembro de dominio es el uso de los mismos protocolos que utiliza el AD:

| Protocolo | Función | Rol en la Integración |
| :--- | :--- | :--- |
| **Kerberos** | Proporciona un servicio de autenticación fuerte y centralizada basado en el concepto de "ticket". | El cliente Linux y el servidor Samba lo utilizan para solicitar tickets al AD que demuestren que la identidad del usuario es válida. |
| **LDAP** | Protocolo para consultar y modificar servicios de directorio (donde están almacenadas las cuentas, grupos, y su información). | Samba y los servicios de soporte lo usan para buscar información de pertenencia a grupos (ej. ¿Pertenece el usuario `falonso` al grupo `gJefesXXX`?). |
| **NSS / PAM** | Módulos internos de Linux (Name Service Switch / Pluggable Authentication Modules). | Son las interfaces que le dicen al sistema operativo Linux que **no** debe buscar usuarios en `/etc/passwd` (local), sino a través de Kerberos y LDAP (AD). |

#### 1.3. La Importancia del Mapeo de Identidades (SID y UID/GID)

Cuando un usuario del AD accede a un recurso en el servidor Linux, existe un problema de identificación:

* **Windows (AD):** Identifica a los usuarios por un identificador único llamado **SID (Security Identifier)**.
* **Linux:** Identifica a los usuarios por un identificador numérico llamado **UID (User ID)** y a los grupos por **GID (Group ID)**.

El proceso de unión al dominio instala servicios (comúnmente `winbind` o `sssd`) que se encargan de **mapear** los SIDs del AD a UIDs/GIDs válidos en Linux, asegurando que, cuando el usuario `falonso` inicia sesión en un cliente y accede a un recurso Samba en `SRVLINUXXXX`, el sistema operativo Linux reconozca el archivo como perteneciente al mismo `falonso` (y aplique correctamente los permisos rwx/ACLs). Este mapeo garantiza que **no tenemos que crear usuarios locales en Linux**. 

---

### 2. Configuración de Samba como Miembro de Dominio

#### 2.1. Herramientas y Servicios para la Unión

Aunque se puede configurar manualmente, hoy en día se utilizan herramientas que automatizan gran parte del proceso:

* **SSSD (System Security Services Daemon):** Es el servicio moderno y recomendado en muchas distribuciones. Actúa como un *proxy* o caché para la autenticación, lo que mejora la velocidad y la resiliencia (si el AD cae brevemente, SSSD puede usar su caché). Se encarga de la comunicación con LDAP y Kerberos.
* **Realmd:** Es una herramienta de línea de comandos que simplifica la tarea de unir un host a un dominio Kerberos/LDAP como el AD. Básicamente, configura SSSD, Kerberos y el sistema para usted con un simple comando (`realm join DOMXXX.LOCAL`).
* **Configuración Kerberos (krb5.conf):** El sistema debe conocer la ubicación del KDC (Key Distribution Center), que es su Controlador de Dominio de Windows Server. Este fichero debe apuntar al AD para la autenticación.

#### 2.2. El Fichero de Configuración Central: `smb.conf`

Este archivo es el corazón de Samba. Se divide en secciones globales y de recursos:

* **Sección `[global]`:** Define el comportamiento general de Samba. Los parámetros críticos para la integración del proyecto ISCASOX son:
    * `workgroup = DOMXXX`: Especifica el nombre NETBIOS del dominio.
    * `security = ads`: Indica que Samba debe operar como miembro de un dominio de Active Directory.
    * `realm = DOMXXX.LOCAL`: Especifica el nombre DNS del dominio Kerberos.
    * `idmap config DOMXXX:backend = rid`: Configura el mapeo de SIDs a UIDs/GIDs, indicando a Samba cómo traducir las identidades del AD.

* **Secciones de Recursos (`[nombre_recurso]`):** Definen las carpetas que se comparten.

#### 2.3. Asignación de Permisos de Recurso

Una vez unido el servidor, el control de acceso se realiza con las identidades del AD. Al configurar un recurso compartido, se utilizan los **Grupos de Seguridad del Active Directory** ya creados en el proyecto (ej. `gJefesXXX`, `gAlumnosXXX`) para restringir quién puede acceder.

* **Parámetro `valid users`:** Esta es la línea clave. Permite definir qué usuarios o grupos tienen permiso para acceder a la compartición de Samba.
    * Ejemplo: `valid users = @DOMXXX\gJefesXXX` (la arroba indica que es un grupo del AD).

**IMPORTANTE:** Los permisos de Samba (`valid users`) solo controlan el acceso a la compartición. Una vez dentro, los permisos finales los controlan los permisos del **sistema de archivos Linux** (rwx y ACLs, vistos en la UD07). Ambos deben ser coherentes para evitar problemas de seguridad.

---

### 3. Enunciado de la Práctica Detallada para el Proyecto ISCASOX

#### UD08.01: Servidor de Archivos Samba en el Dominio DOMXXX

**Módulo Profesional:** Sistemas Informáticos en Red (0224)
**Unidad Didáctica:** UD08 - Servidor de Archivos Heterogéneo (Samba)
**Duración:** 2 Semanas (12 horas)

**Objetivo General:** Configurar el servidor Linux (`SRVLINUXXXX`) como miembro del dominio Active Directory de ISCASOX (`DOMXXX`), e implementar recursos compartidos de Samba utilizando los grupos de seguridad del AD para la gestión de accesos.

**Contexto del Proyecto:**
* **Servidor de Identidades:** Windows Server (Controlador de Dominio con AD).
* **Servidor de Recursos:** `SRVLINUXXXX` (Linux) con volumen de datos montado en `/srv/datasox`.
* **Grupos de Seguridad Críticos (AD):**
    * `gJefesXXX` (Requieren Acceso de Control Total a Recursos Sensibles).
    * `gAlumnosXXX` (Requieren Acceso de Solo Lectura a Recursos Comunes).

**Procedimiento Detallado de las Tareas:**

1.  **Preparación para la Unión al Dominio:**
    * Asegure que la hora de `SRVLINUXXXX` esté sincronizada con la del AD (Kerberos es sensible al tiempo).
    * Verifique la resolución DNS del dominio `DOMXXX.LOCAL` desde el servidor Linux.
    * Instale los paquetes necesarios para la unión al dominio (ej. `samba`, `krb5-user`, `realmd` o `sssd`).

2.  **Unión al Dominio de Active Directory:**
    * Configure el fichero `/etc/krb5.conf` para que apunte al Controlador de Dominio de su AD (KDC).
    * Utilice el comando `realm join DOMXXX.LOCAL` (o la herramienta equivalente) para unir el servidor al dominio, utilizando una cuenta administrativa del AD.
    * **Verifique la unión:** Utilice comandos de consulta (`net ads info` o `klist`) para demostrar que el servidor está unido.

3.  **Verificación de Identidades Centralizadas:**
    * Una vez unido, **verifique** que el servidor Linux reconoce a los usuarios y grupos del AD.
    * Utilice el comando `id` con un usuario del AD (ej. `id falonso@DOMXXX.LOCAL`) para confirmar que el sistema Linux le ha asignado un UID y GID válido a través de SSSD/Winbind.

4.  **Implementación de Recursos Compartidos de la Empresa (Samba):**
    * En `/srv/datasox`, cree la siguiente estructura de compartición:
        * `/srv/datasox/Recursos_Jefes` (Datos sensibles de la dirección).
        * `/srv/datasox/Documentacion_Comun` (Documentos de acceso general).
    * **Configure el fichero `smb.conf`** para crear las dos comparticiones anteriores.
    * **Aplique la política de acceso de Samba:**
        * `Recursos_Jefes`: Solo accesible por el grupo **`gJefesXXX`** del AD.
        * `Documentacion_Comun`: Accesible por el grupo **`gAlumnosXXX`** y **`gJefesXXX`**.
    * **Permisos de Sistema de Archivos (ACLs de UD07):** Asegúrese de que los permisos de Linux sobre las rutas sean lo suficientemente permisivos (ej. `rwx`) para que las restricciones de Samba puedan actuar.

5.  **Pruebas de Acceso Heterogéneo:**
    * Desde un **cliente Windows** unido a `DOMXXX`, intente acceder a las dos comparticiones y verifique que el acceso se deniega o se permite según la pertenencia a los grupos del AD.
    * Desde un **cliente Ubuntu** (como el del aula), instale las herramientas de Samba, acceda a las comparticiones y verifique que la autenticación funciona con las credenciales del AD.

**Entregable (Memoria Técnica):**

1.  **Unión a Dominio:** Capturas de la línea de comandos que muestren el proceso de `realm join` o equivalente, y la salida del comando `net ads info` o `klist`.
2.  **Verificación de Identidad:** Captura del comando `id` ejecutado con un usuario del AD que demuestre el mapeo de la identidad en Linux.
3.  **Configuración de Samba:** Copia de las secciones `[global]` y las dos secciones de recursos de `smb.conf`, donde se vean claramente definidos los parámetros de seguridad (`security = ads`) y la línea **`valid users`** utilizando los grupos de AD.
4.  **Pruebas de Acceso:** Capturas de pantalla desde los clientes (Windows y Ubuntu) que demuestren:
    * Acceso exitoso al recurso `Documentacion_Comun` con un usuario del grupo `gAlumnosXXX`.
    * **Acceso Denegado** al recurso `Recursos_Jefes` con un usuario del grupo `gAlumnosXXX`.