---
title: Introducción. Redes heterogéneas
description:  Gestión del dominio en Windows Server. Integración de Ubuntu 24.04 LTS en Dominio Windows Server
---

## Introduccion a las redes hetereogéneas

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

