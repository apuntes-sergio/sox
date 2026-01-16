---
title: Introducción. Redes heterogéneas
description: Gestión de sistemas Linux. Integración de Ubuntu Desktop y Server en entornos Windows
---

## Introducción a las redes heterogéneas

Hasta ahora hemos trabajado exclusivamente con Windows Server. Hemos montado un dominio, compartido recursos y aplicado directivas en un entorno completamente Microsoft. Sin embargo, cuando visitéis empresas medianas o grandes, os daréis cuenta de que lo habitual es encontrar **Windows y Linux trabajando juntos**. A esto se le llama una **red heterogénea**.

<figure markdown="span" align="center">
  ![Image title](./imgs/ubuntu/redes_heterogenea.png){ width="70%"}
  <figcaption>Esquema de redes heterogéneas</figcaption>
</figure>

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

Nosotros usaremos **Ubuntu 24.04 LTS** en sus dos variantes: Desktop para estaciones de trabajo y Server para servicios de red. Ambas comparten la misma base, lo que facilita la transición entre una y otra.

Ubuntu es completamente gratuita y tiene soporte oficial hasta 2029. Cuenta con una comunidad enorme, lo que facilita encontrar ayuda y documentación. Es la distribución más usada tanto en escritorios Linux como en entornos cloud (AWS, Azure, Google Cloud), por lo que lo que aprendáis aquí os servirá en entornos profesionales reales. Además, es especialmente adecuada para aprender por su facilidad de uso y abundante documentación.

### Diferencias clave con Windows

**Interfaz de administración**: Windows tiene ventanas, menús y herramientas gráficas para todo. Linux Desktop también tiene interfaz gráfica muy completa, pero Linux Server se administra principalmente por **terminal** (línea de comandos). Aunque pueda parecer más difícil al principio, consume muchísimos menos recursos y facilita enormemente la automatización de tareas mediante scripts.

**Sistema de archivos**: 

En Windows tenemos letras de unidad separadas (C:, D:, E:) para cada disco o partición. En Linux existe un único árbol jerárquico que empieza en `/` (raíz). Los discos adicionales se "montan" como carpetas dentro de este árbol.

Por ejemplo, en Windows tendríamos `D:\datos`. En Linux sería `/datos` y el disco físico se monta en esa ubicación.

**Instalación de programas**:

En Windows descargamos ejecutables .exe y hacemos doble clic. En Linux usamos **gestores de paquetes** como `apt` o `snap`, que funcionan como una tienda de aplicaciones pero mucho más potente.

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

Vamos a trabajar con Linux de forma progresiva y práctica, integrándolo completamente en nuestra red ISCASOX. No serán sistemas aislados, sino parte integral de nuestra infraestructura empresarial.

El tema se estructura en cinco bloques principales que siguen un orden lógico de complejidad creciente:

### Bloque 1: Ubuntu Desktop - Primeros pasos con Linux

Comenzaremos con Ubuntu Desktop porque es más familiar para quienes venís de Windows. Este primer contacto incluye:

**Instalación y configuración inicial**: Crearemos una máquina virtual con Ubuntu Desktop y realizaremos la configuración post-instalación básica. Instalaremos Guest Additions de VirtualBox para mejorar la experiencia de uso.

**Gestión de software**: Aprenderemos a instalar aplicaciones usando `apt`, `snap` y la tienda gráfica de Ubuntu. Entenderemos las diferencias entre estos sistemas y cuándo usar cada uno.

**Configuración de red**: Veremos cómo funciona NetworkManager y Netplan, configurando la red tanto desde interfaz gráfica como mediante archivos de configuración.

**Integración con Active Directory**: Uniremos el equipo Ubuntu Desktop al dominio Windows, permitiendo que los usuarios corporativos inicien sesión con sus credenciales del dominio.

Este bloque servirá como introducción y repaso a sistemas Linux y tiene como objetivo dar una base sólida de Linux desde el punto de vista del usuario final, teniendo en cuenta que la finalidad es usar este sistema dentro de dominios Windows y antes de pasar a la administración de servidores.

### Bloque 2: Ubuntu Server - Instalación y configuración básica

Una vez familiarizados con Linux Desktop, pasaremos a Ubuntu Server. Este bloque incluye:

**Instalación de Ubuntu Server**: Crearemos un servidor sin interfaz gráfica, gestionable únicamente por terminal.

**Configuración de red con Netplan**: Aprenderemos a configurar IPs estáticas y parámetros de red mediante archivos YAML.

**SSH para administración remota**: Configuraremos el acceso remoto seguro al servidor, permitiéndonos administrarlo desde nuestro equipo Windows sin necesidad de acceder físicamente a la consola.

**LVM (Logical Volume Manager)**: Dominaremos la gestión avanzada de almacenamiento, creando y ampliando volúmenes lógicos de forma flexible sin interrumpir servicios.

### Bloque 3: Servicios de compartición - Samba y Active Directory

Este es el bloque de integración entre Windows y Linux:

**Samba básico**: Instalaremos y configuraremos Samba para compartir archivos con clientes Windows, creando recursos compartidos básicos.

**Integración con Active Directory usando Winbind**: Uniremos el servidor Linux al dominio Windows, configurando Samba para que respete permisos y grupos del Active Directory. Los usuarios del dominio podrán autenticarse en Linux y acceder a recursos compartidos con sus credenciales corporativas.

**Gestión de permisos**: Aprenderemos a gestionar ACLs de Windows en carpetas Linux, permitiendo configurar permisos desde la interfaz gráfica de Windows que se respetarán en el servidor Linux.

