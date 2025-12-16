# TEMA 7: Linux Server e integración básica con Windows

## Introducción: Redes heterogéneas

Hasta ahora hemos trabajado solo con Windows Server. Hemos montado un dominio, compartido recursos, aplicado directivas... todo en Windows. Pero cuando visitéis empresas medianas o grandes, veréis que lo habitual es encontrar **Windows y Linux trabajando juntos**. A esto se le llama una **red heterogénea**.

> **IMAGEN SUGERIDA**: Diagrama simple mostrando servidor Windows + servidor Linux conectados, con clientes accediendo a ambos

### Por qué mezclar Windows y Linux

Las empresas no mezclan sistemas por capricho. Hay razones muy concretas:

**Ahorro de costes**: Windows Server requiere licencias que cuestan cientos de euros por servidor. Linux es gratuito. Una empresa con 10 servidores puede ahorrarse miles de euros usando Linux donde sea posible.

**Cada sistema tiene sus fortalezas**:
- Windows destaca en: gestión centralizada (AD), integración con Office/Exchange, aplicaciones empresariales específicas
- Linux destaca en: servidores web, estabilidad (meses sin reiniciar), seguridad, rendimiento con pocos recursos

**Aplicaciones modernas**: Muchas tecnologías actuales (Docker, Kubernetes, la mayoría de frameworks web) están optimizadas para Linux. Si quieres usarlas, necesitas servidores Linux.

> **IMAGEN SUGERIDA**: Tabla comparativa simple Windows Server vs Linux Server con iconos

### El reto: que todo funcione junto

El desafío no es técnico, es de integración. **Los usuarios no deben notar que hay sistemas diferentes**. Cuando un empleado accede a una carpeta compartida, no debería importarle si está en Windows o Linux. Debe usar las mismas credenciales para todo.

Para conseguir esto usamos **protocolos estándar** que ambos sistemas entienden:
- **SMB/CIFS**: compartir archivos (lo usa Windows, Samba lo implementa en Linux)
- **LDAP**: directorios de usuarios
- **Kerberos**: autenticación segura
- **DNS**: resolución de nombres

> **IMAGEN SUGERIDA**: Diagrama de flujo mostrando usuario → autenticación AD → acceso a recursos Windows y Linux

## Linux: conceptos básicos

### Qué es Linux

Linux no es un sistema operativo completo, es un **kernel** (núcleo). Las **distribuciones** toman el kernel y le añaden todo lo necesario: herramientas, gestores de paquetes, servicios...

Nosotros usaremos **Ubuntu Server 22.04 LTS** porque:
- Es gratuita
- Tiene soporte hasta 2027
- Comunidad enorme (fácil encontrar ayuda)
- La más usada en cloud (AWS, Azure, Google Cloud)
- Ideal para aprender

> **IMAGEN SUGERIDA**: Logos de distribuciones populares (Ubuntu, Debian, Red Hat, CentOS)

### Diferencias clave con Windows Server

**Interfaz**: Windows tiene ventanas y menús. Linux Server se administra por **terminal** (línea de comandos). Parece más difícil pero consume menos recursos y facilita la automatización.

> **IMAGEN SUGERIDA**: Captura de pantalla de terminal Linux vs ventana de Windows

**Sistema de archivos**: 
- Windows: letras de unidad (C:, D:, E:)
- Linux: un único árbol que empieza en `/` (raíz). Los discos se "montan" en carpetas.

Ejemplo: en Windows tienes `D:\datos`. En Linux sería `/datos` (el disco se monta ahí).

> **IMAGEN SUGERIDA**: Esquema del árbol de directorios Linux (/, /home, /etc, /var, /srv)

**Instalación de programas**:
- Windows: descargas .exe y haces doble clic
- Linux: usas el gestor de paquetes `apt`

```bash
sudo apt install nombre_programa
```

Todo descarga, instala y configura automáticamente. Es como una tienda de aplicaciones pero más potente.

**Usuarios y permisos**: Linux es multiusuario desde su origen. Hay un superusuario llamado **root** (como Administrador en Windows), pero por seguridad no trabajamos como root. Usamos `sudo` para ejecutar comandos con privilegios elevados.

**Configuración**: En Windows está en el Registro. En Linux, todo son **archivos de texto** en `/etc/`. Ventajas: puedes editarlos, copiarlos, hacer backups fácilmente.

## Qué vamos a hacer en este tema

Vamos a incorporar un servidor Linux a nuestra red ISCASOX. No será un sistema aislado, sino integrado con el dominio Windows desde el principio.

**Bloque 1: Instalación y configuración básica**
- Instalar Ubuntu Server en VirtualBox
- Configurar red estática con Netplan
- Configurar SSH para administrar remotamente

**Bloque 2: LVM (Logical Volume Manager)**
- Gestión flexible de almacenamiento
- Crear, ampliar y gestionar volúmenes sin reiniciar

**Bloque 3: Samba**
- Instalar Samba (permite que Linux hable SMB de Windows)
- Crear recursos compartidos básicos
- Probar acceso desde Windows

**Bloque 4: Integración con Active Directory**
- Unir el servidor Linux al dominio Windows
- Usuarios del AD pueden autenticarse en Linux
- Recursos compartidos respetan permisos del AD

> **IMAGEN SUGERIDA**: Diagrama del resultado final: AD Windows + Linux integrado + clientes accediendo a ambos

Al final tendremos una infraestructura híbrida real, como las que veréis en empresas.

## Cómo trabajar en este tema

**En clase**: Explicaremos conceptos y haremos demos de procesos complejos.

**En los apuntes**: Guías paso a paso muy detalladas que seguís a vuestro ritmo.

**Importante**: 
- Haced **snapshots** (instantáneas) de la VM antes de cambios importantes
- Si algo falla, restauráis y lo intentáis de nuevo
- No tengáis miedo a "romper" el sistema: estáis en VMs
- **No copiéis comandos sin leer**: entended qué hace cada comando
- Los errores en Linux suelen ser muy informativos. Leedlos.

**Cuando algo no funcione**:
1. Lee el mensaje de error completo
2. Busca el error en Google (copia el mensaje exacto)
3. Consulta al profesor

