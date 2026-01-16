---
title: Ubuntu Desktop. Organización de discos y carpetas en Linux
description: Gestión de sistemas Linux. Integración de Ubuntu Desktop y Server en entornos Windows
---

## Organización de discos y carpetas en Linux

Una de las diferencias más evidentes entre Windows y Linux es cómo organizan el sistema de archivos. Entender esta estructura es fundamental para moveros con soltura en Ubuntu.

### El sistema de archivos: todo parte de `/`

En Windows estamos acostumbrados a tener varias unidades: `C:` para el sistema, `D:` para datos, etc. Cada disco tiene su propia letra y son independientes entre sí.

En Linux **no existen las letras de unidad**. Todo el sistema de archivos es un único árbol jerárquico que empieza en `/` (llamado "raíz" o "root").

Todos los archivos, carpetas, discos duros, unidades USB, conexiones de red, etc., cuelgan deesta raíz única. Los discos adicionales se "montan" como carpetas dentro de este árbol.

Por ejemplo, si tenemos un disco duro adicional, podríamos montarlo en `/datos` y accederíamos a su contenido a través de esa ruta, pero seguiría siendo parte del mismo árbol de directorios.

### Estructura de directorios estándar

Linux sigue una estructura de directorios estandarizada que es prácticamente idéntica en todas las distribuciones:

**`/` (raíz)**: La cima de la jerarquía. Contiene todos los demás directorios.

**`/home`**: Carpetas personales de los usuarios. Es el equivalente a `C:\Users` en Windows.

- `/home/sergio`: carpeta personal del usuario sergio
- `/home/maria`: carpeta personal del usuario maria

Cada usuario tiene permisos completos sobre su carpeta personal, pero no puede acceder a las carpetas de otros usuarios.

**`/root`**: Carpeta personal del superusuario root. No está dentro de `/home` por razones históricas y de seguridad.

**`/etc`**: Archivos de configuración del sistema. Equivalente al Registro de Windows pero en formato texto legible.

- `/etc/netplan`: configuración de red
- `/etc/samba`: configuración de Samba
- `/etc/passwd`: lista de usuarios del sistema

**`/var`**: Datos variables que cambian durante la operación del sistema.

- `/var/log`: archivos de registro (logs)
- `/var/www`: contenido de servidores web
- `/var/spool`: colas de impresión, correo, etc.

**`/tmp`**: Archivos temporales. Se borra automáticamente al reiniciar. Cualquier usuario puede escribir aquí.

**`/usr`**: Aplicaciones y datos de usuario (pero no confundir con carpetas personales).

- `/usr/bin`: programas ejecutables de usuario
- `/usr/lib`: librerías compartidas
- `/usr/share`: datos compartidos (iconos, documentación, etc.)

**`/opt`**: Software opcional instalado manualmente (no desde repositorios). Es el equivalente a `C:\Program Files` en Windows.

**`/mnt` y `/media`**: Puntos de montaje temporales. En `/media` se montan automáticamente las unidades USB, CDs, etc.

**`/bin` y `/sbin`**: Programas esenciales del sistema.

- `/bin`: comandos básicos disponibles para todos los usuarios (ls, cp, etc.)
- `/sbin`: comandos de administración del sistema (solo root)

**`/boot`**: Archivos necesarios para arrancar el sistema (kernel, initramfs, etc.).

**`/dev`**: Archivos de dispositivos. En Linux, los dispositivos hardware se representan como archivos especiales.

- `/dev/sda`: primer disco duro
- `/dev/sdb1`: primera partición del segundo disco
- `/dev/null`: dispositivo especial que descarta todo lo que se le envía

### Rutas absolutas vs relativas

**Ruta absoluta**: comienza desde la raíz `/`. Ejemplo: `/home/sergio/Documentos/proyecto.txt`

Siempre apunta al mismo lugar independientemente de dónde estemos en el sistema.

**Ruta relativa**: no comienza con `/`. Es relativa al directorio actual.

- Si estamos en `/home/sergio` y escribimos `Documentos/proyecto.txt`, se refiere a `/home/sergio/Documentos/proyecto.txt`
- `.` representa el directorio actual
- `..` representa el directorio padre (un nivel arriba)

Ejemplo práctico:

```bash
cd /home/sergio/Documentos/proyecto
pwd                    # Muestra: /home/sergio/Documentos/proyecto
cd ..                  # Subimos un nivel
pwd                    # Muestra: /home/sergio/Documentos
cd ../Imágenes         # Subimos un nivel y entramos en Imágenes
pwd                    # Muestra: /home/sergio/Imágenes
```

### Carpeta personal y variables de entorno

La carpeta personal del usuario actual se puede referenciar con `~` (tilde).

