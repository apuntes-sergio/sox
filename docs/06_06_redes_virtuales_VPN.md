---
title: Red VPN 
description:  Gestión del dominio en Windows Server. Red VPN
---

Una Red Privada Virtual (VPN) establece una conexión segura y cifrada, denominada **túnel**, a través de una red pública, generalmente Internet. Su objetivo principal es permitir que los usuarios remotos accedan a los recursos de la red corporativa como si estuvieran conectados localmente, manteniendo la privacidad e integridad de los datos transmitidos mediante el cifrado.

### Implementación en Windows Server

Para habilitar la funcionalidad de VPN en Windows Server, se requiere la instalación de un rol específico encargado de gestionar y administrar estas conexiones de acceso remoto.

* **Rol de Servidor:** Se debe instalar el rol **Acceso Remoto (Remote Access)**. Dentro de este rol, el servicio clave que gestiona las conexiones VPN y las políticas de acceso es **Enrutamiento y Acceso Remoto (RRAS - Routing and Remote Access Service)**.
* **Función:** RRAS se encarga de crear y gestionar los puertos virtuales que aceptan las conexiones entrantes, así como de aplicar las políticas de autenticación y red para los usuarios remotos.

### Protocolos de Túnel

Los protocolos de túnel son esenciales, ya que definen el método de encapsulación y cifrado de los datos. La elección del protocolo es crucial para garantizar la seguridad y la compatibilidad con diferentes *firewalls*.

* **SSTP (Secure Socket Tunneling Protocol):** Es un protocolo moderno muy recomendado. Utiliza **SSL/TLS** y opera sobre el puerto **TCP 443** (el mismo que HTTPS), lo que le permite atravesar con éxito la mayoría de los *firewalls* que a menudo restringen otros puertos.
* **IKEv2/IPsec:** Es considerado uno de los protocolos más seguros y robustos actualmente. Es altamente estable y soporta la característica de **VPN Reconnect**, siendo ideal para usuarios con movilidad o dispositivos que experimentan caídas temporales de red.
* **L2TP/IPsec:** Un protocolo seguro que depende de la suite **IPsec** para el cifrado y la autenticación. Es ampliamente compatible, pero requiere que los puertos **UDP 500 y 4500** estén abiertos.
* **PPTP (Point-to-Point Tunneling Protocol):** Es un protocolo heredado, cuyo uso está **desaconsejado** en entornos de producción debido a sus debilidades de seguridad conocidas.

### Seguridad y Control de Acceso

La seguridad de la VPN no depende solo del cifrado del túnel, sino también de una autenticación estricta:

* **Integración con Active Directory:** El servidor VPN valida las credenciales del usuario directamente contra la base de datos de Active Directory.
* **Servidor de Políticas de Red (NPS - Network Policy Server):** A menudo se utiliza conjuntamente con RRAS para establecer reglas y políticas detalladas que controlan el acceso: qué usuarios pueden conectarse, cuánto tiempo y qué recursos de la red interna tienen permitido alcanzar.
