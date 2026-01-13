# TEMA 8: Integración avanzada Linux-Windows

## Introducción

En el tema anterior conseguimos algo fundamental: unir un servidor Linux al dominio Active Directory y compartir recursos básicos. Los usuarios del dominio pueden autenticarse en Linux y acceder a carpetas compartidas. Sin embargo, esto es solo el comienzo.

En un entorno empresarial real, la integración debe ser mucho más profunda y versátil. Los usuarios necesitan poder trabajar desde estaciones Linux igual que lo hacen desde Windows. Las impresoras deben estar disponibles independientemente del sistema operativo. Los recursos compartidos deben tener permisos granulares respetando la estructura organizativa del Active Directory. Y todo debe funcionar de forma automática y transparente.

Este tema lleva nuestra infraestructura híbrida al siguiente nivel. Vamos a completar la integración hasta tener un sistema realmente profesional y funcional.

### Qué vamos a hacer

**Bloque 1: Clientes Ubuntu Desktop en el dominio**

Hasta ahora solo hemos unido el servidor. Ahora vamos a unir estaciones de trabajo Ubuntu Desktop al dominio, para que los usuarios puedan iniciar sesión con sus credenciales del AD desde Linux igual que lo hacen desde Windows.

**Bloque 2: Recursos compartidos avanzados**

Crearemos una estructura completa de carpetas compartidas que refleje la organización de nuestra empresa ISCASOX, con permisos detallados por departamento, gestión de cuotas de disco y acceso diferenciado según roles.

**Bloque 3: Integración con impresoras del dominio**

Las impresoras compartidas en el Windows Server estarán disponibles automáticamente para los usuarios desde Linux. Configuraremos CUPS (el sistema de impresión de Linux) para comunicarse con las impresoras del dominio.

**Bloque 4: Perfiles móviles en entornos híbridos**

Configuraremos perfiles de usuario que funcionen tanto en Windows como en Linux, permitiendo que un usuario tenga su configuración disponible independientemente del sistema desde el que inicie sesión.

**Bloque 5: Automatización con scripts**

Crearemos scripts que automaticen tareas habituales: crear usuarios con todas sus configuraciones, generar informes, realizar backups y mantener el sistema actualizado.

Al finalizar este tema, tendréis una infraestructura híbrida completamente funcional, similar a las que encontraréis en empresas tecnológicamente avanzadas.

---

## 1. Clientes Ubuntu Desktop en el dominio

### Por qué estaciones Linux en un dominio Windows

Cada vez más empresas incorporan estaciones de trabajo Linux por varias razones:

**Desarrolladores y programadores** suelen preferir Linux porque la mayoría de herramientas de desarrollo modernas (Docker, Kubernetes, compiladores, etc.) funcionan mejor o solo funcionan en Linux.

**Ahorro en licencias**: Ubuntu Desktop es gratuito. Si una empresa tiene 50 puestos de trabajo y puede migrar 20 a Linux, se ahorra miles de euros en licencias de Windows.

**Estabilidad y rendimiento**: Linux consume menos recursos que Windows. Equipos antiguos que van lentos con Windows 10/11 funcionan perfectamente con Ubuntu.

**Seguridad**: Linux es inherentemente más seguro contra virus y malware que Windows. No necesita antivirus pesado corriendo constantemente.

**Personalización**: Los departamentos técnicos pueden personalizar completamente el sistema según sus necesidades.

Pero todo esto solo tiene sentido si los usuarios pueden trabajar exactamente igual que desde Windows: iniciar sesión con sus credenciales del AD, acceder a sus carpetas de red, imprimir en las impresoras corporativas, etc. Y eso es exactamente lo que vamos a configurar.

### Instalar Ubuntu Desktop

Primero necesitamos una estación de trabajo Ubuntu Desktop. Vamos a crear una nueva máquina virtual para ello.

**Crear la máquina virtual en VirtualBox**:

- Nombre: `PCXXX_Ubuntu_Inf01` (XXX = tu nombre)
- Tipo: Linux
- Versión: Ubuntu (64-bit)
- Memoria RAM: 4096 MB (4 GB) - Ubuntu Desktop necesita más RAM que Server
- Procesadores: 2 CPUs
- Disco duro: 30 GB (reservado dinámicamente)
- Aceleración 3D: Activada (mejora el rendimiento gráfico)

