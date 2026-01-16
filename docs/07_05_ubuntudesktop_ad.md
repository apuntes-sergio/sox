---
title: Ubuntu Desktop. Integración con Active Directory
description: Gestión de sistemas Linux. Integración de Ubuntu Desktop y Server en entornos Windows
---

## Integración de Ubuntu Desktop con Active Directory

Ahora llega la parte más interesante: vamos a unir este equipo Ubuntu Desktop al dominio Active Directory de Windows Server que creamos en temas anteriores. Una vez completada la integración, los usuarios del dominio podrán iniciar sesión en Ubuntu con sus credenciales corporativas, exactamente igual que lo hacen en los equipos Windows.

### Introducción a la integración

La integración de Linux con Active Directory es una necesidad real en empresas con infraestructuras híbridas. Permite:

**Centralizar la gestión de usuarios**: Un único lugar (Active Directory) gestiona las credenciales de todos los equipos, tanto Windows como Linux.

**Single Sign-On (SSO)**: Los usuarios inician sesión una vez y pueden acceder a recursos tanto en servidores Windows como Linux sin volver a autenticarse.

**Políticas centralizadas**: Aunque limitadas comparadas con GPOs de Windows, pueden aplicarse ciertas políticas a equipos Linux unidos al dominio.

**Auditoría unificada**: Todos los inicios de sesión, tanto Windows como Linux, quedan registrados en el controlador de dominio.

**Experiencia consistente para usuarios**: Los usuarios no necesitan recordar diferentes credenciales para diferentes sistemas.

### Tecnologías involucradas

La integración utiliza varias tecnologías que trabajan juntas:

**Realmd**: Servicio que simplifica la unión a dominios Active Directory. Es una capa de abstracción que facilita todo el proceso.

**SSSD** (System Security Services Daemon): Demonio que gestiona la autenticación y el acceso a recursos de red. Hace de intermediario entre Linux y Active Directory.

**Kerberos**: Protocolo de autenticación que usa Active Directory. Permite autenticación segura sin enviar contraseñas por la red.

**Samba**: Aunque no lo usaremos para compartir archivos en el Desktop, sus utilidades nos permiten comunicarnos con dominios Windows.

### Guías detalladas

El proceso de integración es bastante extenso y requiere múltiples pasos. Afortunadamente, existe documentación externa muy completa que explica todo el proceso paso a paso con capturas de pantalla.

Vais a seguir estas dos guías en orden:

**Parte 1: Preparación e instalación de paquetes**