> **IMAGEN SUGERIDA**: Captura mostrando cómo hacer snapshot en VirtualBox

## Herramientas de ayuda

**Comandos útiles para obtener ayuda**:
```bash
man nombre_comando    # Manual completo del comando
comando --help        # Ayuda rápida del comando
```

**Herramientas gráficas opcionales**:
- **Webmin**: panel web de administración
- **Cockpit**: panel moderno de gestión del sistema

No son obligatorias, pero pueden ayudar cuando empezáis.

> **IMAGEN SUGERIDA**: Captura del dashboard de Cockpit


# 2. Instalación de Ubuntu Server

Una vez creada la máquina virtual, la iniciamos y comenzará el proceso de instalación desde el ISO.

Ubuntu Server no tiene entorno gráfico, todo se hace desde la terminal. No te preocupes, el instalador es muy intuitivo y guiado.

#### Proceso de instalación paso a paso

**Pantalla de idioma:**
- Seleccionamos **English** (es más fácil buscar ayuda en inglés si hay problemas)
- Pulsamos `Enter`

**Keyboard configuration:**
- Layout: **Spanish**
- Variant: **Spanish**
- Pulsamos `Done` (con las flechas nos movemos, con `Enter` seleccionamos)

**Choose type of install:**
- Dejamos seleccionado **Ubuntu Server** (por defecto)
- Pulsamos `Done`

**Network connections:**
Aquí veremos nuestras 3 tarjetas de red. De momento todas tendrán configuración automática (DHCP) o sin configurar. Esto lo arreglaremos después de la instalación.
- No tocamos nada por ahora
- Pulsamos `Done`

**Configure proxy:**
- Dejamos en blanco (no necesitamos proxy)
- Pulsamos `Done`

**Configure Ubuntu archive mirror:**
- Dejamos la URL por defecto
- Pulsamos `Done`

**Guided storage configuration:**
Esta es una de las pantallas más importantes. Aquí decidimos cómo particionar el disco.

Para esta instalación inicial, vamos a usar el **particionado automático**. Más adelante añadiremos discos y configuraremos LVM manualmente.

- Dejamos seleccionado **Use an entire disk**
- Dejamos seleccionado **Set up this disk as an LVM group**
- Pulsamos `Done`

Nos aparecerá un resumen del particionado. Veremos algo como:
```
USED DEVICES
ubuntu-lv     /     12.000G
```

- Pulsamos `Done`
- Nos preguntará si estamos seguros → Seleccionamos `Continue`

**Profile setup:**
Aquí configuramos el usuario administrador principal del sistema.

```
Your name: Administrador
Your server's name: SRVXXX_Linux  (XXX = tu nombre)
Pick a username: admin
Choose a password: (contraseña sencilla que recordemos)
Confirm your password: (repetir contraseña)
```

**IMPORTANTE**: Apunta este usuario y contraseña, lo necesitarás constantemente.

**SSH Setup:**
SSH (Secure Shell) es el protocolo que usaremos para conectarnos remotamente al servidor desde Windows. Es fundamental instalarlo.

- Marcamos con `Espacio` la opción **Install OpenSSH server**
- Dejamos desmarcado "Import SSH identity"
- Pulsamos `Done`

**Featured Server Snaps:**
Nos ofrece instalar algunos paquetes populares. De momento no instalamos nada.
- No marcamos ninguno
- Pulsamos `Done`

**Instalación:**
Comenzará la instalación. Tardará unos minutos. Veremos el progreso en pantalla.

Cuando termine la instalación, aparecerá un botón `Reboot Now`.
- Pulsamos `Reboot Now`
- Si nos pide que quitemos el medio de instalación, simplemente pulsamos `Enter`

El sistema se reiniciará y nos aparecerá una pantalla de login en modo texto:

```
Ubuntu 22.04.X LTS SRVXXX_Linux tty1

SRVXXX_Linux login: _
```

**¡Enhorabuena! Has instalado Ubuntu Server.**

### 1.3. Primer acceso al sistema

Ahora vamos a acceder por primera vez a nuestro servidor.

En la pantalla de login, introducimos:
```
login: admin
password: (nuestra contraseña)
```

Si todo va bien, veremos algo como:
```
Welcome to Ubuntu 22.04.X LTS (GNU/Linux 5.15.0-XX-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: ...
admin@SRVXXX_Linux:~$ _
```

Este es el **prompt** de Linux. Nos indica:
- **admin**: usuario actual
- **SRVXXX_Linux**: nombre del servidor
- **~**: estamos en nuestra carpeta personal (`/home/admin`)
- **$**: símbolo que indica que somos un usuario normal (no root)

#### Comandos básicos para orientarnos

Vamos a ejecutar algunos comandos básicos para familiarizarnos:

**Ver en qué directorio estamos:**
```bash
pwd
```
Resultado: `/home/admin`

**Listar archivos del directorio actual:**
```bash
ls
```

**Listar archivos con detalles:**
```bash
ls -la
```

**Ver información del sistema:**
```bash
uname -a
```

**Ver información de red:**
```bash
ip addr
```
Aquí veremos nuestras tarjetas de red. Deberían aparecer 3 interfaces (además de `lo` que es la interfaz local):
- `enp0s3`: adaptador 1 (NAT o puente)
- `enp0s8`: adaptador 2 (red departamentos)
- `enp0s9`: adaptador 3 (red aula)

Los nombres pueden variar ligeramente (`enp0s3`, `ens33`, `eth0`, etc.), pero habrá 3 interfaces.

**Verificar conectividad a Internet:**
```bash
ping -c 4 google.com
```
El `-c 4` indica que haga 4 pings y pare (en Linux, ping no para automáticamente como en Windows).

Si funciona, veremos algo como:
```
64 bytes from ... time=20ms
...
4 packets transmitted, 4 received, 0% packet loss
```

**Actualizar el sistema:**
Es buena práctica actualizar el sistema recién instalado:

```bash
sudo apt update
```

Nos pedirá la contraseña de `admin`. La escribimos (no se verá nada al escribir, es normal) y pulsamos `Enter`.