**Configuración de red**:

- Adaptador 1: Red interna `red_departamentos`

Esta VM estará en la misma red que el Windows Server y el Linux Server, por lo que podrá comunicarse con ambos.

**Descargar Ubuntu Desktop**:

Descargad **Ubuntu Desktop 22.04 LTS** desde la página oficial de Ubuntu. Es importante que sea la misma versión LTS que el servidor (22.04) para asegurar compatibilidad.

En la configuración de la VM, en Almacenamiento, añadid el ISO de Ubuntu Desktop como disco óptico.

**Instalación de Ubuntu Desktop**:

Iniciamos la VM. El instalador de Ubuntu Desktop es gráfico y mucho más intuitivo que el del servidor.

1. **Idioma**: Seleccionamos **Español** (o el que prefiráis)

2. **Distribución del teclado**: Spanish - Spanish

3. **Actualizaciones y otro software**:
   - Instalación normal (incluye navegador, utilidades, etc.)
   - Marcamos "Descargar actualizaciones al instalar Ubuntu"
   - NO marcamos "Instalar software de terceros..." (de momento)

4. **Tipo de instalación**: "Borrar disco e instalar Ubuntu" (no os preocupéis, es el disco virtual, no vuestro disco real)

5. **Zona horaria**: Seleccionamos nuestra ubicación

6. **Usuario**:
   ```
   Nombre: Usuario Local
   Nombre del equipo: PCXXX-Ubuntu-Inf01
   Nombre de usuario: usuario
   Contraseña: (contraseña sencilla)
   ```
   
   Este usuario local solo lo usaremos inicialmente. Después iniciaremos sesión con usuarios del dominio.

7. Esperamos a que termine la instalación (10-15 minutos)

8. Reiniciamos cuando nos lo pida

Tras el reinicio, nos aparecerá el escritorio de Ubuntu con su interfaz gráfica GNOME.

### Configurar red estática

Al igual que en el servidor, necesitamos configurar una IP estática.

En Ubuntu Desktop la configuración de red puede hacerse gráficamente o por terminal. Como queremos aprender ambas formas, lo haremos por terminal (más rápido y preciso).

Abrimos una terminal: `Ctrl+Alt+T` o buscamos "Terminal" en las aplicaciones.

Verificamos el nombre de nuestra interfaz de red:

```bash
ip addr
```

Veremos algo como `enp0s3` o `ens33`. Apuntamos el nombre.

Editamos la configuración de Netplan:

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

El archivo probablemente tenga este contenido (o similar):

```yaml
network:
  version: 2
  renderer: NetworkManager
```

Lo reemplazamos por esta configuración:

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      addresses:
        - 192.168.10.50/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 192.168.10.1
          - 8.8.8.8
```

**IMPORTANTE**: 
- Sustituimos `enp0s3` por el nombre real de vuestra interfaz
- La IP `192.168.10.50` es un ejemplo. Elegid una IP libre en vuestra red de departamentos (por ejemplo, entre .50 y .100)
- Mantenemos la indentación correcta (2 espacios por nivel)

Guardamos (`Ctrl+O`, `Enter`, `Ctrl+X`).

Aplicamos la configuración:

```bash
sudo netplan apply
```

Verificamos:

```bash
ip addr show enp0s3
```

Deberíamos ver nuestra IP estática asignada.

Probamos conectividad:

```bash
ping -c 4 192.168.10.1
ping -c 4 google.com
```

### Instalar paquetes necesarios

Antes de unir al dominio, instalamos los paquetes necesarios:

```bash
sudo apt update
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

Son los mismos paquetes que instalamos en el servidor.

### Unir al dominio

El proceso es muy similar al servidor, pero con algunas particularidades para el entorno de escritorio.

**Verificar que podemos descubrir el dominio**:

```bash
sudo realm discover DOMXXX.local
```

Deberíamos ver información del dominio.

**Unir al dominio**:

```bash
sudo realm join --user=Administrador DOMXXX.local
```

Introducimos la contraseña del Administrador del dominio.

**Verificar que estamos unidos**:

```bash
sudo realm list
```

Debe mostrar `configured: kerberos-member`.

**Configurar SSSD para nombres cortos**:

```bash
sudo nano /etc/sssd/sssd.conf
```