```bash
cd ~           # Va a /home/sergio (si sergio es el usuario actual)
cd ~/Descargas # Va a /home/sergio/Descargas
```

También existe la variable de entorno `$HOME` que contiene la ruta de nuestra carpeta personal:

```bash
echo $HOME     # Muestra: /home/sergio
cd $HOME       # Mismo efecto que 'cd ~'
```

### Permisos del sistema de archivos

A diferencia de Windows, donde los permisos se gestionan mediante ACLs complejas, Linux usa un sistema más simple de permisos UNIX tradicionales.

Cada archivo o carpeta tiene:

- **Propietario** (un usuario)
- **Grupo** (un grupo de usuarios)
- **Permisos** para propietario, grupo y otros

Los permisos se dividen en tres tipos:

- **r (read)**: permiso de lectura
- **w (write)**: permiso de escritura
- **x (execute)**: permiso de ejecución (para archivos) o acceso (para carpetas)

Cuando hacemos `ls -l`, vemos:

```
-rw-r--r-- 1 sergio sergio 2048 ene 15 10:30 documento.txt
drwxr-xr-x 2 sergio sergio 4096 ene 15 10:35 carpeta
```

La primera columna indica los permisos:

- Primer carácter: tipo (`-` = archivo, `d` = directorio, `l` = enlace simbólico)
- Siguientes 3: permisos del propietario (rw- = lectura y escritura, sin ejecución)
- Siguientes 3: permisos del grupo (r-- = solo lectura)
- Últimos 3: permisos de otros (r-- = solo lectura)

Para cambiar permisos usamos `chmod` (lo veremos más adelante si es necesario).

### Archivos ocultos

En Linux, cualquier archivo o carpeta cuyo nombre comienza con un punto `.` se considera oculto.

Por ejemplo:
- `.bashrc`: archivo de configuración de bash
- `.config`: carpeta de configuraciones de aplicaciones

Para ver archivos ocultos en terminal:

```bash
ls -la
```

En el explorador gráfico (Nautilus), pulsamos `Ctrl+H`.

Los archivos de configuración de usuario normalmente están ocultos en la carpeta personal para no "ensuciar" visualmente el espacio de trabajo del usuario.

---

## Diferencias clave con Windows

Ahora que tenemos Ubuntu Desktop instalado y hemos aprendido lo básico, es importante destacar las diferencias fundamentales que encontraréis al trabajar con Linux en comparación con Windows. Entender estas diferencias os ayudará a adaptaros más rápidamente.

### Filosofía de diseño

**Windows**: Diseñado para que todo sea configurable desde interfaces gráficas. Oculta la complejidad al usuario mediante asistentes y ventanas de propiedades. El usuario promedio nunca necesita abrir una terminal.

**Linux**: Diseñado para que todo sea configurable mediante archivos de texto. La interfaz gráfica es una capa sobre un sistema fundamentalmente basado en terminal. Los usuarios avanzados suelen preferir la terminal porque es más rápida, potente y automatizable.

**Implicación práctica**: En Windows raramente editamos archivos de configuración manualmente. En Linux, editar archivos de texto en `/etc/` es rutinario y normal para configuraciones avanzadas.

### Instalación de software

**Windows**:
- Descargamos archivos `.exe` o `.msi` de Internet
- Cada programa se instala independientemente
- Las actualizaciones las gestiona cada aplicación individualmente
- No hay verificación centralizada de seguridad

**Linux**:
- El software se instala desde repositorios oficiales mediante gestores de paquetes
- Un solo comando instala el programa y todas sus dependencias
- Un solo comando actualiza el sistema operativo y todas las aplicaciones
- Todo el software está firmado digitalmente y verificado

**Implicación práctica**: En Linux es más seguro y cómodo instalar software, pero requiere aprender comandos como `apt install`. A cambio, obtenemos un sistema donde "sudo apt update && sudo apt upgrade" mantiene absolutamente todo actualizado.

### Sistema de archivos