[Unir un cliente Ubuntu 24.04 a un dominio de Active Directory sobre Windows Server 2022 - Parte 1](https://somebooks.es/unir-un-cliente-ubuntu-24-04-a-un-dominio-de-active-directory-sobre-windows-server-2022-parte-1/)

Esta primera parte cubre:

- Requisitos previos de red y DNS
- Instalación de todos los paquetes necesarios
- Configuración inicial del sistema

**Parte 2: Unión al dominio y configuración**

[Unir un cliente Ubuntu 24.04 a un dominio de Active Directory sobre Windows Server 2022 - Parte 2](https://somebooks.es/unir-un-cliente-ubuntu-24-04-a-un-dominio-de-active-directory-sobre-windows-server-2022-parte-2/)

Esta segunda parte cubre:

- Unión real al dominio usando `realm`
- Configuración de SSSD
- Verificación de la integración
- Configuración de carpetas personales para usuarios del dominio
- Resolución de problemas comunes

### Aspectos importantes a tener en cuenta

Antes de comenzar el proceso, tened en cuenta estos puntos críticos para nuestro entorno ISCASOX:

**Configuración de red previa**:

Vuestro Ubuntu Desktop debe tener configurado el Windows Server como DNS primario. Esto es absolutamente crítico porque Active Directory depende completamente de DNS para funcionar.

Debéis configurarlo usando Netplan mediate fichero yaml tal y como hemos visto en apartados anteriores:

Configurar:

   - Dirección IP: `192.168.100.XX` (elegid una libre en la red de departamentos)
   - Máscara: `255.255.255.0`
   - Puerta de enlace: `192.168.100.1` (el Windows Server)
   - DNS: `192.168.100.1` (muy importante: el Windows Server) y `1.1.1.1` por si acaso para acceder a intenet
   - Dominios de búsqueda: `DOMXXX.local` (vuestro dominio)

**Sincronización de hora**:

Kerberos es extremadamente sensible a diferencias de hora. Si el reloj del Ubuntu y del Windows Server difieren en más de 5 minutos, la autenticación fallará.

Ubuntu Desktop normalmente sincroniza la hora automáticamente, pero verificadlo:

```bash
timedatectl
```

Si `System clock synchronized: yes`, estáis bien. Si no, instalad y configurad chrony:

```bash
sudo apt install -y chrony
```

**Nombres de dominio**:

Sustituir correctamente `DOMXXX.local` por el nombre real de vuestro dominio en todos los comandos y archivos de configuración. Un error aquí hará que nada funcione.

### Diferencias respecto a unir un Windows al dominio

Aunque conceptualmente es similar a unir un equipo Windows al dominio, hay diferencias importantes:

**En Windows**:

- Proceso completamente gráfico
- Unos pocos clics en Propiedades del sistema
- Reinicio y listo

**En Ubuntu Desktop**:

- Requiere instalar varios paquetes
- Configuración mediante archivos de texto y comandos
- Más pasos pero también más control sobre el proceso
- No requiere reinicio (aunque es recomendable)

### Qué verificar después de la integración

Una vez completadas ambas guías, debéis verificar que todo funciona:

**Verificar que el equipo está en el dominio**:

```bash
realm list
```

Debe mostrar vuestro dominio con información detallada.

**Listar usuarios del dominio**:

```bash
getent passwd | grep DOMXXX
```

Debéis ver usuarios del Active Directory.

**Probar inicio de sesión**:

Cerrad sesión e intentad iniciar sesión con un usuario del dominio. En la pantalla de login:

1. Si no aparece el usuario, clickad en "Not listed?" o "¿No está en la lista?"
2. Introducid: `usuario@DOMXXX.local` o `DOMXXX\usuario`
3. Introducid la contraseña del dominio

Si podéis iniciar sesión, ¡enhorabuena! La integración es correcta.

**Verificar carpeta personal**:

Una vez dentro con un usuario del dominio, verificad que se ha creado su carpeta personal:

```bash
pwd
ls -la
```

Debería estar en `/home/usuario@domxxx.local/` o similar, y contener las subcarpetas típicas (Documentos, Descargas, etc.).

### Limitaciones de la integración

Es importante entender que aunque el equipo esté unido al dominio, la experiencia no es exactamente igual a la de Windows:

**Las GPOs de Windows no se aplican** a equipos Linux. No podemos usar Group Policy para configurar Ubuntu. Existen alternativas como Ansible o Chef para gestión centralizada de Linux, pero son diferentes sistemas.

**Los perfiles móviles de Windows no funcionan** en Linux. Los perfiles son sistemas diferentes. Cada sistema mantiene su configuración independiente.

**Algunas aplicaciones Windows** que dependen de características específicas de Windows no funcionarán en Linux, aunque las credenciales sean las mismas.

**Los scripts de inicio de sesión** (.bat, .vbs) de Windows no se ejecutan en Linux. Deberíamos crear scripts equivalentes en bash si fuera necesario.

A pesar de estas limitaciones, la integración es muy útil porque:

- Centraliza usuarios y contraseñas
- Permite acceso a recursos compartidos en servidores Windows
- Simplifica la gestión cuando tenemos estaciones de trabajo Linux en la empresa
- Los usuarios no necesitan recordar credenciales diferentes