En la sección `[domain/DOMXXX.local]` añadimos:

```ini
use_fully_qualified_names = False
```

Guardamos y reiniciamos SSSD:

```bash
sudo systemctl restart sssd
```

**Configurar creación automática de carpetas personales**:

```bash
sudo pam-auth-update
```

Marcamos "Create home directory on login" y confirmamos.

### Configurar GDM para mostrar usuarios del dominio

Por defecto, la pantalla de login de Ubuntu (GDM - GNOME Display Manager) no muestra los usuarios del dominio. Debemos configurarla.

Editamos la configuración de GDM:

```bash
sudo nano /etc/gdm3/custom.conf
```

En la sección `[security]` (si no existe, la creamos), añadimos:

```ini
[security]
AllowRemoteAutoLogin=false
DisallowTCP=true

[daemon]
# Habilitar login manual
TimedLoginEnable=false
AutomaticLoginEnable=false
```

Guardamos el archivo.

Ahora configuramos PAM para permitir que GDM acepte usuarios del dominio:

```bash
sudo nano /etc/pam.d/gdm-password
```

Verificamos que contiene estas líneas (deberían estar ya):

```
auth    required    pam_env.so
auth    sufficient  pam_unix.so nullok try_first_pass
auth    requisite   pam_succeed_if.so uid >= 1000 quiet_success
auth    sufficient  pam_sss.so use_first_pass
auth    required    pam_deny.so
```

La línea clave es `auth sufficient pam_sss.so use_first_pass` que permite autenticación contra SSSD (y por tanto, el AD).

Guardamos si hemos hecho cambios.

**Reiniciamos la máquina**:

```bash
sudo reboot
```

### Iniciar sesión con usuario del dominio

Tras el reinicio, en la pantalla de login:

1. Hacemos click en "No está en la lista"
2. Introducimos el nombre de usuario del dominio: `falonso` (sin @dominio)
3. Introducimos la contraseña del usuario del dominio
4. Click en "Iniciar sesión"

Si todo está correctamente configurado, deberíamos entrar al escritorio de Ubuntu con el usuario del dominio.

Podemos verificar que estamos con un usuario del dominio abriendo una terminal:

```bash
whoami
```

Debería mostrar: `falonso`

```bash
id
```

Debería mostrar el UID del dominio y los grupos a los que pertenece el usuario en el AD.

**Carpeta personal**:

Al iniciar sesión por primera vez, se creó automáticamente la carpeta personal:

```bash
pwd
```

Mostrará: `/home/falonso`

Esta carpeta es local al equipo. Más adelante configuraremos perfiles móviles para que sea sincronizada.

### Solución de problemas comunes

**No aparece la opción "No está en la lista"**:

Editamos `/etc/gdm3/greeter.dconf-defaults`:

```bash
sudo nano /etc/gdm3/greeter.dconf-defaults
```

Buscamos o añadimos:

```ini
[org/gnome/login-screen]
disable-user-list=false
```

Guardamos y reiniciamos GDM:

```bash
sudo systemctl restart gdm3
```

**"Authentication failed" al intentar login**:

- Verificar que SSSD está funcionando: `sudo systemctl status sssd`
- Verificar que el usuario existe en el AD: `id nombre_usuario`
- Revisar logs: `sudo journalctl -u sssd -n 50`

**La carpeta personal no se crea**:

- Verificar PAM: `sudo pam-auth-update` debe tener marcado "Create home directory"
- Comprobar permisos de `/home`: debe ser `drwxr-xr-x root root`

---

## 2. Recursos compartidos avanzados

Ahora que tenemos clientes y servidores integrados con el dominio, vamos a crear una estructura completa de carpetas compartidas que refleje la organización real de nuestra empresa ISCASOX.

### Estructura organizativa de ISCASOX

Recordamos la estructura de nuestra empresa (definida en temas anteriores):

**Departamentos**:
- Gestión: María José Esteve (jefa), Jorge Broto, Jesús Calleja, Manolo Escobar
- Taller: José Arribas (jefe), María Mompó, Carlos Ferrer, Fernando Simón
- Comercial: Álex Monllor (jefe), Laura Martí, Mercedes López, Alfred Torró
- Desarrollo: Marta Pérez (jefa), Fernando Alonso, Óscar Pla, Manuel Sánchez