Este comando actualiza la lista de paquetes disponibles.

```bash
sudo apt upgrade -y
```

Este comando instala las actualizaciones. El `-y` responde automáticamente "yes" a todas las preguntas.

Puede tardar unos minutos. Espera a que termine.

#### ¿Qué es sudo?

En Linux, las tareas de administración requieren permisos de **root** (el superusuario, equivalente a "Administrador" en Windows).

Por seguridad, no trabajamos como root directamente. En su lugar, usamos el comando `sudo` (SuperUser DO) antes de comandos que requieren permisos elevados.

Cuando usamos `sudo`, nos pide la contraseña de nuestro usuario, y si tenemos permisos, ejecuta el comando como root.

---

## 2. Configuración de red estática con Netplan

### 2.1. ¿Por qué configuración estática?

Cuando instalamos Ubuntu Server, las tarjetas de red se configuran automáticamente mediante DHCP (si hay un servidor DHCP disponible) o quedan sin configurar.

Para un servidor, necesitamos que las direcciones IP sean **fijas** (estáticas), no pueden cambiar cada vez que se reinicia. Imagina si cada vez que se reinicia el servidor cambia su IP: los clientes no sabrían dónde encontrarlo.

### 2.2. Netplan: el gestor de red de Ubuntu

Ubuntu Server usa **Netplan** para configurar la red. Netplan es un sistema que lee archivos de configuración en formato YAML y aplica la configuración de red.

Los archivos de configuración de Netplan están en:
```
/etc/netplan/
```

Vamos a ver qué archivos hay:
```bash
ls /etc/netplan/
```

Normalmente encontraremos un archivo llamado `00-installer-config.yaml` o similar.

Vamos a ver su contenido:
```bash
sudo cat /etc/netplan/00-installer-config.yaml
```

Veremos algo como:
```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
    enp0s9:
      dhcp4: true
  version: 2
```

Esto significa que las 3 interfaces están configuradas para obtener IP automáticamente mediante DHCP.

### 2.3. Configurar IPs estáticas

Vamos a modificar este archivo para configurar IPs estáticas en las interfaces de las redes internas (`enp0s8` y `enp0s9`). La primera interfaz (`enp0s3`) la dejaremos con DHCP para que tenga salida a Internet automáticamente.

#### Esquema de red a configurar

Según el diseño de nuestra red ISCASOX:

- **enp0s3** (Adaptador 1): DHCP (salida a Internet)
- **enp0s8** (Adaptador 2 - red departamentos): `192.168.10.2/24`
  - Gateway: `192.168.10.1` (Windows Server)
- **enp0s9** (Adaptador 3 - red aula): `172.16.X.2/24` (X = número de tu equipo)
  - Gateway: `172.16.X.1` (Windows Server)

**IMPORTANTE**: Los nombres de las interfaces (`enp0s3`, `enp0s8`, `enp0s9`) pueden variar en tu sistema. Usa el comando `ip addr` para verificar los nombres reales de tus interfaces.

#### Editar el archivo de configuración

Para editar archivos en Linux desde la terminal, usamos un editor de texto. Los más comunes son `nano` (más sencillo) y `vi` (más potente pero complejo).

Usaremos **nano** por ser más intuitivo:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Borramos todo el contenido y escribimos la siguiente configuración:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.10.2/24
      routes:
        - to: default
          via: 192.168.10.1
          metric: 200
      nameservers:
        addresses:
          - 192.168.10.1
          - 8.8.8.8
    enp0s9:
      addresses:
        - 172.16.X.2/24
      nameservers:
        addresses:
          - 192.168.10.1
```

**IMPORTANTE**: 
- Sustituye la `X` por el número de tu equipo
- En YAML, la **indentación es crítica**. Debe ser con **espacios**, no tabuladores
- Cada nivel de indentación son **2 espacios**
- Verifica los nombres de tus interfaces con `ip addr` y ajusta si son diferentes

#### Guardar el archivo

Para guardar en nano:
- `Ctrl + O` (letra O, de "Output") → Confirmar con `Enter`
- `Ctrl + X` para salir

#### Aplicar la configuración

Antes de aplicar, verificamos que la sintaxis del archivo es correcta:

```bash
sudo netplan try
```

Este comando aplica la configuración temporalmente y nos pregunta si funciona. Si no respondemos en 120 segundos, revierte los cambios. Esto es una medida de seguridad para no quedarnos sin acceso.

Si todo va bien, veremos:
```
Do you want to keep these settings?

Press ENTER before the timeout to accept the new configuration

Changes will revert in 120 seconds
```

Pulsamos `Enter` para aceptar.

Si da algún error de sintaxis, nos lo indicará. Los errores más comunes son:
- Indentación incorrecta (usar tabuladores en vez de espacios)
- Nombres de interfaces incorrectos
- Falta de dos puntos (`:`) después de las claves

#### Verificar la configuración

Una vez aplicada, verificamos las IPs:

```bash
ip addr show
```

Deberíamos ver nuestras 3 interfaces con las IPs configuradas:
- `enp0s3`: IP automática (DHCP)
- `enp0s8`: `192.168.10.2/24`
- `enp0s9`: `172.16.X.2/24`

Verificamos conectividad a Internet:
```bash
ping -c 4 google.com
```

Verificamos conectividad con el Windows Server:
```bash
ping -c 4 192.168.10.1
```

Si todo funciona, ¡perfecto! Ya tenemos la red configurada.

### 2.4. Configurar el nombre del servidor (hostname)

El nombre del servidor se configura con el comando `hostnamectl`:

```bash
sudo hostnamectl set-hostname SRVXXX_Linux
```

Para ver el nombre actual:
```bash
hostnamectl
```

**IMPORTANTE**: También debemos editar el archivo `/etc/hosts` para que el sistema resuelva correctamente su propio nombre:

```bash
sudo nano /etc/hosts
```

Debe contener al menos estas líneas:
```
127.0.0.1 localhost
127.0.1.1 SRVXXX_Linux

# Las siguientes líneas son para IPv6
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Guardamos (`Ctrl+O`, `Enter`, `Ctrl+X`).

Para que los cambios se apliquen completamente, reiniciamos:
```bash
sudo reboot
```

