Aquí tienes el **tutorial completo, paso a paso y definitivo**, unificando todas las configuraciones, comandos de instalación, edición de ficheros y pruebas de verificación que hemos aplicado para tu servidor Ubuntu en el dominio `DOMSERGIO.local`.

---

### PASO 1: Preparación del Entorno y Red

Antes de configurar Samba, el sistema debe tener una identidad clara y alcanzar al Controlador de Dominio (DC).

1. **Configurar Netplan (`/etc/netplan/00-installer-config.yaml`):**
Asegúrate de que el DNS primario sea la IP de tu DC.
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.100.10/24]
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses: [192.168.100.1]
        search: [domsergio.local]

```


* **Aplicar:** `sudo netplan apply`


2. **Configurar Hosts (`/etc/hosts`):**
```text
127.0.0.1       localhost
192.168.100.10  ubuntuserver.domsergio.local  ubuntuserver
192.168.100.1 SRVSERGIO.DOMSERGIO.local SRVSERGIO domsergio.local

```


3. **Verificación de Red:**
* `ping -c 3 domsergio.local` (Debe responder el DC).
* `hostname -f` (Debe devolver `ubuntuserver.domsergio.local`).



---

### PASO 2: Instalación de Software

Instalamos Samba, Winbind (para la integración con AD) y las librerías necesarias para que el sistema reconozca a los usuarios del dominio.

```bash
sudo apt update
sudo apt install -y samba winbind libnss-winbind libpam-winbind krb5-config

```

* *Nota: Durante la instalación de Kerberos, si pide el "Default Realm", escribe `DOMSERGIO.LOCAL` en mayúsculas.*

---

### PASO 3: Configuración de Servicios (Ficheros)

#### A. Name Service Switch (`/etc/nsswitch.conf`)

Edita este fichero para que el sistema "busque" usuarios en Winbind:

```text
passwd:         files systemd winbind
group:          files systemd winbind
shadow:         files systemd
gshadow:        files systemd

hosts:          files dns
networks:       files

```

#### B. Samba y Winbind (`/etc/samba/smb.conf`)

Configuración global unificada (incluye la **enumeración** para que funcione `getent`):

```ini
[global]
   workgroup = DOMSERGIO
   realm = DOMSERGIO.LOCAL
   security = ADS
   
   # -- Winbind y Autenticación --
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config DOMSERGIO : backend = rid
   idmap config DOMSERGIO : range = 10000-999999
   template shell = /bin/bash
   template homedir = /home/%U 

   # Configuración de Winbind y Enumeración
   winbind use default domain = yes
   winbind enum users = yes
   winbind enum groups = yes
   winbind refresh tickets = Yes
   
   # Logging y Red
   log file = /var/log/samba/log.%m
   max log size = 1000
   interfaces = lo 192.168.100.0/24
   bind interfaces only = yes
   password server = 192.168.100.1

   # Soporte ACLs de Windows
   vfs objects = acl_xattr
   map acl inherit = Yes
   store dos attributes = Yes

```

---

### PASO 4: Unión al Dominio y Activación

1. **Unirse al AD:**
```bash
sudo net ads join -U Administrador

```


2. **Reiniciar y habilitar servicios:**
```bash
sudo systemctl restart winbind smbd nmbd
sudo systemctl enable winbind smbd nmbd

```



---

### PASO 5: Configuración de la Carpeta y Permisos NTFS

Para que puedas gestionar los permisos desde la pestaña "Seguridad" de Windows:

```bash
# Crear directorio
sudo mkdir -p /srv/samba/informatica

# Asignar grupo del dominio como propietario
sudo chown root:"gInformatica" /srv/samba/informatica

# Permisos de Linux con bit de herencia (SGID)
sudo chmod 2775 /srv/samba/informatica

# ACLs de POSIX para que Samba pueda escribir las ACLs de Windows
sudo setfacl -m g:"gInformatica":rwx /srv/samba/informatica
sudo setfacl -d -m g:"gInformatica":rwx /srv/samba/informatica