**Aula de formación**:
- Profesores: varios
- Alumnos: varios

Esta estructura debe reflejarse en los recursos compartidos que vamos a crear.

### Diseño de la estructura de carpetas compartidas

Vamos a crear una estructura jerárquica y organizada en nuestro servidor Linux:

```
/srv/empresa/
├── Comun/              # Acceso: todos los empleados (no alumnos/profes)
├── Direccion/          # Acceso: solo jefes de departamento
│   └── Confidencial/   # Acceso: solo gerente (Álex Monllor)
├── Departamentos/
│   ├── Gestion/        # Acceso: miembros del dpto + jefes
│   ├── Taller/
│   ├── Comercial/
│   └── Desarrollo/
└── Personal/           # Carpetas personales de cada empleado

/srv/aula/
├── Apuntes/            # Profes escriben, alumnos leen
├── Entregas/           # Alumnos escriben (sin ver), profes leen/escriben
└── Compartido/         # Profes y alumnos leen/escriben
```

### Crear la estructura de directorios

En el servidor Linux (`SRVXXX_Linux`), creamos toda la estructura de directorios:

```bash
sudo mkdir -p /srv/empresa/Comun
sudo mkdir -p /srv/empresa/Direccion/Confidencial
sudo mkdir -p /srv/empresa/Departamentos/{Gestion,Taller,Comercial,Desarrollo}
sudo mkdir -p /srv/empresa/Personal
sudo mkdir -p /srv/aula/{Apuntes,Entregas,Compartido}
```

El parámetro `-p` crea todas las carpetas intermedias necesarias.

Verificamos la estructura creada:

```bash
tree /srv/ -L 3
```

Si `tree` no está instalado:

```bash
sudo apt install -y tree
```

### Configurar permisos del sistema de archivos

Antes de compartir por Samba, configuramos los permisos a nivel del sistema de archivos Linux. Esto es fundamental porque Samba respeta estos permisos.

**Carpeta Común**:

Todos los empleados pueden leer y escribir:

```bash
sudo chown root:"DOMXXX\\domain users" /srv/empresa/Comun
sudo chmod 2775 /srv/empresa/Comun
```

**Explicación del permiso 2775**:
- `2`: bit SGID (Set Group ID). Los archivos creados heredan el grupo de la carpeta
- `7`: propietario (root) tiene rwx
- `7`: grupo (domain users) tiene rwx
- `5`: otros tienen rx (pueden entrar y listar, pero no crear/modificar)

**Carpeta Dirección**:

Solo jefes de departamento:

```bash
sudo chown root:"DOMXXX\\jefes departamento" /srv/empresa/Direccion
sudo chmod 2770 /srv/empresa/Direccion
```

Suponiendo que tienes un grupo "Jefes Departamento" en el AD. Si no lo tienes, créalo en el Windows Server con todos los jefes como miembros.

**Carpeta Confidencial dentro de Dirección**:

Solo el gerente (Álex Monllor):

```bash
sudo chown "DOMXXX\\amonllor":"DOMXXX\\domain users" /srv/empresa/Direccion/Confidencial
sudo chmod 2700 /srv/empresa/Direccion/Confidencial
```

**Carpetas de Departamentos**:

Cada departamento tiene acceso su propia carpeta. Por ejemplo, Gestión:

```bash
sudo chown root:"DOMXXX\\gestion" /srv/empresa/Departamentos/Gestion
sudo chmod 2770 /srv/empresa/Departamentos/Gestion
```

Suponiendo que tienes grupos del AD llamados `Gestion`, `Taller`, `Comercial`, `Desarrollo`. Repetimos para cada departamento:

```bash
sudo chown root:"DOMXXX\\taller" /srv/empresa/Departamentos/Taller
sudo chmod 2770 /srv/empresa/Departamentos/Taller

sudo chown root:"DOMXXX\\comercial" /srv/empresa/Departamentos/Comercial
sudo chmod 2770 /srv/empresa/Departamentos/Comercial

sudo chown root:"DOMXXX\\desarrollo" /srv/empresa/Departamentos/Desarrollo
sudo chmod 2770 /srv/empresa/Departamentos/Desarrollo
```

**Carpetas del aula**:

Apuntes (profes escriben, alumnos leen):

