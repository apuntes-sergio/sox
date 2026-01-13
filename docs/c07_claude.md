# TEMA 7: Linux Server e integración básica con Windows

## Introducción: Redes heterogéneas

Hasta ahora hemos trabajado exclusivamente con Windows Server. Hemos montado un dominio, compartido recursos y aplicado directivas en un entorno completamente Microsoft. Sin embargo, cuando visitéis empresas medianas o grandes, os daréis cuenta de que lo habitual es encontrar **Windows y Linux trabajando juntos**. A esto se le llama una **red heterogénea**.

### Por qué mezclar Windows y Linux

Las empresas no mezclan sistemas operativos por capricho. Existen razones muy concretas que justifican esta decisión:

**Ahorro de costes**: Windows Server requiere licencias que cuestan cientos de euros por servidor y por cliente. Linux, en cambio, es gratuito. Una empresa con 10 servidores puede ahorrarse miles de euros anuales usando Linux donde sea técnicamente viable.

**Cada sistema tiene sus fortalezas**: 

Windows destaca en gestión centralizada mediante Active Directory, integración perfecta con Office y Exchange, y soporte nativo para aplicaciones empresariales que solo existen en este ecosistema (por ejemplo, muchas aplicaciones de gestión empresarial, contabilidad o ERP).

Linux destaca en servidores web (la mayoría de sitios web del mundo corren en Linux), estabilidad excepcional (servidores que funcionan meses sin reiniciar), mayor seguridad por diseño y rendimiento óptimo incluso con recursos limitados.

**Aplicaciones modernas**: Muchas tecnologías actuales como Docker, Kubernetes y la mayoría de frameworks de desarrollo web están optimizadas para Linux. Si una empresa quiere usar estas tecnologías, necesita servidores Linux en su infraestructura.

### El reto: que todo funcione junto

El desafío no es técnico en sí mismo, sino de integración. Los usuarios finales no deben notar que existen sistemas diferentes trabajando detrás. Cuando un empleado accede a una carpeta compartida, no debería importarle si está físicamente en Windows o en Linux. Debe poder usar las mismas credenciales para todo y el sistema debe comportarse de manera uniforme.

Para conseguir esta transparencia usamos **protocolos estándar** que ambos sistemas entienden:

- **SMB/CIFS**: protocolo para compartir archivos. Windows lo usa nativamente y en Linux lo implementamos mediante Samba.

- **LDAP**: protocolo de directorios de usuarios que permite consultar y gestionar información de autenticación de forma centralizada.

- **Kerberos**: sistema de autenticación segura que usa tokens cifrados para verificar identidades.

- **DNS**: sistema de resolución de nombres que permite usar nombres de equipo en lugar de direcciones IP.

---

## Linux: conceptos básicos

### Qué es Linux

Linux no es un sistema operativo completo, es un **kernel** (núcleo del sistema). Las **distribuciones** toman ese kernel y le añaden todo lo necesario: herramientas del sistema, gestores de paquetes, servicios, interfaces gráficas, etc.

Nosotros usaremos **Ubuntu Server 22.04 LTS** por varias razones prácticas:

Es completamente gratuita y tiene soporte oficial hasta 2027. Cuenta con una comunidad enorme, lo que facilita encontrar ayuda y documentación. Es la distribución más usada en entornos cloud (AWS, Azure, Google Cloud), por lo que lo que aprendáis aquí os servirá en entornos profesionales reales. Además, es especialmente adecuada para aprender por su facilidad de uso y abundante documentación.

### Diferencias clave con Windows Server

**Interfaz de administración**: Windows Server tiene ventanas, menús y herramientas gráficas. Linux Server se administra principalmente por **terminal** (línea de comandos). Aunque pueda parecer más difícil al principio, consume muchísimos menos recursos y facilita enormemente la automatización de tareas mediante scripts.

**Sistema de archivos**: 

En Windows tenemos letras de unidad separadas (C:, D:, E:) para cada disco o partición. En Linux existe un único árbol jerárquico que empieza en `/` (raíz). Los discos adicionales se "montan" como carpetas dentro de este árbol.

Por ejemplo, en Windows tendríamos `D:\datos`. En Linux sería `/datos` y el disco físico se monta en esa ubicación.

**Instalación de programas**:

En Windows descargamos ejecutables .exe y hacemos doble clic. En Linux usamos el **gestor de paquetes** `apt`, que funciona como una tienda de aplicaciones pero mucho más potente.

```bash
sudo apt install nombre_programa
```

Este comando descarga el programa, lo instala, configura dependencias y lo deja listo para usar, todo automáticamente.

**Usuarios y permisos**: 

Linux es multiusuario desde su diseño original. Existe un superusuario llamado **root** (equivalente a Administrador en Windows), pero por seguridad nunca trabajamos directamente como root. Usamos el comando `sudo` para ejecutar comandos específicos con privilegios elevados.

**Configuración del sistema**: 

En Windows la configuración está en el Registro, una base de datos binaria compleja. En Linux, toda la configuración son **archivos de texto plano** ubicados principalmente en `/etc/`. Esto tiene enormes ventajas: puedes editarlos con cualquier editor, copiarlos fácilmente, hacer backups simples y compartirlos entre sistemas.

---

## Qué vamos a hacer en este tema

Vamos a incorporar un servidor Linux a nuestra red ISCASOX. No será un sistema aislado, sino completamente integrado con el dominio Windows desde el principio.

El tema se estructura en cuatro bloques principales:

**Bloque 1: Instalación y configuración básica**

Instalaremos Ubuntu Server en VirtualBox, configuraremos una dirección IP estática usando Netplan (el gestor de red de Ubuntu) y habilitaremos SSH para poder administrar el servidor remotamente desde nuestro equipo Windows.

**Bloque 2: LVM (Logical Volume Manager)**

Aprenderemos a gestionar el almacenamiento de forma flexible. Crearemos, ampliaremos y gestionaremos volúmenes lógicos sin necesidad de reiniciar el servidor ni interrumpir servicios.

**Bloque 3: Samba**

Instalaremos y configuraremos Samba, el software que permite que Linux "hable" el protocolo SMB de Windows. Crearemos recursos compartidos básicos y probaremos el acceso desde clientes Windows.

**Bloque 4: Integración con Active Directory**

Esta es la parte más interesante: uniremos el servidor Linux al dominio Windows. Los usuarios del Active Directory podrán autenticarse en Linux y los recursos compartidos respetarán los permisos establecidos en el AD.

Al final de este tema tendremos una infraestructura híbrida completamente funcional, similar a las que encontraréis en empresas reales.

---

!!!top "Cómo trabajar en este tema"

    **En clase** explicaremos los conceptos fundamentales y haremos demostraciones en vivo de los procesos más complejos.

    **En los apuntes** encontraréis guías paso a paso muy detalladas que podréis seguir a vuestro ritmo.

    **Aspectos importantes a tener en cuenta**:

    Haced **snapshots** (instantáneas) de la máquina virtual antes de realizar cambios importantes. Si algo falla, simplemente restauráis la instantánea y volvéis a intentarlo sin perder todo el trabajo anterior.

    No tengáis miedo a "romper" el sistema. Estáis trabajando en máquinas virtuales, así que experimentad tranquilamente. Los errores son parte del aprendizaje.

    **No copiéis comandos sin leer qué hacen**. Es fundamental que entendáis cada comando antes de ejecutarlo. Los errores en Linux suelen ser muy informativos si os tomáis el tiempo de leerlos.

    **Cuando algo no funcione**, seguid estos pasos:

    Primero, leed el mensaje de error completo, palabra por palabra. Segundo, buscad el error exacto en Google (copiad el mensaje tal cual). Tercero, consultad al profesor si seguís atascados después de intentar solucionarlo.

    **Herramientas de ayuda**: Linux tiene sistemas de ayuda integrados muy potentes:

    ```bash
    man nombre_comando    # Manual completo y detallado del comando
    comando --help        # Ayuda rápida con las opciones principales
    ```

    Existen también herramientas gráficas opcionales como **Webmin** o **Cockpit** que proporcionan paneles web de administración. No son obligatorias, pero pueden ayudar cuando estáis empezando a familiarizaros con el sistema.

---

## 1. Instalación de Ubuntu Server

### Preparación de la máquina virtual

Vamos a crear una nueva máquina virtual en VirtualBox específicamente para Ubuntu Server. La configuración debe ser la siguiente:

**Parámetros básicos**:

- Nombre: `SERVERXXX_Linux` (donde XXX es tu nombre)
- Tipo: Linux
- Versión: Ubuntu (64-bit)
- Memoria RAM: 2048 MB (2 GB)
- Procesadores: 2 CPUs
- Disco duro: 25 GB (reservado dinámicamente)

**Configuración de red**:

Para simplificar el montaje, vamos a usar una única tarjeta de red que conectará el servidor Linux directamente a la red de departamentos de nuestra empresa ISCASOX.

Adaptador 1: Red interna `red_departamentos`. IP estática `192.168.100.5/24`. Esta red conecta con el Windows Server (`192.168.100.1`) y con todos los departamentos de la empresa (gestión, taller, comercial, desarrollo).

Esta configuración es suficiente porque el Windows Server ya tiene salida a Internet y actúa como gateway. El servidor Linux podrá comunicarse con Internet a través del Windows Server.

**Añadir el ISO de instalación**:

Descarga Ubuntu Server 22.04 LTS desde la [página oficial de Ubuntu](https://releases.ubuntu.com/jammy/). En la configuración de la VM, en la sección Almacenamiento, añadid el archivo ISO descargado como disco óptico.

### Proceso de instalación paso a paso

Una vez creada la máquina virtual, la iniciamos. Arrancará automáticamente desde el ISO y comenzará el instalador.

**Pantalla de idioma**: Seleccionamos **English**. Aunque sea en inglés, es más fácil buscar ayuda en Internet cuando aparecen mensajes en este idioma. Pulsamos Enter.

**Configuración del teclado**: 

Layout: **Spanish**

Variant: **Spanish**

Navegamos con las flechas del teclado y seleccionamos con Enter. Pulsamos "Done" para continuar.

**Tipo de instalación**: Dejamos seleccionado **Ubuntu Server** (es la opción por defecto). Pulsamos Done.

**Configuración de red**: 

Aquí veremos la tarjeta de red que configuramos en VirtualBox. De momento tendrá configuración automática o sin configurar. No os preocupéis, esto lo arreglaremos después de la instalación. No tocamos nada y pulsamos Done.

**Configuración de proxy**: Dejamos el campo en blanco (no necesitamos proxy) y pulsamos Done.

**Mirror de Ubuntu**: Dejamos la URL por defecto y pulsamos Done.

**Configuración del almacenamiento**:

Esta es una de las pantallas más importantes. Aquí decidimos cómo particionar el disco. Para esta instalación inicial vamos a usar el particionado automático. Más adelante añadiremos discos adicionales y configuraremos LVM manualmente.

Dejamos seleccionado **Use an entire disk** y **Set up this disk as an LVM group**. Pulsamos Done.

Nos mostrará un resumen del particionado propuesto. Veremos algo como:

```
USED DEVICES
ubuntu-lv     /     12.000G
```

Pulsamos Done y nos preguntará si estamos seguros. Seleccionamos Continue para proceder.

**Configuración de perfil**:

Aquí configuramos el usuario administrador principal del sistema. Este usuario tendrá permisos para usar `sudo` y administrar el servidor.

```
Your name: Administrador
Your server's name: SRVXXX_Linux  (XXX = tu nombre)
Pick a username: admin
Choose a password: (contraseña sencilla que recordéis)
Confirm your password: (repetir contraseña)
```

**IMPORTANTE**: Apuntad este usuario y contraseña en algún lugar seguro. Lo necesitaréis constantemente.

**Configuración SSH**:

SSH (Secure Shell) es el protocolo que usaremos para conectarnos remotamente al servidor desde Windows. Es absolutamente fundamental instalarlo.

Marcamos con la barra espaciadora la opción **Install OpenSSH server**. Dejamos desmarcado "Import SSH identity". Pulsamos Done.

**Paquetes adicionales**: 

El instalador nos ofrece instalar algunos paquetes populares. De momento no instalamos ninguno. Pulsamos Done sin marcar nada.

**Instalación**: 

Comenzará el proceso de instalación. Tardará varios minutos. Podemos ver el progreso en pantalla con mensajes como "Installing system", "Configuring apt", etc.

**Reinicio**: 

Cuando termine la instalación aparecerá un botón "Reboot Now". Pulsamos sobre él. Si nos pide que quitemos el medio de instalación, simplemente pulsamos Enter. El sistema se reiniciará automáticamente.

### Primer acceso al sistema

Tras el reinicio, aparecerá una pantalla de login en modo texto:

```
Ubuntu 22.04.X LTS SRVXXX_Linux tty1

SRVXXX_Linux login: _
```

Introducimos nuestras credenciales:

```
login: admin
password: (nuestra contraseña)
```

La contraseña no se ve al escribir, es una medida de seguridad. Es completamente normal.

Si todo va bien, veremos la pantalla de bienvenida y el prompt del sistema:

```
Welcome to Ubuntu 22.04.X LTS (GNU/Linux 5.15.0-XX-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: ...
admin@SRVXXX_Linux:~$ _
```

Este es el **prompt** de Linux. Nos proporciona información importante:

- **admin**: el usuario con el que estamos trabajando
- **SRVXXX_Linux**: nombre del servidor
- **~**: indica que estamos en nuestra carpeta personal (`/home/admin`)
- **$**: símbolo que indica que somos un usuario normal (no root). Si fuéramos root, aparecería `#`

### Comandos básicos para orientarnos

Vamos a ejecutar algunos comandos básicos para familiarizarnos con el sistema y verificar que todo funciona correctamente.

**Ver en qué directorio estamos**:

```bash
pwd
```

Resultado: `/home/admin`

Este comando significa "Print Working Directory" (imprimir directorio de trabajo). Siempre nos dice dónde estamos en el árbol de directorios.

**Listar archivos del directorio actual**:

```bash
ls
```

Probablemente no veamos nada porque el directorio está vacío al ser recién creado.

**Listar archivos con detalles**:

```bash
ls -la
```

La opción `-l` muestra formato largo con permisos, propietario, tamaño y fecha. La opción `-a` muestra también archivos ocultos (los que empiezan por punto).

**Ver información del sistema**:

```bash
uname -a
```

Muestra información sobre el kernel: versión, arquitectura, nombre del host, etc.

**Ver información de red**:

```bash
ip addr
```

Este comando es fundamental. Muestra todas las interfaces de red. Deberíamos ver algo similar a:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.10.X/24 brd 192.168.10.255 scope global dynamic enp0s3
```

Aquí vemos:

- `lo`: interfaz de loopback local (siempre presente)
- `enp0s3`: nuestra interfaz de red (puede que aún sin configurar o con IP temporal)

El nombre exacto (`enp0s3`, `eth0`, `ens33`, etc.) puede variar según la configuración de VirtualBox y el hardware virtualizado.

**Actualizar el sistema**:

Es buena práctica actualizar el sistema recién instalado para tener todas las últimas correcciones de seguridad:

```bash
sudo apt update
```

La primera vez que usamos `sudo` nos pedirá la contraseña de nuestro usuario `admin`. La escribimos (no se verá nada al escribir, es normal por seguridad) y pulsamos Enter.

Este comando actualiza la lista de paquetes disponibles desde los repositorios.

```bash
sudo apt upgrade -y
```

Este comando instala las actualizaciones disponibles. El parámetro `-y` responde automáticamente "yes" a todas las preguntas de confirmación.

Puede tardar varios minutos. Es normal. Esperamos pacientemente a que termine.

### Qué es sudo

En Linux, las tareas de administración del sistema requieren permisos de **root** (el superusuario, equivalente a "Administrador" en Windows).

Por razones de seguridad, no trabajamos directamente como root. En su lugar, usamos el comando `sudo` (que significa "SuperUser DO") delante de comandos que requieren permisos elevados.

Cuando usamos `sudo`, el sistema:

1. Nos pide la contraseña de nuestro usuario actual
2. Verifica que nuestro usuario tiene permisos para usar sudo
3. Ejecuta el comando con privilegios de root
4. Registra la acción en los logs del sistema

La primera vez que usamos `sudo` en una sesión pide contraseña. Durante los siguientes 15 minutos no volverá a pedirla (por comodidad). Después de ese tiempo, volverá a solicitarla.

---

## 2. Configuración de red estática con Netplan

### Por qué configuración estática

Cuando instalamos Ubuntu Server, la tarjeta de red se configura automáticamente mediante DHCP si hay un servidor DHCP disponible, o queda sin configurar si no lo hay.

Para un servidor, es imprescindible que la dirección IP sea **fija** (estática). No puede cambiar cada vez que se reinicia el sistema. Imaginad si cada vez que el servidor se reinicia su IP cambia: los clientes no sabrían dónde encontrarlo, las configuraciones de red fallarían y todo dejaría de funcionar.

### Netplan: el gestor de red de Ubuntu

Ubuntu Server usa **Netplan** para configurar la red. Netplan es un sistema que:

- Lee archivos de configuración en formato YAML
- Genera la configuración para el gestor de red subyacente (NetworkManager o systemd-networkd)
- Aplica los cambios de forma consistente

Los archivos de configuración de Netplan están en el directorio `/etc/netplan/`.

Veamos qué archivos hay:

```bash
ls /etc/netplan/
```

Normalmente encontraremos un archivo llamado `00-installer-config.yaml` o algo similar. Los números al principio determinan el orden de aplicación si hay varios archivos.

Veamos su contenido actual:

```bash
sudo cat /etc/netplan/00-installer-config.yaml
```

Veremos algo parecido a esto:

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```

Esto significa que la interfaz está configurada para obtener IP automáticamente mediante DHCP.

### Configurar IP estática

Vamos a modificar este archivo para configurar una IP estática en nuestra interfaz de red.

**Esquema de red a configurar**:

Según el diseño de nuestra red ISCASOX, la configuración quedará así:

**enp0s3** (única interfaz - red departamentos):
- IP: `192.168.10.2/24`
- Gateway: `192.168.10.1` (Windows Server)
- DNS: `192.168.10.1`, `8.8.8.8`

El Windows Server actuará como puerta de enlace para dar salida a Internet a nuestro servidor Linux.

**IMPORTANTE**: El nombre de la interfaz puede variar en vuestro sistema. Usad el comando `ip addr` para verificar el nombre real y ajustad la configuración en consecuencia.

### Editar el archivo de configuración

Para editar archivos en Linux desde la terminal usamos editores de texto. Los más comunes son:

- **nano**: más sencillo e intuitivo, ideal para principiantes
- **vi** o **vim**: más potente pero con curva de aprendizaje pronunciada

Usaremos **nano** porque es más fácil de usar:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Nos pedirá la contraseña de sudo si han pasado más de 15 minutos desde la última vez que lo usamos.

El editor nano se abrirá mostrando el contenido actual del archivo. En la parte inferior veremos atajos de teclado (el símbolo `^` representa la tecla Ctrl).

Borramos todo el contenido actual (con Supr o Backspace) y escribimos la siguiente configuración:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 192.168.10.2/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 192.168.10.1
          - 8.8.8.8
```

**IMPORTANTE - Aspectos críticos de YAML**:

En YAML, la **indentación es absolutamente crítica**. El formato usa espacios para indicar jerarquía.

Debe indentarse con **espacios**, NUNCA con tabuladores. Si usáis tabuladores, dará error.

Cada nivel de indentación son exactamente **2 espacios**.

Los dos puntos (`:`) después de cada clave son obligatorios.

Las listas se indican con guiones (`-`).

Verificad cuidadosamente que el nombre de vuestra interfaz coincide con el que mostró `ip addr`. Si en vuestro sistema la interfaz se llama `ens33`, `eth0`, etc., debéis usar ese nombre.

**Explicación de la configuración**:

- `addresses`: la IP estática que asignamos al servidor
- `routes`: definimos la ruta por defecto (gateway) hacia el Windows Server
- `nameservers`: servidores DNS que usará el sistema (primero el Windows Server, luego Google DNS como backup)

### Guardar el archivo

Para guardar los cambios en nano:

1. Pulsamos `Ctrl + O` (letra O, de "Output" o salida)
2. Nos preguntará el nombre del archivo. Como no lo cambiamos, simplemente pulsamos `Enter`
3. Para salir del editor: `Ctrl + X`

### Aplicar la configuración

Antes de aplicar la configuración definitivamente, Netplan nos permite probarla de forma segura:

```bash
sudo netplan try
```

Este comando es muy inteligente. Aplica la configuración temporalmente y nos pregunta si funciona. Si no respondemos en 120 segundos, **revierte automáticamente** los cambios. Esto es una medida de seguridad fundamental para no quedarnos sin acceso al servidor.

Si todo va bien, veremos:

```
Do you want to keep these settings?

Press ENTER before the timeout to accept the new configuration

Changes will revert in 120 seconds
Configuration accepted.
```

Pulsamos Enter para aceptar definitivamente la nueva configuración.

**Si hay errores de sintaxis**, nos los mostrará. Los errores más comunes son:

- Indentación incorrecta (usar tabuladores en vez de espacios, o número incorrecto de espacios)
- Nombre de interfaz incorrecto
- Falta de dos puntos (`:`) después de las claves YAML
- Guiones (`-`) mal colocados en las listas

Si hay error, el comando no aplicará los cambios. Debemos volver a editar el archivo y corregir.

### Verificar la configuración

Una vez aplicada correctamente, verificamos que la IP se ha asignado:

```bash
ip addr show
```

Deberíamos ver nuestra interfaz con la dirección IP configurada:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.10.2/24 brd 192.168.10.255 scope global enp0s3
```

Verificamos que podemos comunicarnos con el Windows Server:

```bash
ping -c 4 192.168.10.1
```

Si el Windows Server está configurado correctamente como gateway, deberíamos tener también acceso a Internet:

```bash
ping -c 4 google.com
```

Si ambos funcionan, ¡perfecto! La red está correctamente configurada.

**Nota**: Si el ping a Internet no funciona, aseguraos de que el Windows Server tiene configurado correctamente el enrutamiento y el NAT para dar salida a Internet a las redes internas.

### Configurar el nombre del servidor (hostname)

El nombre del servidor (hostname) se configura con el comando `hostnamectl`:

```bash
sudo hostnamectl set-hostname SRVXXX_Linux
```

Para verificar que se ha cambiado correctamente:

```bash
hostnamectl
```

Veremos información detallada del sistema incluyendo el nuevo hostname.

**IMPORTANTE**: También debemos editar el archivo `/etc/hosts` para que el sistema resuelva correctamente su propio nombre:

```bash
sudo nano /etc/hosts
```

El archivo debe contener al menos estas líneas:

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

La línea importante es `127.0.1.1 SRVXXX_Linux`. Aseguraos de que coincide con vuestro hostname.

Guardamos el archivo (`Ctrl+O`, `Enter`, `Ctrl+X`).

Para que todos los cambios se apliquen completamente, reiniciamos el servidor:

```bash
sudo reboot
```

El sistema se reiniciará. Esperamos unos 30 segundos y volvemos a hacer login. El prompt debería mostrar ya el nuevo nombre del servidor:

```
admin@SRVXXX_Linux:~$
```

---

## 3. Gestión avanzada de almacenamiento con LVM

### Qué es LVM y por qué usarlo

**LVM** (Logical Volume Manager) es un sistema de gestión de almacenamiento que añade una capa de abstracción entre los discos físicos y el sistema de archivos. Esta abstracción nos proporciona una flexibilidad extraordinaria que no existe con las particiones tradicionales.

#### Comparación: Particiones tradicionales vs LVM

**Con particiones tradicionales**:

Cuando creamos una partición, definimos su tamaño de forma fija en el disco. Si esa partición se queda sin espacio más adelante, ampliarla es extremadamente complicado y arriesgado. Normalmente requiere:

- Parar el sistema completamente
- Redimensionar la partición con herramientas especiales
- Rezar para que no haya corrupción de datos
- Reiniciar el sistema

No podemos "mover" espacio libre de una partición a otra fácilmente. Si tenemos 50GB libres en una partición y otra se ha quedado sin espacio, no hay forma sencilla de transferir ese espacio.

Tampoco podemos combinar varios discos físicos para formar una única partición grande. Cada disco es independiente.

**Con LVM**:

Podemos **ampliar** volúmenes fácilmente, incluso con el sistema en funcionamiento y sin parar servicios. Los usuarios ni se enterarán.

Podemos **reducir** volúmenes si necesitamos recuperar espacio para asignarlo a otro volumen.

Podemos **combinar** varios discos físicos en un único espacio de almacenamiento lógico.

Podemos crear **snapshots** (copias instantáneas) de volúmenes, ideales para backups o pruebas.

Podemos **mover** datos entre discos sin detener el sistema ni afectar a las aplicaciones.

**Analogía práctica**: 

Las particiones tradicionales son como habitaciones de una casa con paredes de hormigón. Una vez construidas, cambiar su tamaño implica derribar paredes, lo cual es costoso y arriesgado.

LVM es como una oficina moderna con paneles modulares. Puedes mover los "tabiques" fácilmente, combinar espacios, dividirlos o reorganizarlos según las necesidades cambien, todo sin obras mayores.

En Windows Server existe algo similar llamado **Storage Spaces** o **Espacios de almacenamiento**, aunque LVM es más maduro, estable y potente al tener décadas de desarrollo.

### Conceptos fundamentales de LVM

LVM funciona con tres niveles jerárquicos que debemos entender claramente:

**1. PV (Physical Volume - Volumen Físico)**:

Es la base de todo. Un PV es un disco duro físico (o partición) que hemos "preparado" para que LVM pueda trabajar con él. Básicamente le decimos a LVM: "este disco está disponible para que lo gestiones".

Comandos principales:
- `pvcreate`: convierte un disco en PV
- `pvs` o `pvdisplay`: muestra información de los PVs

**2. VG (Volume Group - Grupo de Volúmenes)**:

Es un "contenedor" que agrupa uno o varios PVs. Pensad en el VG como una "bolsa común de espacio" donde depositamos todos nuestros discos. Este espacio conjunto luego lo repartiremos como queramos.

Por ejemplo, si tenemos tres discos de 10GB cada uno, los agrupamos en un VG y tenemos 30GB de espacio total para trabajar.

Comandos principales:
- `vgcreate`: crea un nuevo grupo de volúmenes
- `vgs` o `vgdisplay`: muestra información de los VGs

**3. LV (Logical Volume - Volumen Lógico)**:

Es el "volumen" que usaremos realmente. Los LV se crean dentro de un VG, tomando el espacio que necesitemos de la "bolsa común". Este es el que finalmente montaremos como carpeta en el sistema y donde guardaremos datos.

Por ejemplo, de nuestros 30GB totales del VG, podemos crear:
- Un LV de 15GB para datos de empresa
- Un LV de 10GB para carpetas de usuarios
- Un LV de 5GB para backups

Comandos principales:
- `lvcreate`: crea un nuevo volumen lógico
- `lvs` o `lvdisplay`: muestra información de los LVs

**Diagrama conceptual**:

```
Discos físicos:  [Disco 10GB] [Disco 10GB] [Disco 10GB]
                    ↓            ↓            ↓
                    └───────── PVs ───────────┘
                              ↓
                    [VG: vg_datos - 30GB]
                              ↓
              ┌───────────────┴───────────────┐
              ↓               ↓               ↓
      [LV: lv_empresa]  [LV: lv_usuarios]  [LV: lv_backup]
           15GB              10GB              5GB
              ↓               ↓               ↓
        /srv/empresa    /srv/usuarios    /srv/backup
```

### Añadir discos a la máquina virtual

Antes de poder trabajar con LVM, necesitamos añadir discos adicionales a nuestra máquina virtual. Vamos a añadir tres discos de 10GB cada uno.

**Apagar la máquina virtual**:

Desde la terminal del servidor ejecutamos:

```bash
sudo poweroff
```

El sistema se apagará de forma ordenada, cerrando todos los servicios correctamente.

**Añadir 3 discos en VirtualBox**:

Con la VM completamente apagada (no pausada, apagada), en VirtualBox:

1. Seleccionamos nuestra VM `SRVXXX_Linux`
2. Click en **Configuración**
3. Navegamos a la sección **Almacenamiento**
4. En el controlador **Controladora: SATA**, hacemos click en el icono de disco con el símbolo `+` (Añadir disco duro)
5. Click en **Crear**
6. Configuramos el nuevo disco:
   - Tipo de archivo: **VDI (VirtualBox Disk Image)**
   - Almacenamiento: **Reservado dinámicamente** (solo usa espacio real cuando se escribe en él)
   - Tamaño: **10 GB**
   - Nombre: dejamos el sugerido (VirtualBox le pondrá un nombre automático)
7. Click en **Crear**
8. Repetimos todo el proceso 2 veces más para tener un total de **3 discos nuevos de 10GB**

Al finalizar, en la sección de almacenamiento de la controladora SATA deberíamos ver:

- `SRVXXX_Linux.vdi` (disco principal del sistema, 25GB)
- `SRVXXX_Linux_1.vdi` (disco adicional 1, 10GB)
- `SRVXXX_Linux_2.vdi` (disco adicional 2, 10GB)
- `SRVXXX_Linux_3.vdi` (disco adicional 3, 10GB)

Hacemos click en **Aceptar** para guardar todos los cambios.

**Iniciar la VM y verificar discos**:

Iniciamos la máquina virtual normalmente y hacemos login con nuestro usuario `admin`.

Verificamos que el sistema ha detectado los nuevos discos:

```bash
lsblk
```

Este comando lista todos los dispositivos de bloque (discos) disponibles en el sistema. Deberíamos ver algo similar a:

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

**Interpretación de la salida**:

- `sda`: nuestro disco principal (25GB) donde ya está instalado Ubuntu. Vemos sus particiones (`sda1`, `sda2`, `sda3`)
- `sdb`, `sdc`, `sdd`: los tres discos nuevos de 10GB que acabamos de añadir. Aparecen sin particiones ni formato, completamente vírgenes.

**Nota importante**: Los nombres de los discos (`sda`, `sdb`, etc.) pueden variar ligeramente según la configuración. En algunos sistemas pueden aparecer como `vda`, `vdb`, etc. Lo importante es identificar visualmente los tres discos de 10GB que acabamos de añadir.

### Crear la estructura LVM paso a paso

Ahora vamos a crear nuestra estructura LVM completa, nivel por nivel, de forma muy detallada.

#### Paso 1: Crear Physical Volumes (PVs)

El primer paso es convertir nuestros tres discos físicos en volúmenes físicos que LVM pueda gestionar:

```bash
sudo pvcreate /dev/sdb /dev/sdc /dev/sdd
```

Deberíamos ver una confirmación para cada disco:

```
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
```

Verificamos que se han creado correctamente:

```bash
sudo pvs
```

La salida mostrará:

```
  PV         VG        Fmt  Attr PSize  PFree 
  /dev/sda3  ubuntu-vg lvm2 a--  23.00g    0 
  /dev/sdb             lvm2 ---  10.00g 10.00g
  /dev/sdc             lvm2 ---  10.00g 10.00g
  /dev/sdd             lvm2 ---  10.00g 10.00g
```

**Interpretación**:

- `/dev/sda3`: el PV que se creó automáticamente durante la instalación para el sistema
- `/dev/sdb`, `/dev/sdc`, `/dev/sdd`: nuestros tres PVs nuevos, todavía sin asignar a ningún VG (columna VG vacía)
- La columna `PFree` muestra que los 10GB de cada disco están completamente disponibles

#### Paso 2: Crear Volume Group (VG)

Ahora agrupamos nuestros tres PVs en un único grupo llamado `vg_datos`:

```bash
sudo vgcreate vg_datos /dev/sdb /dev/sdc /dev/sdd
```

Confirmación:

```
  Volume group "vg_datos" successfully created
```

Verificamos el grupo creado:

```bash
sudo vgs
```

Salida:

```
  VG        #PV #LV #SN Attr   VSize  VFree 
  ubuntu-vg   1   1   0 wz--n- 23.00g     0 
  vg_datos    3   0   0 wz--n- 29.99g 29.99g
```

**Interpretación**:

- `vg_datos`: nuestro nuevo grupo de volúmenes
- `#PV`: 3 (contiene 3 volúmenes físicos)
- `#LV`: 0 (todavía no hemos creado volúmenes lógicos dentro)
- `VSize`: ~30GB (10GB × 3 discos, menos un pequeño espacio para metadatos de LVM)
- `VFree`: ~30GB (todo el espacio está disponible)

Podemos ver más detalles con:

```bash
sudo vgdisplay vg_datos
```

Esto muestra información completa sobre el grupo, incluyendo el tamaño exacto de las unidades de asignación (PE - Physical Extents), cuántas hay libres, etc.

#### Paso 3: Crear Logical Volumes (LVs)

Ahora creamos tres volúmenes lógicos dentro de `vg_datos`. Estos serán los "discos virtuales" que montaremos y usaremos realmente.

**Volumen para datos de empresa (15GB)**:

```bash
sudo lvcreate -L 15G -n lv_empresa vg_datos
```

**Explicación del comando**:
- `-L 15G`: tamaño del volumen (15 gigabytes)
- `-n lv_empresa`: nombre del volumen lógico
- `vg_datos`: grupo de volúmenes donde se crea

Confirmación:

```
  Logical volume "lv_empresa" created.
```

**Volumen para carpetas de usuarios (10GB)**:

```bash
sudo lvcreate -L 10G -n lv_usuarios vg_datos
```

**Volumen para datos compartidos (5GB)**:

```bash
sudo lvcreate -L 5G -n lv_compartido vg_datos
```

Verificamos todos los volúmenes creados:

```bash
sudo lvs
```

Salida:

```
  LV            VG        Attr       LSize  
  ubuntu-lv     ubuntu-vg -wi-ao---- 11.00g
  lv_compartido vg_datos  -wi-a-----  5.00g
  lv_empresa    vg_datos  -wi-a----- 15.00g
  lv_usuarios   vg_datos  -wi-a----- 10.00g
```

Perfecto. Tenemos nuestros tres volúmenes lógicos creados con los tamaños especificados.

Verificamos el espacio restante en el grupo de volúmenes:

```bash
sudo vgs
```

```
  VG        #PV #LV #SN Attr   VSize  VFree 
  ubuntu-vg   1   1   0 wz--n- 23.00g     0 
  vg_datos    3   3   0 wz--n- 29.99g 4.99g
```

Tenemos aproximadamente 5GB libres en `vg_datos` que podremos usar más adelante para ampliar volúmenes existentes o crear nuevos.

#### Paso 4: Crear sistemas de archivos

Los volúmenes lógicos están creados, pero son como "discos en blanco" sin formato. Necesitan un sistema de archivos para poder almacenar datos.

En Linux, el sistema de archivos más común y recomendado es **ext4**. Es estable, eficiente y ampliamente soportado.

Formateamos cada volumen lógico:

```bash
sudo mkfs.ext4 /dev/vg_datos/lv_empresa
```

Este comando tardará unos segundos y mostrará información sobre el sistema de archivos creado:

```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 3932160 4k blocks and 983040 inodes
Filesystem UUID: abc123-def456-...
Superblock backups stored on blocks: ...
...
Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Repetimos para los otros dos volúmenes:

```bash
sudo mkfs.ext4 /dev/vg_datos/lv_usuarios
sudo mkfs.ext4 /dev/vg_datos/lv_compartido
```

Ahora nuestros volúmenes tienen un sistema de archivos y están listos para usarse.

#### Paso 5: Crear puntos de montaje

En Linux, para acceder a un volumen debemos **montarlo** en una carpeta del sistema. Esa carpeta se llama **punto de montaje**.

Por convención, los datos de servidores suelen montarse en `/srv/` (de "service" - servicio).

Creamos las carpetas donde montaremos nuestros volúmenes:

```bash
sudo mkdir -p /srv/empresa
sudo mkdir -p /srv/usuarios
sudo mkdir -p /srv/compartido
```

El parámetro `-p` (parent) crea todas las carpetas necesarias en la ruta. Si `/srv` no existiera (aunque normalmente sí existe), también la crearía.

Verificamos que se han creado:

```bash
ls -l /srv/
```

Deberíamos ver:

```
total 12
drwxr-xr-x 2 root root 4096 ... compartido
drwxr-xr-x 2 root root 4096 ... empresa
drwxr-xr-x 2 root root 4096 ... usuarios
```

#### Paso 6: Montar los volúmenes

Ahora montamos cada volumen lógico en su carpeta correspondiente:

```bash
sudo mount /dev/vg_datos/lv_empresa /srv/empresa
sudo mount /dev/vg_datos/lv_usuarios /srv/usuarios
sudo mount /dev/vg_datos/lv_compartido /srv/compartido
```

Estos comandos no muestran ninguna salida si todo va bien (en Linux, "ninguna noticia es buena noticia").

Verificamos que los volúmenes están montados correctamente:

```bash
df -h
```

El comando `df` (disk free) muestra el espacio de todos los sistemas de archivos montados. Con la opción `-h` (human-readable) muestra los tamaños en formato legible.

Al final de la lista deberíamos ver:

```
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/vg_datos-lv_empresa     15G   24K   14G   1% /srv/empresa
/dev/mapper/vg_datos-lv_usuarios   9.8G   24K  9.3G   1% /srv/usuarios
/dev/mapper/vg_datos-lv_compartido 4.9G   24K  4.7G   1% /srv/compartido
```

**Interpretación**:

- Los volúmenes están montados en las rutas correctas
- El espacio disponible es ligeramente menor que el tamaño asignado (normal, el sistema de archivos necesita espacio para metadatos)
- Uso actual: casi 0% (solo metadatos del sistema de archivos)

Podemos probar a crear un archivo de prueba:

```bash
sudo touch /srv/empresa/prueba.txt
ls -l /srv/empresa/
```

Deberíamos ver el archivo creado.

#### Paso 7: Montaje automático con /etc/fstab

Hay un problema: si reiniciamos el servidor ahora, los volúmenes NO se montarán automáticamente. Después de cada reinicio tendríamos que montarlos manualmente.

Para que se monten automáticamente al iniciar el sistema, debemos configurar el archivo `/etc/fstab` (file systems table - tabla de sistemas de archivos).

Primero, hacemos una copia de seguridad del archivo (buena práctica antes de modificar archivos críticos):

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Editamos el archivo:

```bash
sudo nano /etc/fstab
```

Veremos líneas similares a estas (puede variar):

```
# /etc/fstab: static file system information.
UUID=abc-123... / ext4 defaults 0 1
UUID=def-456... /boot ext4 defaults 0 2
/swap.img none swap sw 0 0
```

Nos desplazamos al final del archivo y añadimos estas tres líneas:

```
/dev/vg_datos/lv_empresa    /srv/empresa     ext4  defaults  0  2
/dev/vg_datos/lv_usuarios   /srv/usuarios    ext4  defaults  0  2
/dev/vg_datos/lv_compartido /srv/compartido  ext4  defaults  0  2
```

**Explicación de cada campo**:

1. Dispositivo: `/dev/vg_datos/lv_empresa` (ruta al volumen)
2. Punto de montaje: `/srv/empresa` (dónde se monta)
3. Tipo de sistema de archivos: `ext4`
4. Opciones: `defaults` (usa opciones por defecto: lectura/escritura, etc.)
5. Dump: `0` (no hacer backup automático con dump)
6. Pass: `2` (orden de comprobación en el arranque. 0=no comprobar, 1=primero el root, 2=después los demás)

Guardamos el archivo (`Ctrl+O`, `Enter`, `Ctrl+X`).

**Verificar que no hay errores** (PASO CRÍTICO):

Antes de reiniciar, SIEMPRE debemos verificar que el archivo `/etc/fstab` no contiene errores. Un error en este archivo puede hacer que el sistema no arranque.

```bash
sudo mount -a
```

Este comando intenta montar todos los sistemas de archivos listados en `/etc/fstab`. Si hay algún error, nos lo mostrará. Si no muestra nada, significa que todo está correcto.

Verificamos de nuevo que los volúmenes siguen montados:

```bash
df -h | grep srv
```

Deberíamos ver nuestros tres volúmenes.

**Reiniciar para comprobar que funciona**:

```bash
sudo reboot
```

El sistema se reiniciará. Esperamos unos 30 segundos y volvemos a hacer login.

Tras el reinicio, verificamos inmediatamente que los volúmenes se han montado automáticamente:

```bash
df -h | grep srv
```

Si vemos los tres volúmenes montados, ¡perfecto! El montaje automático funciona correctamente.

### Ampliar un volumen lógico

Una de las grandes ventajas de LVM es la capacidad de ampliar volúmenes fácilmente, incluso con el sistema en funcionamiento. Vamos a demostrarlo ampliando `lv_empresa` en 3GB adicionales.

Este proceso se hace sin detener el servidor, sin desmontar el volumen y sin interrumpir el acceso a los datos. Es una funcionalidad extraordinaria que no existe con particiones tradicionales.

**Paso 1: Verificar espacio disponible en el VG**:

Antes de ampliar, confirmamos que tenemos espacio libre en el grupo de volúmenes:

```bash
sudo vgs
```

```
  VG        #PV #LV #SN Attr   VSize  VFree 
  vg_datos    3   3   0 wz--n- 29.99g 4.99g
```

Tenemos aproximadamente 5GB libres, suficiente para añadir 3GB a un volumen.

**Paso 2: Ampliar el volumen lógico**:

```bash
sudo lvextend -L +3G /dev/vg_datos/lv_empresa
```

**Explicación**:
- `lvextend`: comando para extender (ampliar) un volumen lógico
- `-L +3G`: añadir 3 gigabytes al tamaño actual (el `+` indica que es adicional, no el tamaño total)
- `/dev/vg_datos/lv_empresa`: ruta al volumen que queremos ampliar

Confirmación:

```
  Size of logical volume vg_datos/lv_empresa changed from 15.00 GiB to 18.00 GiB.
  Logical volume vg_datos/lv_empresa successfully resized.
```

**Paso 3: Ampliar el sistema de archivos**:

Hemos ampliado el volumen lógico, pero el sistema de archivos ext4 que hay dentro todavía "piensa" que tiene 15GB. Debemos indicarle que ocupe todo el nuevo espacio:

```bash
sudo resize2fs /dev/vg_datos/lv_empresa
```

Salida:

```
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/vg_datos/lv_empresa is mounted on /srv/empresa; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/vg_datos/lv_empresa is now 4718592 (4k) blocks long.
```

La línea clave es `on-line resizing required` que confirma que se está redimensionando con el sistema en funcionamiento.

**Paso 4: Verificar el resultado**:

```bash
df -h | grep empresa
```

```
/dev/mapper/vg_datos-lv_empresa  18G  24K  17G   1% /srv/empresa
```

¡Perfecto! El volumen ahora tiene 18GB en lugar de 15GB.

**Reflexión sobre lo que hemos hecho**:

Acabamos de ampliar un volumen de 15GB a 18GB:
- Sin detener el servidor
- Sin desmontar el volumen
- Sin interrumpir el acceso a los datos
- En menos de un minuto
- Sin riesgo de pérdida de datos

Con particiones tradicionales, esto habría requerido:
1. Hacer backup completo de los datos
2. Apagar el servidor
3. Particionar de nuevo el disco
4. Formatear
5. Restaurar los datos
6. Reiniciar y cruzar los dedos

Esta es la potencia de LVM.

### Comandos útiles de LVM para el día a día

**Ver información resumida**:
```bash
sudo pvs    # Resumen de volúmenes físicos
sudo vgs    # Resumen de grupos de volúmenes
sudo lvs    # Resumen de volúmenes lógicos
```

**Ver información detallada**:
```bash
sudo pvdisplay                    # Detalle de todos los PVs
sudo vgdisplay vg_datos          # Detalle de un VG específico
sudo lvdisplay /dev/vg_datos/lv_empresa  # Detalle de un LV específico
```

**Ampliar un volumen** (ya lo vimos):
```bash
sudo lvextend -L +2G /dev/vg_datos/lv_usuarios
sudo resize2fs /dev/vg_datos/lv_usuarios
```

**Añadir un disco nuevo a un VG existente**:
```bash
sudo pvcreate /dev/sde          # Preparar el disco nuevo
sudo vgextend vg_datos /dev/sde # Añadirlo al grupo
```

**Ver estado del montaje**:
```bash
df -h               # Ver todos los sistemas de archivos montados
mount | grep srv    # Ver solo nuestros volúmenes
```

---

## 4. Instalación y configuración básica de Samba

### Qué es Samba y para qué sirve

**Samba** es una suite de software que permite que sistemas Linux/Unix compartan carpetas, archivos e impresoras con sistemas Windows de forma nativa y transparente.

Samba implementa el protocolo **SMB/CIFS** (Server Message Block / Common Internet File System), que es exactamente el mismo protocolo que Windows usa para compartir recursos en red. Esto significa que desde un cliente Windows, acceder a una carpeta compartida de Linux con Samba es idéntico a acceder a una carpeta compartida de otro Windows. El usuario no nota ninguna diferencia.

**¿Por qué necesitamos Samba?**

Windows y Linux hablan "idiomas" diferentes para compartir archivos. Windows usa SMB/CIFS. Linux históricamente usaba NFS (Network File System). Sin Samba, los clientes Windows no podrían acceder a carpetas compartidas en servidores Linux.

Samba actúa como un "traductor" que hace que el servidor Linux hable SMB perfectamente, permitiendo la integración con redes Windows.

### Instalar Samba

La instalación de Samba en Ubuntu Server es muy sencilla gracias al gestor de paquetes `apt`.

Primero actualizamos la lista de paquetes disponibles:

```bash
sudo apt update
```

Instalamos Samba y sus herramientas:

```bash
sudo apt install -y samba samba-common-bin
```

**Explicación de los paquetes**:
- `samba`: el servidor Samba propiamente dicho
- `samba-common-bin`: herramientas de línea de comandos para gestionar Samba
- El parámetro `-y` responde automáticamente "yes" a la confirmación

La instalación tardará un par de minutos. Descargará los paquetes, los instalará y configurará los servicios automáticamente.

Una vez instalado, verificamos que el servicio de Samba está activo y funcionando:

```bash
sudo systemctl status smbd
```

Deberíamos ver algo como:

```
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
     ...
```

**Puntos clave a verificar**:
- `Loaded: ...enabled...`: el servicio está habilitado para iniciarse automáticamente al arrancar
- `Active: active (running)`: el servicio está ejecutándose ahora mismo

Si por alguna razón el servicio no estuviera activo, lo iniciamos y habilitamos:

```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

Verificamos también el estado del servicio de nombres NetBIOS (necesario para la resolución de nombres en redes Windows):

```bash
sudo systemctl status nmbd
```

Debe mostrar también `active (running)`.

### Configuración del firewall

Ubuntu Server por defecto no tiene firewall activado, pero si lo tuviéramos, necesitaríamos abrir los puertos de Samba:

```bash
sudo ufw allow samba
```

En nuestro caso, como estamos en un entorno de laboratorio y probablemente no tenemos el firewall activado, este paso es opcional. Podemos verificar el estado del firewall con:

```bash
sudo ufw status
```

Si muestra `Status: inactive`, no necesitamos hacer nada más.

### Crear una carpeta compartida simple

Antes de complicarnos con Active Directory y configuraciones avanzadas, vamos a crear una carpeta compartida simple para entender cómo funciona Samba y verificar que todo está correctamente instalado.

#### Crear la carpeta y asignar permisos

Vamos a crear una carpeta de prueba en uno de nuestros volúmenes LVM:

```bash
sudo mkdir -p /srv/compartido/prueba
```

De momento, vamos a permitir que cualquiera pueda escribir en esta carpeta (solo para pruebas, en producción jamás haríamos esto):

```bash
sudo chmod 777 /srv/compartido/prueba
```

**Explicación del permiso 777**:
- Primer 7: permisos del propietario (lectura + escritura + ejecución)
- Segundo 7: permisos del grupo (lectura + escritura + ejecución)
- Tercer 7: permisos de otros (lectura + escritura + ejecución)

Esto es **muy inseguro** y solo lo hacemos para las pruebas iniciales. Más adelante configuraremos permisos adecuados.

#### Configurar Samba

El archivo de configuración principal de Samba es `/etc/samba/smb.conf`. Este archivo controla todos los aspectos del servidor: recursos compartidos, permisos, autenticación, etc.

Antes de modificarlo, hacemos una copia de seguridad del archivo original:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.original
```

Ahora editamos el archivo:

```bash
sudo nano /etc/samba/smb.conf
```

El archivo es bastante largo y contiene muchas opciones comentadas (líneas que empiezan con `;` o `#`). No necesitamos entender todo ahora.

Nos desplazamos al final del archivo (podemos usar `Ctrl+V` varias veces para avanzar páginas rápidamente en nano).

Al final del archivo, añadimos la configuración de nuestro recurso compartido:

```ini
[Prueba]
   comment = Carpeta de prueba
   path = /srv/compartido/prueba
   browseable = yes
   read only = no
   guest ok = yes
   create mask = 0777
   directory mask = 0777
```

**Explicación de cada parámetro**:

- `[Prueba]`: nombre del recurso compartido (es lo que verán los clientes Windows al conectarse)
- `comment`: descripción del recurso (aparece en algunas herramientas de Windows)
- `path`: ruta real en el sistema Linux donde están los archivos
- `browseable = yes`: el recurso aparecerá cuando se navegue por la red (lo veremos sin tener que escribir su nombre)
- `read only = no`: permite escritura (si fuera `yes`, solo lectura)
- `guest ok = yes`: permite acceso sin autenticación (cualquiera puede entrar sin usuario/contraseña)
- `create mask = 0777`: permisos que tendrán los archivos nuevos que se creen
- `directory mask = 0777`: permisos que tendrán las carpetas nuevas que se creen

**IMPORTANTE**: La configuración con `guest ok = yes` es insegura y solo para pruebas. Más adelante la cambiaremos.

Guardamos el archivo (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### Verificar la configuración

Antes de reiniciar Samba, verificamos que no hay errores de sintaxis en el archivo de configuración:

```bash
testparm
```

Este comando es muy útil. Lee el archivo `smb.conf`, detecta errores de sintaxis y muestra la configuración real que se aplicará (ignora líneas comentadas y valores por defecto).

Si hay errores, los mostrará en rojo. Si todo está bien, veremos algo como:

```
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
   ...

[Prueba]
   comment = Carpeta de prueba
   create mask = 0777
   directory mask = 0777
   guest ok = Yes
   path = /srv/compartido/prueba
   read only = No
```

La línea `Loaded services file OK.` confirma que no hay errores de sintaxis.

Pulsamos `Enter` para ver la configuración completa o `Ctrl+C` para salir.

#### Reiniciar Samba

Para que los cambios tengan efecto, reiniciamos el servicio de Samba:

```bash
sudo systemctl restart smbd
```

Verificamos que sigue activo después del reinicio:

```bash
sudo systemctl status smbd
```

Debe mostrar `active (running)`.

#### Probar desde Windows

Ahora viene el momento de la verdad: probar que podemos acceder desde un cliente Windows.

Desde un cliente Windows conectado a la misma red que el servidor Linux (en nuestro caso, la red de departamentos `192.168.10.0/24`):

1. Abrimos el **Explorador de archivos**
2. En la barra de direcciones (donde aparece la ruta) escribimos: `\\192.168.10.2` (la IP de nuestro servidor Linux)
3. Pulsamos `Enter`

Deberíamos ver una ventana con el recurso compartido **Prueba**.

4. Hacemos doble clic en **Prueba** para entrar
5. Intentamos crear un archivo o carpeta dentro

Si podemos crear archivos y carpetas, ¡enhorabuena! Samba está funcionando correctamente.

**Solución de problemas comunes**:

**No aparece el recurso compartido**:
- Verificar que el firewall del Windows Server no está bloqueando la conexión
- Verificar que la IP del servidor Linux es correcta y hay conectividad: `ping 192.168.10.2` desde Windows
- Verificar que el servicio smbd está activo: `sudo systemctl status smbd`

**Aparece pero no puedo acceder**:
- Verificar permisos de la carpeta: `ls -ld /srv/compartido/prueba`
- Verificar configuración de Samba: `testparm -s | grep -A 10 "\[Prueba\]"`

**Puedo acceder pero no crear archivos**:
- Verificar que `read only = no` en la configuración
- Verificar permisos de la carpeta: debe tener al menos `chmod 775` o `777`

### Usuarios de Samba

Aunque hemos configurado acceso anónimo (`guest ok = yes`), en producción siempre necesitaremos autenticación con usuarios. Samba mantiene su propia base de datos de usuarios y contraseñas, separada de los usuarios del sistema Linux.

Para crear un usuario de Samba, primero debe existir como usuario del sistema Linux. Luego le asignamos una contraseña específica para Samba.

**Crear un usuario de prueba**:

```bash
sudo useradd -M -s /usr/sbin/nologin samba_test
```

**Explicación**:
- `useradd`: comando para crear usuarios
- `-M`: no crear carpeta personal (no la necesita)
- `-s /usr/sbin/nologin`: no permitir login interactivo (este usuario solo existe para Samba)
- `samba_test`: nombre del usuario

Ahora le asignamos una contraseña para Samba:

```bash
sudo smbpasswd -a samba_test
```

Nos pedirá la contraseña dos veces. Elegimos una sencilla de recordar.

Confirmación:

```
Added user samba_test.
```

Este usuario ahora puede autenticarse en recursos compartidos que requieran usuario/contraseña.

Para listar los usuarios de Samba:

```bash
sudo pdbedit -L
```

Para eliminar un usuario de Samba (si lo necesitamos más adelante):

```bash
sudo smbpasswd -x nombre_usuario
```

---

## 5. Integración con Active Directory

Ahora llega la parte más interesante y compleja de este tema: vamos a unir nuestro servidor Linux al dominio Active Directory de Windows Server que creamos en temas anteriores.

Una vez completada esta integración, conseguiremos que:

- Los usuarios del dominio Windows puedan autenticarse en el servidor Linux con sus credenciales del AD
- Los recursos compartidos en Linux respeten los permisos y grupos del Active Directory
- Todo funcione de forma transparente: los usuarios no notarán diferencia entre recursos Windows y Linux

Esta configuración es habitual en empresas con infraestructuras híbridas y representa una habilidad muy valorada profesionalmente.

### Requisitos previos

Antes de comenzar el proceso de integración, debemos asegurarnos de que se cumplen varios requisitos técnicos fundamentales.

#### 1. Conectividad con el Windows Server

Primero verificamos que existe comunicación de red con el controlador de dominio Windows:

```bash
ping -c 4 192.168.10.1
```

Deberíamos recibir respuestas confirmando la conectividad.

#### 2. Resolución DNS del dominio

Este es uno de los puntos más críticos. El servidor Linux debe poder resolver el nombre del dominio Active Directory. Para ello, **debe usar el Windows Server como servidor DNS**.

Verificamos que nuestra configuración DNS es correcta:

```bash
cat /etc/netplan/00-installer-config.yaml | grep -A 3 nameservers
```

Deberíamos ver que incluye la IP del Windows Server (`192.168.10.1`) como nameserver.

Probamos la resolución del dominio:

```bash
nslookup DOMXXX.local
```

(Sustituimos DOMXXX por el nombre real de vuestro dominio)

Si funciona, debería mostrar la IP del Windows Server. Si da error "server can't find...", hay un problema de DNS que debemos resolver antes de continuar.

Si no teníamos `nslookup` instalado:

```bash
sudo apt install -y dnsutils
```

También podemos probar con:

```bash
host DOMXXX.local
```

o

```bash
dig DOMXXX.local
```

#### 3. Sincronización de hora

La autenticación Kerberos (que usa Active Directory) es extremadamente sensible a diferencias de hora. Si el reloj del servidor Linux y del Windows Server difieren en más de 5 minutos, la autenticación fallará.

Instalamos el cliente NTP para sincronización de hora:

```bash
sudo apt install -y chrony
```

Verificamos que el servicio está activo:

```bash
sudo systemctl status chronyd
```

Verificamos la sincronización:

```bash
timedatectl
```

Deberíamos ver:

```
      Local time: ...
  Universal time: ...
        RTC time: ...
       Time zone: ...
     NTP enabled: yes
NTP synchronized: yes
```

La línea clave es `NTP synchronized: yes`. Si muestra `no`, esperamos un minuto y volvemos a verificar.

Opcionalmente, podemos configurar el servidor NTP para que use el Windows Server como fuente de hora, pero con chrony configurado por defecto suele ser suficiente.

#### 4. Instalar paquetes necesarios

Para unir Linux a Active Directory necesitamos varios paquetes que gestionan la autenticación, la comunicación con el AD y la integración con Samba:

```bash
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

**Explicación de los paquetes principales**:

- `realmd`: herramienta de alto nivel para descubrir y unirse a dominios
- `sssd`: System Security Services Daemon, gestiona la autenticación contra el AD
- `adcli`: herramientas de línea de comandos para Active Directory
- `samba-common-bin`: herramientas de Samba necesarias
- `oddjob-mkhomedir`: crea automáticamente carpetas personales cuando los usuarios del AD inician sesión por primera vez

La instalación tardará varios minutos.

### Unir el servidor al dominio

Con todos los requisitos cumplidos, procedemos a unir el servidor Linux al dominio Active Directory.

#### Descubrir el dominio

Primero verificamos que podemos "descubrir" el dominio (es decir, que el servidor puede ver el Active Directory y obtener información sobre él):

```bash
sudo realm discover DOMXXX.local
```

(Sustituimos `DOMXXX` por el nombre de vuestro dominio)

Si todo funciona correctamente, veremos información detallada del dominio:

```
DOMXXX.local
  type: kerberos
  realm-name: DOMXXX.LOCAL
  domain-name: domxxx.local
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

**Puntos clave a verificar**:

- `type: kerberos`: confirma que es un dominio Kerberos (Active Directory)
- `configured: no`: todavía no estamos unidos al dominio
- `server-software: active-directory`: confirma que es un AD de Windows
- `client-software: sssd`: usaremos SSSD para la autenticación

Si este comando falla, el problema suele ser DNS. Volvemos a verificar la resolución del dominio.

#### Unir al dominio

Ahora ejecutamos el comando para unir el servidor al dominio. Necesitaremos credenciales de un usuario administrador del dominio Windows:

```bash
sudo realm join --user=Administrador DOMXXX.local
```

**Explicación**:
- `realm join`: comando para unirse al dominio
- `--user=Administrador`: usamos la cuenta de Administrador del dominio (o cualquier usuario con permisos de administrador del dominio)
- `DOMXXX.local`: nombre del dominio

El comando nos pedirá la contraseña del Administrador del dominio Windows. La escribimos (no se verá al escribir) y pulsamos `Enter`.

Si todo va bien, el comando completará sin mostrar ningún mensaje de error. El proceso puede tardar 10-30 segundos.

**Posibles errores comunes**:

**"Failed to join domain: failed to lookup DC info"**:
- Problema de DNS. Verificar que el servidor puede resolver el nombre del dominio

**"Couldn't authenticate to active directory: SASL(-1)"**:
- Problema de Kerberos. Suele ser por diferencia de hora. Verificar `timedatectl`

**"Insufficient permissions"**:
- El usuario proporcionado no tiene permisos. Usar Administrador del dominio o un usuario con derechos adecuados

#### Verificar que estamos unidos

Verificamos que la unión fue exitosa:

```bash
sudo realm list
```

Deberíamos ver información del dominio con `configured: kerberos-member`:

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

La línea `configured: kerberos-member` confirma que estamos correctamente unidos al dominio como miembro.

#### Verificar usuarios del dominio

Podemos listar usuarios del dominio Windows desde Linux:

```bash
sudo getent passwd | grep DOMXXX
```

Deberíamos ver los usuarios del dominio en formato `usuario@domxxx.local` o solo `usuario` dependiendo de la configuración.

Por ejemplo:

```
falonso@domxxx.local:*:1721601103:1721600513::/home/falonso@domxxx.local:/bin/bash
jbroto@domxxx.local:*:1721601104:1721600513::/home/jbroto@domxxx.local:/bin/bash
...
```

También podemos listar grupos del dominio:

```bash
sudo getent group | grep DOMXXX
```

### Configurar inicio de sesión

Por defecto, para iniciar sesión con un usuario del dominio debemos usar el formato completo: `usuario@domxxx.local`. Esto es incómodo. Vamos a configurar el sistema para poder usar solo el nombre de usuario.

Editamos el archivo de configuración de SSSD:

```bash
sudo nano /etc/sssd/sssd.conf
```

Buscamos la sección `[domain/DOMXXX.local]` y añadimos o modificamos esta línea:

```ini
[domain/DOMXXX.local]
use_fully_qualified_names = False
```

Si no encontramos esa línea, la añadimos debajo del encabezado `[domain/DOMXXX.local]`.

Guardamos el archivo (`Ctrl+O`, `Enter`, `Ctrl+X`).

Reiniciamos SSSD para aplicar los cambios:

```bash
sudo systemctl restart sssd
```

Ahora verificamos que funciona:

```bash
id falonso
```

Deberíamos ver información del usuario (UID, GID, grupos) sin necesidad de escribir `@domxxx.local`.

### Crear carpeta personal automática

Cuando un usuario del dominio inicia sesión por primera vez en el servidor Linux, necesita una carpeta personal. Configuramos el sistema para crearla automáticamente:

```bash
sudo pam-auth-update
```

Aparecerá una interfaz en modo texto. Con las flechas navegamos a la opción:

```
[*] Create home directory on login
```

Nos aseguramos de que está marcada con un asterisco `*` (si no lo está, la marcamos con la barra espaciadora).

Navegamos hasta `<Ok>` y pulsamos `Enter`.

### Configurar Samba como miembro del dominio

Ahora que el servidor está unido al dominio, necesitamos configurar Samba para que también use la autenticación del dominio.

Editamos el archivo de configuración de Samba:

```bash
sudo nano /etc/samba/smb.conf
```

Buscamos la sección `[global]` al principio del archivo. Modificamos o añadimos estas líneas:

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

**IMPORTANTE**: Sustituimos todas las ocurrencias de `DOMXXX` por el nombre real de vuestro dominio (en mayúsculas donde corresponda).

**Explicación de la configuración**:

- `workgroup = DOMXXX`: nombre NetBIOS del dominio (la parte antes del `.local`)
- `security = ADS`: usar Active Directory Security
- `realm = DOMXXX.LOCAL`: nombre completo del dominio en mayúsculas
- Las líneas `idmap` configuran cómo se mapean los IDs de Windows (SIDs) a IDs de Linux (UIDs/GIDs)
- `template shell` y `template homedir`: configuran el shell y carpeta personal para usuarios del dominio
- Las líneas `winbind` permiten enumerar usuarios y grupos del dominio

Guardamos el archivo.

Instalamos Winbind (si no lo teníamos ya):

```bash
sudo apt install -y winbind
```

Reiniciamos todos los servicios relacionados:

```bash
sudo systemctl restart smbd nmbd winbind
```

Verificamos que Winbind funciona correctamente:

```bash
sudo wbinfo -u
```

Deberíamos ver la lista de usuarios del dominio.

```bash
sudo wbinfo -g
```

Deberíamos ver la lista de grupos del dominio.

Si estos comandos funcionan, significa que Samba puede comunicarse correctamente con el Active Directory.

### Crear recurso compartido con autenticación del dominio

Ahora vamos a crear un recurso compartido que use autenticación del dominio y respete los grupos del AD.

Primero, eliminamos o comentamos la sección `[Prueba]` que creamos anteriormente (ya no la necesitamos):

```bash
sudo nano /etc/samba/smb.conf
```

Buscamos `[Prueba]` y comentamos todas sus líneas añadiendo `#` al principio, o directamente las borramos.

Al final del archivo, añadimos una nueva configuración:

```ini
[Compartido]
   comment = Carpeta compartida para usuarios del dominio
   path = /srv/compartido
   browseable = yes
   read only = no
   valid users = @"DOMXXX\Domain Users"
   write list = @"DOMXXX\Domain Users"
   force group = "DOMXXX\Domain Users"
   create mask = 0770
   directory mask = 0770
```

**Explicación de la configuración**:

- `valid users = @"DOMXXX\Domain Users"`: solo usuarios del grupo "Domain Users" del dominio pueden acceder
- El `@` indica que es un grupo (no un usuario individual)
- Las comillas son necesarias porque el nombre contiene una contrabarra
- `write list`: quién puede escribir (en este caso, el mismo grupo)
- `force group`: los archivos creados pertenecerán a este grupo
- `create mask` y `directory mask`: permisos de archivos y carpetas creados (770 = rwx para propietario y grupo, nada para otros)

Guardamos el archivo.

Ahora ajustamos los permisos de la carpeta en el sistema Linux:

```bash
sudo chown -R root:"DOMXXX\\domain users" /srv/compartido
sudo chmod -R 770 /srv/compartido
```

**Nota**: La doble contrabarra `\\` es necesaria en la línea de comandos para escapar el carácter.

Verificamos la configuración de Samba:

```bash
testparm
```

Si no hay errores, reiniciamos Samba:

```bash
sudo systemctl restart smbd
```

### Probar acceso desde Windows

Ahora viene la prueba final: verificar que un usuario del dominio puede acceder al recurso compartido desde Windows usando sus credenciales del AD.

Desde un cliente Windows **unido al dominio**:

1. Iniciamos sesión con un usuario del dominio (por ejemplo, `falonso` que creamos en el tema de Active Directory)
2. Abrimos el **Explorador de archivos**
3. En la barra de direcciones escribimos: `\\192.168.10.2` o `\\SRVXXX_Linux`
4. Deberíamos ver el recurso compartido **Compartido**
5. Hacemos doble clic para entrar

Si todo funciona correctamente:

- No debería pedir credenciales (usa automáticamente las del usuario actual del dominio)
- Podemos crear archivos y carpetas dentro
- Los permisos se respetan según la configuración

Si pide credenciales, introducimos: `DOMXXX\falonso` y su contraseña del dominio.

Para verificar que los permisos funcionan correctamente, podemos:

1. Crear un archivo desde Windows
2. Desde el servidor Linux, verificar el propietario:

```bash
ls -l /srv/compartido/
```

Deberíamos ver que el archivo pertenece al usuario del dominio que lo creó.

---

## Resumen del Tema 7

En este tema hemos recorrido un camino largo y complejo, pero hemos conseguido algo extraordinario: integrar completamente un servidor Linux con una infraestructura de Windows Server y Active Directory.

**Lo que hemos aprendido**:

1. **Instalación y configuración básica de Ubuntu Server**: Hemos instalado Ubuntu Server 22.04 LTS desde cero, configurado la red de forma estática usando Netplan y habilitado el acceso remoto por SSH.

2. **LVM (Logical Volume Manager)**: Hemos aprendido a gestionar el almacenamiento de forma flexible. Sabemos crear volúmenes físicos (PV), agruparlos en grupos de volúmenes (VG), crear volúmenes lógicos (LV) y ampliarlos sin detener el sistema. Esta habilidad es fundamental en administración de servidores Linux.

3. **Samba**: Hemos instalado y configurado Samba para compartir recursos con clientes Windows. Empezamos con configuraciones simples y avanzamos hasta integración completa con Active Directory.

4. **Integración con Active Directory**: Este es el logro más importante. Hemos unido un servidor Linux a un dominio Windows, permitiendo que los usuarios del AD se autentiquen en Linux y accedan a recursos compartidos con sus credenciales del dominio. Los permisos y grupos del AD se respetan en ambos sistemas.

**Habilidades profesionales adquiridas**:

- Administración básica de servidores Linux por terminal
- Gestión avanzada de almacenamiento con LVM
- Implementación de recursos compartidos multiplataforma
- Integración de redes heterogéneas (Linux + Windows)
- Autenticación centralizada mediante Active Directory
- Resolución de problemas en entornos híbridos

**Aplicaciones en el mundo real**:

Todo lo que hemos aprendido es directamente aplicable en empresas reales. Las redes heterogéneas son la norma, no la excepción. Saber integrar Linux y Windows es una habilidad muy demandada y bien valorada profesionalmente.

---

## Práctica evaluable del Tema 7

Al final del tema tendréis que entregar una práctica donde demostréis que habéis completado todos los pasos y que el sistema funciona correctamente.

**La práctica incluirá**:

1. Capturas de pantalla de la instalación y configuración de red
2. Verificación de la estructura LVM creada
3. Demostración de Samba funcionando con autenticación del dominio
4. Pruebas de acceso desde clientes Windows
5. Documentación del proceso y problemas encontrados

**Plazo de entrega**: Se comunicará en clase.

**Formato**: Documento PDF con capturas comentadas y explicaciones.
