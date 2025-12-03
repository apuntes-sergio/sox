---
title: Escritorio Remoto 
description:  Gestión del dominio en Windows Server. Escritorio Remoto 
---

## Escritorio Remoto (Remote Desktop)

El Escritorio Remoto es un servicio esencial que permite a los administradores acceder a la interfaz gráfica completa del servidor desde una ubicación remota. Es la herramienta principal para la gestión y mantenimiento de Windows Server, ya que permite ejecutar cualquier aplicación y realizar configuraciones como si se estuviera físicamente frente a la máquina.

### Protocolo y Conectividad

La funcionalidad de Escritorio Remoto se basa en un protocolo de capa de aplicación específico desarrollado por Microsoft. Es fundamental que los administradores conozcan el puerto asociado para asegurar la conectividad a través de la red local y gestionar correctamente las políticas de *firewall*.

* **Protocolo:** Utiliza el **Protocolo de Escritorio Remoto (RDP)**, diseñado para proporcionar una interfaz gráfica eficiente y de bajo ancho de banda.
* **Puerto Predeterminado:** RDP utiliza por defecto el puerto **TCP 3389**. Si se requiere acceso externo (Internet), a menudo se recomienda cambiar este puerto predeterminado para reducir la exposición a escaneos automatizados.
* **Firewall:** Es obligatorio que el puerto TCP 3389 esté abierto en el *firewall* de Windows Server para permitir las conexiones entrantes. La configuración debe ser lo más restrictiva posible, limitando el tráfico solo a las direcciones IP o subredes de origen autorizadas.

### Configuración de Seguridad

Para garantizar la integridad del servidor durante una conexión remota, es necesario habilitar y aplicar medidas de seguridad avanzadas en la configuración de RDP:

* **Autenticación a Nivel de Red (NLA - Network Level Authentication):** Esta característica es una medida de seguridad crítica. Requiere que el cliente se autentique antes de que se establezca la sesión completa de RDP. Esto consume menos recursos del servidor y previene ciertos ataques de denegación de servicio, ya que el servidor no carga la sesión gráfica hasta que las credenciales son validadas.
* **Grupos de Usuarios:** Por defecto, solo los miembros del grupo local **Administradores** y del grupo **Usuarios de Escritorio Remoto** pueden iniciar una sesión RDP. Es una buena práctica agregar solo las cuentas de usuario necesarias a este último grupo.

### Habilitación del Servicio

La habilitación del servicio de Escritorio Remoto se realiza de la siguiente manera:

1.  Acceder a la **Configuración Remota** del sistema (disponible en las propiedades avanzadas del sistema).
2.  Seleccionar la opción **Permitir las conexiones remotas a este equipo**.
3.  Marcar la casilla **Permitir solo las conexiones de equipos que ejecuten Escritorio remoto con Autenticación a nivel de red (más seguro)** para forzar el uso de NLA.

---

El siguiente tema es **Red VPN**. Para configurar una **Red VPN (Virtual Private Network)** en Windows Server, ¿qué **rol o servicio** específico de servidor necesitamos instalar, y cuál es el **protocolo de túnel** más seguro y moderno que se recomienda usar hoy en día para las conexiones remotas?