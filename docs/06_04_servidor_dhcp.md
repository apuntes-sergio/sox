---
title: Servidor DHCP
description:  Gestión del dominio en Windows Server. Servidor DHCP
---


El Servidor DHCP *(Dynamic Host Configuration Protocol)* es un servicio de infraestructura crucial en cualquier red moderna. Su función principal es **asignar dinámicamente direcciones IP** y otros parámetros de configuración de red a los clientes (hosts) de forma automática. Esto elimina la necesidad de configurar manualmente cada dispositivo y reduce drásticamente los errores y los conflictos de direcciones IP, simplificando la administración de la red.

Como en otros servicios, debemos instalar el rol correspondiente para poder usarlo en nuestro servidor.

<figure markdown="span" align="center">
    ![](./imgs/adicional/rol_DHCP.png){ width="85%" }
    <figcaption>Instalación de rol DHCP.</figcaption>
</figure>

## Parámetros de Configuración Esenciales

Para que un cliente pueda comunicarse eficientemente, necesita una serie de parámetros que le permitan interactuar tanto con la red local (LAN) como con redes externas (Internet).

| Parámetro | Propósito | Esencial para Comunicación Externa |
| :--- | :--- | :--- |
| **Dirección IP** | Identificador único del host en la red. | Sí |
| **Máscara de Subred** | Define qué parte de la dirección IP es la red y qué parte es el host. | Sí |
| **Puerta de Enlace Predeterminada** | Es la dirección IP del *router* o dispositivo que permite la salida de los paquetes de datos hacia redes externas (Internet). | **Sí** |
| **Servidores DNS** | Direcciones IP de los servidores que resuelven nombres de dominio (URLs) a direcciones IP. | **Sí** |

Los **tres parámetros esenciales** para que un cliente se comunique **fuera de su red local** son la **Dirección IP**, la **Puerta de Enlace Predeterminada** y los **Servidores DNS**.

## Proceso de Asignación DORA

La comunicación entre el cliente y el servidor DHCP sigue un protocolo de cuatro pasos conocido por sus siglas: **DORA** (Discovery, Offer, Request, Acknowledge). Este proceso asegura que la asignación sea coordinada.

* **Discovery (Descubrimiento):** El cliente envía un mensaje de difusión (*broadcast*) a la red para localizar servidores DHCP.
* **Offer (Oferta):** Los servidores DHCP disponibles responden ofreciendo una dirección IP libre al cliente.
* **Request (Solicitud):** El cliente selecciona una de las ofertas y solicita formalmente esa dirección IP específica.
* **Acknowledge (Reconocimiento):** El servidor asigna la dirección de forma definitiva y envía el paquete de configuración completo.

## Componentes de Gestión

La administración del servicio DHCP se realiza mediante la configuración de los siguientes elementos dentro de la consola de Windows Server:

* **Ámbito (Scope):** Es el rango completo de direcciones IP válidas y disponibles que el servidor puede asignar a un segmento de red. Se debe configurar al menos un ámbito para que el servidor funcione.
* **Concesión (Lease):** Es el período de tiempo (tiempo de vida) durante el cual un cliente puede utilizar la dirección IP asignada. Al expirar, el cliente debe renovar la concesión o solicitar una nueva.
* **Reserva (Reservation):** Es una asignación permanente de una dirección IP específica a un cliente concreto, identificado por su dirección **MAC**. Se utiliza para dispositivos que necesitan una IP estática garantizada (como servidores o impresoras).
* **Exclusión (Exclusion):** Son rangos de direcciones IP dentro del ámbito que el servidor tiene prohibido asignar. Se utilizan para direcciones que ya están asignadas de forma estática a equipos de infraestructura (*routers*, otros servidores, etc.).