El sistema se reiniciará. Esperamos unos segundos y volvemos a hacer login.

---

## 3. Gestión avanzada de almacenamiento con LVM

### 3.1. ¿Qué es LVM y por qué usarlo?

**LVM** (Logical Volume Manager) es un sistema de gestión de almacenamiento que añade una capa de abstracción entre los discos físicos y el sistema de archivos.

#### Comparación: Particiones tradicionales vs LVM

**Con particiones tradicionales:**
- Creas particiones de tamaño fijo en el disco
- Si una partición se queda sin espacio, es **muy difícil** ampliarla
- No puedes "mover" espacio de una partición a otra fácilmente
- No puedes combinar varios discos en una sola partición

**Con LVM:**
- Puedes **ampliar** volúmenes fácilmente (incluso con el sistema en funcionamiento)
- Puedes **reducir** volúmenes si necesitas recuperar espacio
- Puedes **combinar** varios discos físicos en un único volumen
- Puedes crear **snapshots** (copias instantáneas) de volúmenes
- Puedes **mover** datos entre discos sin parar el sistema

**Analogía**: 
- Particiones tradicionales son como habitaciones de una casa con paredes de hormigón: muy difícil cambiar su tamaño.
- LVM es como una oficina con paneles modulares: puedes mover las "paredes" y reorganizar el espacio fácilmente.

En Windows Server algo similar serían los **Storage Spaces**, aunque LVM es más potente y flexible.

### 3.2. Conceptos de LVM

LVM funciona con 3 niveles:

1. **PV (Physical Volume - Volumen Físico)**:
   - Es un disco duro físico (o partición) preparado para LVM
   - Comando para crear: `pvcreate`
   - Comando para ver: `pvs` o `pvdisplay`

2. **VG (Volume Group - Grupo de Volúmenes)**:
   - Es un "conjunto" de uno o más PVs
   - Es como una "bolsa de espacio" que puedes repartir
   - Comando para crear: `vgcreate`
   - Comando para ver: `vgs` o `vgdisplay`

3. **LV (Logical Volume - Volumen Lógico)**:
   - Es el "volumen" que usaremos realmente
   - Se crea dentro de un VG
   - Es lo que montaremos como carpeta en el sistema
   - Comando para crear: `lvcreate`
   - Comando para ver: `lvs` o `lvdisplay`

**Ejemplo visual:**
```
Discos físicos: [Disco1] [Disco2] [Disco3]
       ↓           ↓         ↓        ↓
       └───────── PVs ─────────┘
                   ↓
               [VG: vg_datos]  ← Grupo que agrupa los 3 discos
                   ↓
       ┌───────────┴───────────┐
       ↓           ↓           ↓
  [LV: empresa] [LV: usuarios] [LV: backup]
       ↓           ↓           ↓
   /srv/empresa /srv/usuarios /srv/backup
```

### 3.3. Añadir discos a la máquina virtual

Antes de trabajar con LVM, necesitamos añadir discos adicionales a nuestra VM.

#### Apagar la máquina virtual

Desde la terminal del servidor:
```bash
sudo poweroff
```

#### Añadir 3 discos en VirtualBox

Con la VM apagada, en VirtualBox:

1. Seleccionamos la VM `SRVXXX_Linux`
2. Click en **Configuración**
3. Vamos a **Almacenamiento**
4. En **Controladora: SATA**, click en el icono de disco con el `+` (Añadir disco duro)
5. Click en **Crear**
6. Configurar el disco:
   - Tipo: **VDI**
   - Reservado dinámicamente
   - Tamaño: **10 GB**
   - Click en **Crear**
7. Repetimos el proceso 2 veces más para tener **3 discos de 10GB**

Al final deberíamos tener en la controladora SATA:
- `SRVXXX_Linux.vdi` (disco principal, 25GB)
- `SRVXXX_Linux_1.vdi` (disco adicional 1, 10GB)
- `SRVXXX_Linux_2.vdi` (disco adicional 2, 10GB)
- `SRVXXX_Linux_3.vdi` (disco adicional 3, 10GB)

Click en **Aceptar** para guardar cambios.

#### Iniciar la VM y verificar discos

Iniciamos la VM y hacemos login.

Verificamos que los nuevos discos se detectan:
```bash
lsblk
```

Este comando lista los dispositivos de bloque (discos). Deberíamos ver algo como:

```
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0   11G  0 lvm  /
sdb                         8:16   0   10G  0 disk 
sdc                         8:32   0   10G  0 disk 
sdd                         8:48   0   10G  0 disk 
```

Aquí vemos:
- `sda`: disco principal (25GB) con el sistema ya instalado
- `sdb`, `sdc`, `sdd`: los 3 discos nuevos de 10GB que acabamos de añadir

**Nota**: Los nombres pueden ser diferentes en tu sistema (`vda`, `vdb`, etc.). Lo importante es identificar los 3 discos de 10GB.

### 3.4. Crear la estructura LVM

Vamos a crear paso a paso nuestra estructura LVM.

#### Paso 1: Crear Physical Volumes (PVs)

Convertimos los 3 discos en volúmenes físicos para LVM:

```bash
sudo pvcreate /dev/sdb /dev/sdc /dev/sdd
```

Deberías ver:
```
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
```

Verificamos:
```bash
sudo pvs
```

Resultado:
```
  PV         VG        Fmt  Attr PSize  PFree 
  /dev/sda3  ubuntu-vg lvm2 a--  23.00g    0 
  /dev/sdb             lvm2 ---  10.00g 10.00g
  /dev/sdc             lvm2 ---  10.00g 10.00g
  /dev/sdd             lvm2 ---  10.00g 10.00g
```

Vemos los 3 PVs nuevos (`/dev/sdb`, `/dev/sdc`, `/dev/sdd`) sin asignar a ningún VG todavía.

#### Paso 2: Crear Volume Group (VG)

Ahora agrupamos los 3 PVs en un único VG llamado `vg_datos`:

```bash
sudo vgcreate vg_datos /dev/sdb /dev/sdc /dev/sdd
```

Resultado:
```
  Volume group "vg_datos" successfully created
```