**Windows**:
- Letras de unidad separadas (C:, D:, E:)
- Barra invertida `\` como separador de rutas
- No distingue entre mayúsculas y minúsculas en nombres de archivo
- Muchas extensiones de archivo con significado (.exe, .dll, .bat, etc.)

**Linux**:
- Un único árbol de directorios desde `/`
- Barra normal `/` como separador de rutas
- **Distingue mayúsculas y minúsculas**: `archivo.txt` y `Archivo.txt` son archivos diferentes
- Las extensiones son solo convenciones, no determinan el tipo de archivo
- Los permisos de ejecución determinan si un archivo es ejecutable, no su extensión

**Implicación práctica**: En Linux debéis tener mucho cuidado con mayúsculas/minúsculas al escribir nombres de archivo. `cd Documentos` funciona, pero `cd documentos` dará error si la carpeta se llama "Documentos" con D mayúscula.

### Usuarios y permisos

**Windows**:
- Usuarios estándar y administradores
- Control de Cuentas de Usuario (UAC) para elevar privilegios
- Permisos basados en ACLs complejas

**Linux**:
- Usuario normal, superusuario (root) y usuarios del sistema
- `sudo` para ejecutar comandos específicos con privilegios
- Permisos más simples pero muy efectivos (propietario, grupo, otros)

**Implicación práctica**: En Linux NUNCA trabajamos como root. Siempre trabajamos como usuario normal y usamos `sudo` solo cuando es necesario. Esto hace el sistema mucho más seguro.

### Terminal y línea de comandos

**Windows**:
- La terminal (cmd, PowerShell) es opcional y poco usada por usuarios normales
- Muchas tareas administrativas requieren interfaz gráfica
- Nombres de comandos verbosos (`Get-ChildItem`, `Remove-Item`)

**Linux**:
- La terminal es fundamental y extremadamente potente
- Prácticamente todo se puede hacer desde terminal
- Comandos cortos y eficientes (`ls`, `rm`, `cp`)
- Scripting integrado en el sistema (bash)

**Implicación práctica**: Aunque Ubuntu Desktop tiene buenas herramientas gráficas, merece la pena aprender comandos básicos de terminal porque son más rápidos y potentes. Además, toda la documentación en Internet asume que sabéis usar la terminal.

### Configuración del sistema

**Windows**:
- Configuración almacenada en el Registro (base de datos binaria)
- Difícil de editar manualmente o hacer backups selectivos

**Linux**:
- Toda la configuración en archivos de texto plano en `/etc/`
- Fácil de editar, copiar, versionar y compartir
- Fácil hacer backup de configuraciones específicas

**Implicación práctica**: En Linux es trivial hacer backup de una configuración: simplemente copiamos el archivo correspondiente de `/etc/`. También es fácil compartir configuraciones o buscar ayuda en Internet porque podemos copiar y pegar fragmentos de archivos de configuración.

### Seguridad

**Windows**:
- Objetivo principal de virus y malware
- Requiere antivirus
- Actualizaciones a veces problemáticas
- Muchos exploits conocidos

**Linux**:
- Muy pocos virus (aunque existen)
- No requiere antivirus tradicional en uso normal
- Modelo de permisos más restrictivo por diseño
- Actualizaciones generalmente sin problemas

**Implicación práctica**: En Linux no necesitáis antivirus (salvo que el servidor actúe de gateway para clientes Windows). El modelo de permisos de UNIX hace que sea muy difícil que un virus se propague o afecte al sistema.

### Gestión de procesos

**Windows**:
- Servicios gestionados por Services.msc
- Tareas programadas en Programador de tareas
- Inicio automático configurado en el Registro

**Linux**:
- Servicios gestionados por systemd (comandos `systemctl`)
- Tareas programadas con cron
- Inicio automático configurado en archivos de texto

**Implicación práctica**: En Linux, comandos como `systemctl status`, `systemctl start`, `systemctl enable` son fundamentales para gestionar servicios. Es más directo y potente que navegar por menús gráficos.

### Nombres de cosas comunes

Para facilitar la transición, aquí hay una tabla de equivalencias:

| Windows | Linux | Función |
|---------|-------|---------|
| C:\Users | /home | Carpetas de usuarios |
| C:\Program Files | /usr o /opt | Programas instalados |
| Panel de Control | Configuración (Settings) | Configuración del sistema |
| Administrador de tareas | Monitor del sistema (gnome-system-monitor) | Ver procesos y recursos |
| Explorador de archivos | Nautilus (Archivos) | Gestor de archivos |
| Bloc de notas | gedit o nano | Editor de texto |
| cmd / PowerShell | bash (Terminal) | Línea de comandos |
| Administrador de discos | GParted | Gestión de particiones |
| Registro de Windows | /etc/ (archivos de configuración) | Configuración del sistema |

### Actitud y aprendizaje

La diferencia más importante no es técnica, sino de mentalidad:

**En Windows**: Si algo no funciona con la interfaz gráfica, es un problema. Esperamos que todo tenga un botón o un asistente.

**En Linux**: Si algo no funciona con la interfaz gráfica, usamos la terminal. Es normal consultar documentación, leer mensajes de error y buscar soluciones en Internet.

Linux premia la curiosidad y el aprendizaje continuo. No tengáis miedo de experimentar, leer man pages (`man comando`) y buscar en Google cuando algo no funcione. La comunidad Linux es enorme y hay soluciones documentadas para prácticamente cualquier problema que encontréis.

