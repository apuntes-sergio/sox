---
title: Controlador de Dominio Secundario
description:  Gestión del dominio en Windows Server. Controlador de Dominio Secundario
---

## Controlador de Dominio Secundario (Additional Domain Controller - ADC)

Una infraestructura de red que depende de Active Directory para la autenticación y la gestión de usuarios requiere alta disponibilidad. Un **Controlador de Dominio Secundario (ADC)**, también conocido como Controlador de Dominio Adicional, es un servidor instalado en el mismo dominio lógico que el Controlador de Dominio primario. Su propósito es garantizar la continuidad del servicio de directorio y distribuir la carga de las solicitudes de autenticación en la red.

### Objetivo Principal: Redundancia y Disponibilidad

El objetivo principal de instalar un ADC es eliminar los **puntos únicos de fallo (SPOF)** en la autenticación. Si el Controlador de Dominio principal falla, el ADC asume automáticamente las peticiones de inicio de sesión y gestión, evitando la interrupción del servicio para los usuarios.

* **Tolerancia a Fallos:** Si el DC principal deja de funcionar, el ADC sigue autenticando usuarios y procesando cambios en el directorio sin intervención manual.
* **Balanceo de Carga:** El ADC ayuda a distribuir las solicitudes de autenticación (inicios de sesión) y las consultas DNS entre varios servidores, mejorando el rendimiento general, especialmente en redes grandes.
* **Autenticación Local:** En entornos con múltiples ubicaciones geográficas, un ADC instalado en cada sede garantiza que los usuarios se autentiquen rápidamente contra un servidor local.

### Instalación y Replicación

La instalación de un ADC se realiza mediante el asistente de promoción del servidor, que gestiona automáticamente el proceso de copia de la base de datos de Active Directory.

* **Proceso:** El servidor se une primero al dominio como un servidor miembro normal. Posteriormente, se promueve al rol de Controlador de Dominio Adicional utilizando el asistente de **Servicios de Dominio de Active Directory (AD DS)**.
* **Replicación:** El paso clave es la **Replicación de Active Directory**. La base de datos del directorio (`NTDS.DIT`) y las Directivas de Grupo se copian y mantienen sincronizadas entre el DC principal y el ADC. Esta replicación se realiza automáticamente y de forma periódica, asegurando la coherencia de los datos en toda la infraestructura.

### Roles FSMO (Flexible Single Master Operations)

Aunque todos los Controladores de Dominio contienen una copia de la base de datos, ciertos roles críticos (FSMO) son únicos y solo pueden ser poseídos por un DC a la vez.

* **Redundancia de Roles:** Para mejorar la redundancia, los roles FSMO se pueden distribuir entre el DC principal y el ADC. En caso de fallo prolongado del DC principal, el administrador puede realizar una **toma de control (*seizure*)** de los roles FSMO para transferirlos al ADC.
