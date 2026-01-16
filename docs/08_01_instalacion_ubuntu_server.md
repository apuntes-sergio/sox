---
title: Instalación de Ubuntu Server
description:  Gestión del dominio en Windows Server. Integración de Ubuntu 24.04 LTS en Dominio Windows Server
---

## Instalación de Ubuntu Server

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