Verificamos:
```bash
sudo vgs
```

Resultado:
```
  VG        #PV #LV #SN Attr   VSize  VFree 
  ubuntu-vg   1   1   0 wz--n- 23.00g     0 
  vg_datos    3   0   0 wz--n- 29.99g 29.99g
```

Vemos nuestro VG `vg_datos` con ~30GB de espacio (10GB × 3 discos).

#### Paso 3: Crear Logical Volumes (LVs)

Ahora creamos 3 volúmenes lógicos dentro de `vg_datos`:

**Volumen para datos de empresa (15GB):**
```bash
sudo lvcreate -L 15G -n lv_empresa vg_datos
```

**Volumen para carpetas de usuarios (10GB):**
```bash
sudo lvcreate -L 10G -n lv_usuarios vg_datos
```

**Volumen para datos compartidos (5GB):**
```bash
sudo lvcreate -L 5G -n lv_compartido vg_datos
```

Verificamos:
```bash
sudo lvs
```

Resultado:
```
  LV           VG        Attr       LSize  
  ubuntu-lv    ubuntu-vg -wi-ao---- 11.00g
  lv_compartido vg_datos -wi-a-----  5.00g
  lv_empresa   vg_datos  -wi-a----- 15.00g
  lv_usuarios  vg_datos  -wi-a----- 10.00g
```

Perfecto, tenemos nuestros 3 volúmenes creados.

Verificamos el espacio restante en el VG:
```bash
sudo vgs
```

```
  VG        #PV #LV #SN Attr   VSize  VFree 
  ubuntu-vg   1   1   0 wz--n- 23.00g     0 
  vg_datos    3   3   0 wz--n- 29.99g 4.99g
```

Quedan ~5GB libres en `vg_datos` que podremos usar más adelante.

#### Paso 4: Crear sistemas de archivos en los LVs

Los LVs están creados, pero son como "discos en blanco", necesitan un sistema de archivos.

En Linux, el sistema de archivos más común es **ext4**. Vamos a formatear nuestros LVs con ext4:

```bash
sudo mkfs.ext4 /dev/vg_datos/lv_empresa
sudo mkfs.ext4 /dev/vg_datos/lv_usuarios
sudo mkfs.ext4 /dev/vg_datos/lv_compartido
```

Cada comando tardará unos segundos y mostrará información sobre el sistema de archivos creado.

#### Paso 5: Crear puntos de montaje

En Linux, para usar un volumen, debemos **montarlo** en una carpeta. Esa carpeta se llama **punto de montaje**.

Vamos a crear las carpetas donde montaremos nuestros volúmenes:

```bash
sudo mkdir -p /srv/empresa
sudo mkdir -p /srv/usuarios
sudo mkdir -p /srv/compartido
```

El parámetro `-p` crea las carpetas intermedias si no existen.

#### Paso 6: Montar los volúmenes

Ahora montamos cada LV en su carpeta correspondiente:

```bash
sudo mount /dev/vg_datos/lv_empresa /srv/empresa
sudo mount /dev/vg_datos/lv_usuarios /srv/usuarios
sudo mount /dev/vg_datos/lv_compartido /srv/compartido
```

Verificamos que están montados:
```bash
df -h
```

Deberíamos ver al final de la lista:
```
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/vg_datos-lv_empresa     15G   24K   14G   1% /srv/empresa
/dev/mapper/vg_datos-lv_usuarios   9.8G   24K  9.3G   1% /srv/usuarios
/dev/mapper/vg_datos-lv_compartido 4.9G   24K  4.7G   1% /srv/compartido
```

¡Perfecto! Los volúmenes están montados y listos para usar.

#### Paso 7: Montaje automático con /etc/fstab

Si reiniciamos ahora el servidor, los volúmenes NO se montarán automáticamente. Debemos configurar el archivo `/etc/fstab` para que se monten en cada arranque.

Editamos el archivo:
```bash
sudo nano /etc/fstab
```

Añadimos al final estas 3 líneas:
```
/dev/vg_datos/lv_empresa    /srv/empresa     ext4  defaults  0  2
/dev/vg_datos/lv_usuarios   /srv/usuarios    ext4  defaults  0  2
/dev/vg_datos/lv_compartido /srv/compartido  ext4  defaults  0  2
```

Guardamos (`Ctrl+O`, `Enter`, `Ctrl+X`).

**Verificar que no hay errores:**

Antes de reiniciar, verificamos que el archivo `fstab` no tiene errores:
```bash
sudo mount -a
```

Si no da ningún error, significa que la configuración es correcta.

Verificamos de nuevo:
```bash
df -h
```

Los volúmenes deben seguir montados.

**Reiniciamos para comprobar que funcionan al arrancar:**
```bash
sudo reboot
```

Tras el reinicio, hacemos login y verificamos:
```bash
df -h | grep srv
```

Deberíamos ver nuestros 3 volúmenes montados.

### 3.5. Ampliar un volumen lógico

Una de las grandes ventajas de LVM es poder ampliar volúmenes fácilmente. Vamos a ampliar `lv_empresa` en 3GB.

**Paso 1: Verificar espacio disponible en el VG:**
```bash
sudo vgs
```

Deberíamos tener ~5GB libres en `vg_datos`.

**Paso 2: Ampliar el LV:**
```bash
sudo lvextend -L +3G /dev/vg_datos/lv_empresa
```

El `+3G` significa "añadir 3GB al tamaño actual".

Resultado:
```
  Size of logical volume vg_datos/lv_empresa changed from 15.00 GiB (3840 extents) to 18.00 GiB (4608 extents).
  Logical volume vg_datos/lv_empresa successfully resized.
```

**Paso 3: Ampliar el sistema de archivos:**

Hemos ampliado el volumen, pero el sistema de archivos todavía "piensa" que tiene 15GB. Debemos ampliarlo también:

```bash
sudo resize2fs /dev/vg_datos/lv_empresa
```

Resultado:
```
Filesystem at /dev/vg_datos/lv_empresa is mounted on /srv/empresa; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/vg_datos/lv_empresa is now 4718592 blocks long.
```