```bash
sudo chown root:"DOMXXX\\profesores" /srv/aula/Apuntes
sudo chmod 2775 /srv/aula/Apuntes
```

Entregas (alumnos solo escriben, no ven):

```bash
sudo chown root:"DOMXXX\\profesores" /srv/aula/Entregas
sudo chmod 3733 /srv/aula/Entregas
```

**Explicación del permiso 3733**:
- `3`: bits SGID + Sticky (archivos solo pueden ser eliminados por su propietario)
- `7`: propietario (root) tiene rwx
- `3`: grupo (profesores) tiene wx (escribir y ejecutar, no leer)
- `3`: otros (alumnos) tienen wx (pueden crear archivos pero no ver los existentes)

Compartido (todos leen y escriben):

```bash
sudo chown root:"DOMXXX\\domain users" /srv/aula/Compartido
sudo chmod 2775 /srv/aula/Compartido
```

### Configurar recursos compartidos en Samba

Ahora configuramos Samba para compartir todas estas carpetas con los permisos adecuados.

Editamos la configuración de Samba:

```bash
sudo nano /etc/samba/smb.conf
```

Eliminamos la configuración anterior de `[Compartido]` y añadimos todas las nuevas comparticiones al final del archivo:

```ini
# ==========================================
# RECURSOS COMPARTIDOS DE LA EMPRESA
# ==========================================

[Comun]
   comment = Carpeta común para todos los empleados
   path = /srv/empresa/Comun
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Domain Users"
   write list = @"DOMXXX\Domain Users"
   create mask = 0664
   directory mask = 0775

[Direccion]
   comment = Carpeta para jefes de departamento
   path = /srv/empresa/Direccion
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Jefes Departamento"
   write list = @"DOMXXX\Jefes Departamento"
   create mask = 0660
   directory mask = 0770

[Confidencial]
   comment = Carpeta confidencial del gerente
   path = /srv/empresa/Direccion/Confidencial
   browseable = no
   read only = no
   valid users = "DOMXXX\amonllor"
   write list = "DOMXXX\amonllor"
   create mask = 0600
   directory mask = 0700

[Gestion]
   comment = Departamento de Gestión
   path = /srv/empresa/Departamentos/Gestion
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Gestion", @"DOMXXX\Jefes Departamento"
   write list = @"DOMXXX\Gestion"
   create mask = 0660
   directory mask = 0770

[Taller]
   comment = Departamento de Taller
   path = /srv/empresa/Departamentos/Taller
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Taller", @"DOMXXX\Jefes Departamento"
   write list = @"DOMXXX\Taller"
   create mask = 0660
   directory mask = 0770

[Comercial]
   comment = Departamento Comercial
   path = /srv/empresa/Departamentos/Comercial
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Comercial", @"DOMXXX\Jefes Departamento"
   write list = @"DOMXXX\Comercial"
   create mask = 0660
   directory mask = 0770

[Desarrollo]
   comment = Departamento de Desarrollo
   path = /srv/empresa/Departamentos/Desarrollo
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Desarrollo", @"DOMXXX\Jefes Departamento"
   write list = @"DOMXXX\Desarrollo"
   create mask = 0660
   directory mask = 0770

# ==========================================
# RECURSOS COMPARTIDOS DEL AULA
# ==========================================

[Apuntes]
   comment = Apuntes para alumnos
   path = /srv/aula/Apuntes
   browseable = yes
   read only = yes
   write list = @"DOMXXX\Profesores"
   valid users = @"DOMXXX\Profesores", @"DOMXXX\Alumnos"
   create mask = 0664
   directory mask = 0775

[Entregas]
   comment = Entregas de trabajos
   path = /srv/aula/Entregas
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Profesores", @"DOMXXX\Alumnos"
   write list = @"DOMXXX\Profesores", @"DOMXXX\Alumnos"
   create mask = 0640
   directory mask = 0750

[Compartido_Aula]
   comment = Carpeta compartida del aula
   path = /srv/aula/Compartido
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Profesores", @"DOMXXX\Alumnos"
   write list = @"DOMXXX\Profesores", @"DOMXXX\Alumnos"
   create mask = 0664
   directory mask = 0775
```

**IMPORTANTE**: Sustituimos `DOMXXX` por el nombre real del dominio en todas las ocurrencias.

**Notas sobre la configuración**:

- `browseable = yes/no`: si aparece al navegar por la red
- `read only = yes/no`: si es de solo lectura por defecto
- `valid users`: quién puede acceder (ver)
- `write list`: quién puede escribir
- El `@` antes del nombre indica que es un grupo
- Las comillas son necesarias cuando el nombre contiene espacios o contrabarras

Guardamos el archivo.

Verificamos que no hay errores de sintaxis:

```bash
testparm
```

Si todo está correcto, reiniciamos Samba:

```bash
sudo systemctl restart smbd
```

### Probar los recursos compartidos

Ahora probamos que todo funciona correctamente desde diferentes usuarios.

**Desde Windows**:

1. Iniciamos sesión en un cliente Windows con un usuario del departamento de Gestión (por ejemplo, `jbroto`)

2. Abrimos el Explorador de archivos y navegamos a `\\192.168.10.2` o `\\SRVXXX_Linux`

3. Deberíamos ver:
   - Comun
   - Gestion (puede acceder)
   - Apuntes, Entregas, Compartido_Aula

4. NO deberíamos ver:
   - Confidencial (no somos el gerente)
   - Taller, Comercial, Desarrollo (no somos de esos departamentos)

5. Entramos en Comun y probamos a crear un archivo

6. Entramos en Gestion y probamos a crear un archivo

**Desde Ubuntu Desktop**:

1. Iniciamos sesión con el mismo usuario (`jbroto`)

2. Abrimos el gestor de archivos (Nautilus)

3. En la barra lateral izquierda, click en "+ Otras ubicaciones"

4. En el campo "Conectarse a un servidor" escribimos: `smb://192.168.10.2`

5. Pulsamos "Conectar"

6. Nos pedirá credenciales (aunque ya hemos iniciado sesión). Introducimos:
   - Usuario: `DOMXXX\jbroto`
   - Contraseña: (su contraseña del AD)

7. Deberíamos ver los mismos recursos compartidos que desde Windows

8. Probamos a crear archivos en las carpetas a las que tenemos acceso

### Configurar montaje automático de recursos

Para que los usuarios no tengan que conectarse manualmente cada vez, configuramos el montaje automático de las carpetas de red.

En Ubuntu Desktop, esto se puede hacer de varias formas. Vamos a usar la más sencilla: configurar `fstab` con credenciales del usuario.

**ADVERTENCIA DE SEGURIDAD**: Guardar contraseñas en archivos es inseguro. En producción usaríamos Kerberos tickets. Para el entorno de aprendizaje, lo haremos así por simplicidad.

Creamos un archivo con las credenciales:

```bash
nano ~/.smbcredentials
```

Contenido:

```
username=jbroto
password=ContraseñaDelUsuario
domain=DOMXXX
```

Protegemos el archivo para que solo el propietario pueda leerlo:

```bash
chmod 600 ~/.smbcredentials
```

Ahora creamos puntos de montaje locales:

```bash
mkdir -p ~/Empresa/Comun
mkdir -p ~/Empresa/Gestion
```

Editamos `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Añadimos al final:

```
//192.168.10.2/Comun    /home/jbroto/Empresa/Comun    cifs    credentials=/home/jbroto/.smbcredentials,uid=jbroto,gid=jbroto,iocharset=utf8    0    0
//192.168.10.2/Gestion  /home/jbroto/Empresa/Gestion  cifs    credentials=/home/jbroto/.smbcredentials,uid=jbroto,gid=jbroto,iocharset=utf8    0    0
```

**Explicación**:
- `cifs`: tipo de sistema de archivos (SMB/CIFS)
- `credentials`: archivo con las credenciales
- `uid` y `gid`: propietario de los archivos montados
- `iocharset=utf8`: codificación de caracteres

Guardamos.

Montamos sin reiniciar:

```bash
sudo mount -a
```

Verificamos:

```bash
ls ~/Empresa/Comun
ls ~/Empresa/Gestion
```

Deberíamos ver el contenido de las carpetas compartidas.

Ahora, cada vez que el usuario inicie sesión, estas carpetas estarán automáticamente montadas y accesibles desde su carpeta personal.

---

[Continúo con los siguientes bloques: Impresoras, Perfiles móviles y Scripts. ¿Quieres que continúe ahora o prefieres revisar lo que llevamos hasta ahora?]

----
---
---
---
---
---