```

---

### PASO 6: Pruebas de Verificación (Checklist)

Ejecuta estos comandos uno a uno para asegurar que todo está OK:

1. **¿La unión es válida?**
* `sudo net ads testjoin` -> Debe decir: `Join is OK`.


2. **¿Hay comunicación con el DC?**
* `sudo wbinfo -t` -> Debe decir: `checking the trust secret ... succeeded`.


3. **¿El sistema ve a los usuarios del dominio?**
* `id Administrador` -> Debe mostrar el UID y los grupos del AD.


4. **¿Funciona la enumeración completa?** (Lo que te fallaba antes):
* `getent passwd | grep -i DOMSERGIO` -> Debe listar todos los usuarios del AD.


5. **Prueba desde Windows:**
* Accede a `\\192.168.100.10\Informatica`.
* Crea una carpeta, clic derecho -> Propiedades -> Seguridad. Deberías poder ver y añadir usuarios de tu Active Directory.


Para completar tu tutorial, aquí tienes la sección detallada para compartir la carpeta `/srv/samba/informatica`, configurando tanto el servicio Samba como los permisos profundos del sistema de archivos para permitir la gestión desde Windows.

---

### 1. Configuración del Recurso en Samba

Edita el fichero `/etc/samba/smb.conf` y añade al final la definición del recurso. He incluido las directivas necesarias para que los permisos de Windows (ACLs) se sincronicen correctamente con Linux.

```ini
[Informatica]
   comment = Carpeta de equipo gestionada desde Active Directory
   path = /srv/samba/informatica
   browseable = yes
   read only = no
   guest ok = no
   
   # Restricción de acceso a nivel de protocolo
   valid users = @gInformatica
   
   # Herencia de permisos NTFS
   vfs objects = acl_xattr
   map acl inherit = Yes
   store dos attributes = Yes
   
   # Máscaras de creación para asegurar compatibilidad
   create mask = 0664
   directory mask = 0775

```

* **Aplicar cambios:** `sudo smbcontrol all reload-config`.

---

### 2. Configuración de Permisos en el Servidor (Shell)

Para que Samba pueda asignar permisos de Windows a una carpeta de Linux, el sistema de archivos debe estar preparado. Estos comandos garantizan que el grupo `gInformatica` tenga el control inicial.

1. **Creación del directorio:**
`sudo mkdir -p /srv/samba/informatica`
2. **Asignación de grupo propietario:**
`sudo chgrp "gInformatica" /srv/samba/informatica`
3. **Permisos de bit de grupo (SGID):**
`sudo chmod 2775 /srv/samba/informatica`
*(Esto hace que cualquier archivo nuevo creado dentro pertenezca automáticamente al grupo gInformatica)*.
4. **Configuración de ACLs de Linux (POSIX):**
Samba necesita estas entradas para permitir que un usuario de Windows modifique la pestaña "Seguridad".
* `sudo setfacl -m g:"gInformatica":rwx /srv/samba/informatica`
* `sudo setfacl -d -m g:"gInformatica":rwx /srv/samba/informatica`



---

### 3. Gestión de permisos desde un cliente Windows

Una vez configurado lo anterior, ya no necesitas usar la consola de Linux para dar permisos a usuarios específicos.

1. Desde un PC del dominio, entra en `\\ubuntuserver\Informatica` con un usuario que pertenezca a `gInformatica`.
2. Haz clic derecho en la carpeta (o crea una subcarpeta) y ve a **Propiedades** > **Seguridad**.
3. Pulsa en **Editar** o **Opciones avanzadas**.
4. Ahora puedes buscar cualquier usuario de tu Active Directory (ej. `Juan Perez`) y darle permisos de "Solo lectura" o "Control Total" directamente desde la interfaz de Windows.

---

### 4. Comprobaciones de funcionamiento

Ejecuta estos comandos para verificar que la parte de compartición está lista:

* **Validar configuración de Samba:** `testparm -s` (Verifica que el recurso [Informatica] aparece correctamente).
* **Verificar visibilidad del grupo:** `getent group gInformatica` (Debe devolver el GID y los miembros del AD).
* **Verificar permisos efectivos en Linux:** `getfacl /srv/samba/informatica` (Debe mostrar las entradas `default:group:ginformatica:rwx`).

¿Te gustaría que configuremos ahora las cuotas de disco para este recurso o pasamos a la configuración de logs específicos por usuario?