**Paso 4: Verificar:**
```bash
df -h | grep empresa
```

Resultado:
```
/dev/mapper/vg_datos-lv_empresa  18G  24K  17G   1% /srv/empresa
```

¡Perfecto! El volumen ahora tiene 18GB.

**IMPORTANTE**: Esto lo hemos hecho con el sistema en funcionamiento, sin reiniciar, sin desmontar nada. Esta es la potencia de LVM.

---

## 4. Instalación y configuración básica de Samba

### 4.1. ¿Qué es Samba?

**Samba** es un software que permite que sistemas Linux compartan carpetas e impresoras con sistemas Windows.

Samba implementa el protocolo **SMB/CIFS** (el mismo que usa Windows para compartir recursos). Esto significa que desde un cliente Windows, accederemos a carpetas de Linux exactamente igual que si fueran carpetas compartidas de otro Windows.

### 4.2. Instalar Samba

Instalamos Samba con `apt`:

```bash
sudo apt update
sudo apt install -y samba
```

El `-y` responde automáticamente "yes" a la pregunta de confirmación.

La instalación tardará unos minutos.

Una vez instalado, verificamos que el servicio está activo:
```bash
sudo systemctl status smbd
```

Deberíamos ver:
```
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

La línea `Active: active (running)` confirma que Samba está funcionando.

Si no estuviera activo, lo iniciamos:
```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

### 4.3. Crear una carpeta compartida simple

Antes de complicarnos con Active Directory, vamos a crear una carpeta compartida simple para entender cómo funciona Samba.

#### Paso 1: Crear una carpeta de prueba

```bash
sudo mkdir -p /srv/compartido/prueba
```

Ponemos permisos para que cualquiera pueda escribir (de momento):
```bash
sudo chmod 777 /srv/compartido/prueba
```

#### Paso 2: Configurar Samba

El archivo de configuración de Samba es `/etc/samba/smb.conf`.

Hacemos una copia de seguridad:
```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

Editamos el archivo:
```bash
sudo nano /etc/samba/smb.conf
```

Nos desplazamos al final del archivo (con las flechas o `Ctrl+V` para avanzar páginas).

Añadimos esta configuración:
```ini
[Prueba]
   comment = Carpeta de prueba
   path = /srv/compartido/prueba
   browseable = yes
   read only = no
   guest ok = yes
```

**Explicación:**
- `[Prueba]`: nombre del recurso compartido (se verá así en Windows)
- `comment`: descripción
- `path`: ruta real en Linux
- `browseable`: si aparece al navegar por la red
- `read only = no`: se puede escribir
- `guest ok = yes`: se puede acceder sin autenticación (solo para pruebas)

Guardamos (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### Paso 3: Verificar la configuración

Antes de reiniciar Samba, verificamos que no hay errores:
```bash
testparm
```

Este comando revisa el archivo de configuración. Si hay errores, los mostrará.

Si todo está bien, veremos al final:
```
Loaded services file OK.
```

#### Paso 4: Reiniciar Samba

```bash
sudo systemctl restart smbd
```

Verificamos que sigue activo:
```bash
sudo systemctl status smbd
```

#### Paso 5: Probar desde Windows

Desde un cliente Windows conectado a la misma red que el servidor Linux:

1. Abrimos el Explorador de archivos
2. En la barra de direcciones escribimos: `\\192.168.10.2` (la IP de nuestro servidor Linux)
3. Deberíamos ver la carpeta compartida **Prueba**
4. Entramos y probamos a crear un archivo o carpeta

Si funciona, ¡enhorabuena! Samba está operativo.

---

## 5. Integración con Active Directory

Ahora viene la parte más interesante: vamos a unir nuestro servidor Linux al dominio Active Directory de Windows Server que creamos anteriormente.

De esta forma, los usuarios del dominio Windows podrán autenticarse en el servidor Linux y acceder a recursos compartidos con sus credenciales del dominio.

### 5.1. Requisitos previos

Antes de empezar, verificamos:

**1. Conectividad con el Windows Server:**
```bash
ping -c 4 192.168.10.1
```

**2. Resolución DNS del dominio:**

Nuestro servidor Linux debe poder resolver el nombre del dominio Windows. Para ello, debe usar el Windows Server como DNS.

Editamos la configuración de Netplan para asegurarnos de que usa el Windows Server como DNS:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Verificamos que en `enp0s8` (red departamentos) los `nameservers` incluyen la IP del Windows Server (`192.168.10.1`).

Si hicimos correctamente la configuración anterior, debería estar bien.

Aplicamos la configuración (por si acaso):
```bash
sudo netplan apply
```

**3. Sincronización de hora:**

Para que funcione la autenticación Kerberos (que usa Active Directory), los relojes del servidor Linux y del Windows Server deben estar sincronizados.

Instalamos el cliente NTP:
```bash
sudo apt install -y chrony
```

Verificamos que está sincronizado:
```bash
timedatectl
```

Deberíamos ver `System clock synchronized: yes`.

**4. Instalar paquetes necesarios:**

Necesitamos varios paquetes para unir Linux a Active Directory:

```bash
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

Estos paquetes:
- `realmd`: herramienta para unir al dominio
- `sssd`: demonio de autenticación
- `adcli`: herramientas para Active Directory
- `samba-common-bin`: herramientas de Samba
- `oddjob-mkhomedir`: crea automáticamente carpetas personales

### 5.2. Unir el servidor al dominio

**Paso 1: Descubrir el dominio:**

Primero verificamos que podemos "ver" el dominio:
```bash
sudo realm discover DOMXXX.local
```

(Sustituye `DOMXXX` por el nombre de tu dominio)

Si todo va bien, deberíamos ver información del dominio:
```
DOMXXX.local
  type: kerberos
  realm-name: DOMXXX.LOCAL
  domain-name: DOMXXX.local
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
```

**Paso 2: Unir al dominio:**

Ahora unimos el servidor al dominio. Necesitamos credenciales de un usuario administrador del dominio:

```bash
sudo realm join --user=Administrador DOMXXX.local
```

Nos pedirá la contraseña del Administrador del dominio Windows. La escribimos y pulsamos `Enter`.