### Bloque 4: Administración avanzada - Automatización y copias de seguridad

Con la infraestructura funcionando, aprenderemos a mantenerla y automatizar tareas:

**Planificación de tareas con cron**: Automatizaremos tareas periódicas como copias de seguridad, limpieza de logs, actualizaciones programadas, etc. Entenderemos la sintaxis de cron y crearemos nuestros propios trabajos programados.

**Copias de seguridad con rsync y tar**: Implementaremos estrategias de backup utilizando `rsync` para sincronización incremental de archivos y `tar` para archivos comprimidos. Crearemos scripts automatizados de backup.

**Systemd timers**: Como alternativa moderna a cron, aprenderemos a usar systemd timers para tareas programadas con mejor integración en el sistema.

Este bloque os dará las herramientas necesarias para que vuestros servidores funcionen de forma autónoma y fiable.

### Bloque 5: Mantenimiento y control - Monitorización y logs

Finalizaremos con las herramientas de diagnóstico y resolución de problemas:

**Gestión de logs con journald**: Aprenderemos a usar `journalctl` para consultar y analizar los registros del sistema, diagnosticando problemas y monitorizando la actividad.

**Monitorización del sistema**: Utilizaremos herramientas como `htop`, `iotop`, `netstat` y comandos de análisis de rendimiento para vigilar el estado del servidor.

**Análisis de espacio en disco**: Herramientas como `df`, `du` y `ncdu` para controlar el uso del almacenamiento.

**Resolución de incidencias**: Metodología para diagnosticar y resolver problemas comunes en servidores Linux.

Este bloque cierra el ciclo de administración de sistemas, dotándoos de herramientas para mantener servidores en producción y resolver problemas cuando surjan.

---

## Estructura del aprendizaje

Esta organización no es casual, sino que sigue una progresión pedagógica pensada:

**De lo familiar a lo desconocido**: Empezamos con Desktop porque su interfaz gráfica es más parecida a Windows, reduciendo la barrera de entrada.

**De cliente a servidor**: Primero usamos Linux como cliente (Desktop en el dominio), después como servidor (Server ofreciendo servicios). Esto replica cómo funcionan las redes reales.

**De lo básico a lo avanzado**: Primero instalamos y configuramos, después compartimos recursos, y finalmente automatizamos y monitorizamos.

**De la teoría a la práctica**: Cada concepto se aplica inmediatamente en nuestro proyecto ISCASOX, dándole sentido práctico.

Cada unos de los bloques indicados se deben realizar en tiempo y forma adecuado, de forma que se plantearán una o dos actividades entregables en cada uno de los bloques.

---

!!!tip "Cómo trabajar en este tema"

    **En clase** explicaremos los conceptos fundamentales y haremos demostraciones en vivo de los procesos más complejos.

    **En los apuntes** encontraréis guías paso a paso muy detalladas que podréis seguir a vuestro ritmo.

    **Aspectos importantes a tener en cuenta**:

    Haced **snapshots** (instantáneas) de las máquinas virtuales antes de realizar cambios importantes. Si algo falla, simplemente restauráis la instantánea y volvéis a intentarlo sin perder todo el trabajo anterior.

    No tengáis miedo a "romper" el sistema. Estáis trabajando en máquinas virtuales, así que experimentad tranquilamente. Los errores son parte del aprendizaje y en un entorno virtualizado no hay riesgo real.

    **No copiéis comandos sin leer qué hacen**. Es fundamental que entendáis cada comando antes de ejecutarlo. Los errores en Linux suelen ser muy informativos si os tomáis el tiempo de leerlos con atención.

    **Cuando algo no funcione**, seguid estos pasos:

    Primero, leed el mensaje de error completo, palabra por palabra. Los mensajes de error en Linux suelen ser muy descriptivos y a menudo indican exactamente qué está mal.

    Segundo, buscad el error exacto en Google (copiad el mensaje tal cual). La comunidad Linux es enorme y es muy probable que alguien haya tenido el mismo problema.

    Tercero, consultad al profesor si seguís atascados después de intentar solucionarlo por vuestra cuenta. Traed anotado qué habéis probado para que podamos diagnosticar más eficientemente.

    **Herramientas de ayuda integradas**: Linux tiene sistemas de ayuda muy potentes:

    ```bash
    man nombre_comando    # Manual completo y detallado del comando
    comando --help        # Ayuda rápida con las opciones principales
    ```

    Los manuales (`man pages`) son extremadamente completos. Aprender a leerlos es una habilidad fundamental en Linux.

    **Documentación online**: Ubuntu tiene documentación oficial excelente en [help.ubuntu.com](https://help.ubuntu.com) y [ubuntu.com/server/docs](https://ubuntu.com/server/docs). Son recursos oficiales y fiables.

    **Entorno de trabajo recomendado**:

    - Tened una terminal abierta en el servidor Linux
    - Tened estos apuntes abiertos en vuestro navegador
    - Tened un editor de texto (Bloc de notas, Notepad++, VSCode) para anotar comandos y configuraciones
    - Usad SSH desde Windows para administrar el servidor cómodamente (lo configuraremos en el tema)

    **Existen también herramientas gráficas opcionales** como **Webmin** o **Cockpit** que proporcionan paneles web de administración. No son obligatorias ni las usaremos en el curso, pero pueden ayudar cuando estéis empezando a familiarizaros con el sistema si queréis explorarlas por vuestra cuenta.