Si todo va bien, no mostrará ningún mensaje de error.

Verificamos que estamos unidos:
```bash
sudo realm list
```

Deberíamos ver:
```
DOMXXX.local
  type: kerberos
  realm-name: DOMXXX.LOCAL
  domain-name: domxxx.local
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  ...
```

La línea `configured: kerberos-member` confirma que estamos unidos al dominio.

**Paso 3: Verificar usuarios del dominio:**

Podemos listar usuarios del dominio con:
```bash
sudo getent passwd | grep DOMXXX
```

Deberíamos ver los usuarios del dominio en formato `usuario@domxxx.local`.

**Paso 4: Configurar inicio de sesión:**

Por defecto, para iniciar sesión debemos usar el formato completo: `usuario@domxxx.local`.

Podemos configurar para usar solo el nombre corto. Editamos:
```bash
sudo nano /etc/sssd/sssd.conf
```

Buscamos la sección `[sssd]` y añadimos (si no está):
```ini
[sssd]
domains = DOMXXX.local
config_file_version = 2
services = nss, pam

[domain/DOMXXX.local]
use_fully_qualified_names = False
...
```

La línea clave es `use_fully_qualified_names = False`.

Guardamos y reiniciamos `sssd`:
```bash
sudo systemctl restart sssd
```

**Paso 5: Crear carpeta personal automática:**

Configuramos el sistema para que cree automáticamente la carpeta personal cuando un usuario del dominio inicia sesión por primera vez:

```bash
sudo pam-auth-update --enable mkhomedir
```

Seleccionamos con la barra espaciadora y pulsamos `Enter`.

### 5.3. Configurar Samba como miembro del dominio

Ahora que el servidor está unido al dominio, configuramos Samba para que use autenticación del dominio.

Editamos el archivo de configuración de Samba:
```bash
sudo nano /etc/samba/smb.conf
```

En la sección `[global]` (al principio del archivo), añadimos/modificamos:

```ini
[global]
   workgroup = DOMXXX
   security = ADS
   realm = DOMXXX.LOCAL
   
   # Usar Winbind para autenticación
   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   idmap config DOMXXX : backend = rid
   idmap config DOMXXX : range = 10000-999999
   
   template shell = /bin/bash
   template homedir = /home/%U
   
   # Permitir que usuarios del dominio accedan
   winbind use default domain = yes
   winbind enum users = yes
   winbind enum groups = yes
```

**IMPORTANTE**: Sustituye `DOMXXX` por el nombre de tu dominio (en mayúsculas donde corresponda).

Guardamos el archivo.

Instalamos y configuramos Winbind (componente que conecta Samba con AD):
```bash
sudo apt install -y winbind
```

Reiniciamos servicios:
```bash
sudo systemctl restart smbd nmbd winbind
```

Verificamos que Winbind funciona:
```bash
sudo wbinfo -u
```

Deberíamos ver la lista de usuarios del dominio.

```bash
sudo wbinfo -g
```

Deberíamos ver la lista de grupos del dominio.

### 5.4. Crear recurso compartido accesible para usuarios del dominio

Vamos a crear un recurso compartido en `/srv/compartido` accesible por usuarios del dominio.

Editamos la configuración de Samba:
```bash
sudo nano /etc/samba/smb.conf
```

Eliminamos o comentamos (con `#`) la sección `[Prueba]` que creamos antes.

Añadimos al final:
```ini
[Compartido]
   comment = Carpeta compartida para usuarios del dominio
   path = /srv/compartido
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Trabajadores"
```

**Explicación:**
- `valid users = @"DOMXXX\Trabajadores"`: solo usuarios del grupo "Trabajadores" del dominio pueden acceder
- El `@` indica que es un grupo
- Las comillas son necesarias por la contrabarra

Ajustamos permisos de la carpeta:
```bash
sudo chown -R root:"DOMXXX\domain users" /srv/compartido
sudo chmod -R 770 /srv/compartido
```

Reiniciamos Samba:
```bash
sudo systemctl restart smbd
```

### 5.5. Probar acceso desde Windows

Desde un cliente Windows unido al dominio:

1. Iniciamos sesión con un usuario del dominio (por ejemplo, `falonso` del departamento de Desarrollo)
2. Abrimos el Explorador de archivos
3. Escribimos en la barra de direcciones: `\\192.168.10.2` o `\\SRVXXX_Linux`
4. Deberíamos ver la carpeta **Compartido**
5. Entramos y probamos a crear un archivo

Si funciona, ¡perfecto! La integración es correcta.

Si nos pide credenciales, las proporcionamos (usuario y contraseña del dominio).

---

## Resumen del Tema 7

En este tema hemos:

1. ✅ Instalado Ubuntu Server 22.04 LTS
2. ✅ Configurado red estática con Netplan
3. ✅ Configurado SSH para acceso remoto
4. ✅ Creado estructura de almacenamiento con LVM (PV, VG, LV)
5. ✅ Ampliado un volumen lógico sin parar el sistema
6. ✅ Instalado y configurado Samba
7. ✅ Unido el servidor Linux al dominio Active Directory de Windows
8. ✅ Configurado recursos compartidos accesibles por usuarios del dominio

### Conceptos clave aprendidos

- **Netplan**: configuración de red en Ubuntu Server
- **LVM**: gestión flexible de almacenamiento (PV, VG, LV)
- **Samba**: compartir recursos entre Linux y Windows
- **Realm/SSSD**: unir Linux a Active Directory
- **Integración híbrida**: Windows Server + Linux Server trabajando juntos

---

## Práctica T7 a entregar

[El enunciado completo de la práctica lo tenemos definido anteriormente en el documento]

**Plazo de entrega**: Final de la semana 2

**Formato**: Documento PDF con capturas y explicaciones

---

**¿Dudas?** Consulta al profesor o revisa la documentación en los enlaces proporcionados.

**Próximo tema**: Profundizaremos en la integración Windows-Linux con recursos compartidos avanzados, permisos, clientes Ubuntu Desktop en el dominio, y más.

---

¿Te parece bien este nivel de detalle y estructura? He intentado:

- Ser muy didáctico y paso a paso
- Explicar el "por qué" antes del "cómo"
- Incluir capturas/ejemplos de lo que deberían ver
- Advertir de errores comunes
- No usar iconos
- Mezclar teoría corta con práctica inmediata
- Usar analogías para conceptos complejos

¿Quieres que continúe con el resto del tema o prefieres ajustar algo antes de seguir?











---

# TEMA 7: Linux Server e integración básica con Windows



### Preparación de la máquina virtual

Vamos a crear una VM en VirtualBox para instalar Ubuntu Server.

**Configuración de la VM**:
- Nombre: `SRVXXX_Linux` (XXX = tu nombre)
- Tipo: Linux / Ubuntu (64-bit)
- RAM: 2048 MB (2 GB)
- CPU: 2 procesadores
- Disco: 25 GB (dinámico)

**Red (3 adaptadores)**:
1. **Adaptador 1**: NAT o Puente → salida a Internet
2. **Adaptador 2**: Red interna `red_departamentos` → IP `192.168.10.2/24`
3. **Adaptador 3**: Red interna `red_aula` → IP `172.16.X.2/24` (X = número de tu equipo)

> **IMAGEN SUGERIDA**: Captura de la configuración de red en VirtualBox (3 adaptadores)

### Instalación paso a paso

**Idioma**: Dejamos **English** (más fácil buscar ayuda si hay problemas)

**Teclado**: Spanish / Spanish

**Tipo de instalación**: Ubuntu Server (opción por defecto)

**Red**: De momento no tocamos nada (se configurará después)

**Proxy**: Dejar en blanco

**Mirror**: Dejar por defecto

**Particionado**: 
- Usar **Use an entire disk**
- Marcar **Set up this disk as an LVM group**
- Continuar

> **IMAGEN SUGERIDA**: Captura de la pantalla de particionado

**Usuario administrador**:
```
Your name: Administrador
Server's name: SRVXXX_Linux
Username: admin
Password: (contraseña sencilla que recordéis)
```

⚠️ **IMPORTANTE**: Apuntad este usuario y contraseña.

**SSH**: Marcar **Install OpenSSH server** (lo necesitaremos para administración remota)

**Paquetes adicionales**: No marcar ninguno

**Instalación**: Esperar a que termine (unos minutos)

**Reboot**: Cuando termine, click en **Reboot Now**

> **IMAGEN SUGERIDA**: Captura del login tras la instalación

### Primer acceso

Tras reiniciar, aparece la pantalla de login. Introducimos:
```
login: admin
password: (tu contraseña)
```

Si todo va bien, veremos el **prompt**:
```
admin@SRVXXX_Linux:~$
```

Esto indica:
- `admin`: usuario actual
- `SRVXXX_Linux`: nombre del servidor
- `~`: estamos en nuestra carpeta personal
- `$`: usuario normal (no root)

> **IMAGEN SUGERIDA**: Captura del prompt tras login exitoso

### Primeros comandos

Vamos a familiarizarnos con comandos básicos:

**Ver directorio actual**:
```bash
pwd
```
Resultado: `/home/admin`

**Listar archivos**:
```bash
ls
ls -la
```

**Ver tarjetas de red**:
```bash
ip addr
```
Deberíamos ver 3 interfaces: `enp0s3`, `enp0s8`, `enp0s9` (los nombres pueden variar ligeramente).

**Probar Internet**:
```bash
ping -c 4 google.com
```
El `-c 4` hace 4 pings y para.

**Actualizar el sistema**:
```bash
sudo apt update
sudo apt upgrade -y
```

El comando `sudo` nos pide la contraseña (no se ve al escribir, es normal). Permite ejecutar comandos como administrador.

> **IMAGEN SUGERIDA**: Captura mostrando salida de `ip addr` con las 3 interfaces

---

## 2. Configuración de red con Netplan

### Por qué necesitamos IPs fijas

Un servidor debe tener **IP estática** (fija). Si cambiara cada vez que se reinicia, los clientes no sabrían dónde encontrarlo.

### Netplan: el gestor de red

Ubuntu Server usa **Netplan** para configurar la red. Los archivos de configuración están en `/etc/netplan/` en formato YAML.

Ver archivo actual:
```bash
ls /etc/netplan/
sudo cat /etc/netplan/00-installer-config.yaml
```

Veremos que todas las interfaces están en DHCP (configuración automática).

### Configurar IPs estáticas

Vamos a configurar IPs fijas en las redes internas, dejando la primera con DHCP para Internet.

**Esquema de red**:
- `enp0s3`: DHCP (Internet)
- `enp0s8`: `192.168.10.2/24` (red departamentos, gateway `192.168.10.1`)
- `enp0s9`: `172.16.X.2/24` (red aula, X = tu número de equipo)

⚠️ **IMPORTANTE**: Verifica los nombres de tus interfaces con `ip addr`. Pueden ser diferentes.

**Editar configuración**:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Borrar todo y escribir:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.10.2/24
      routes:
        - to: default
          via: 192.168.10.1
          metric: 200
      nameservers:
        addresses:
          - 192.168.10.1
          - 8.8.8.8
    enp0s9:
      addresses:
        - 172.16.X.2/24
      nameservers:
        addresses:
          - 192.168.10.1
```

⚠️ **CRÍTICO**: En YAML, la indentación es con **2 espacios**, NO tabuladores.

Guardar: `Ctrl+O`, `Enter`, `Ctrl+X`

> **IMAGEN SUGERIDA**: Captura del archivo Netplan correctamente indentado

**Aplicar configuración**:
```bash
sudo netplan try
```

Este comando aplica temporalmente y pregunta si funciona (medida de seguridad). Pulsar `Enter` para aceptar.

**Verificar**:
```bash
ip addr show
ping -c 4 google.com
ping -c 4 192.168.10.1
```

### Configurar nombre del servidor

```bash
sudo hostnamectl set-hostname SRVXXX_Linux
```

Editar `/etc/hosts`:
```bash
sudo nano /etc/hosts
```

Debe contener:
```
127.0.0.1 localhost
127.1.1 SRVXXX_Linux
```

Guardar y reiniciar:
```bash
sudo reboot
```
