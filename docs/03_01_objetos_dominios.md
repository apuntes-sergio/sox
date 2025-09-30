---
title: Objetos del Dominio en Active Directory
description:  Gestión del dominio en Windows Server. Gestión de usuarios y grupos
---

El **Directorio Activo (Active Directory)** es, en esencia, una base de datos jerárquica que almacena objetos que representan las entidades administrables dentro de una red informática. En el contexto de un dominio basado en **Windows Server**, esta base de datos centralizada permite la consulta por parte de todos los equipos miembros del dominio y puede ser modificada por los **controladores de dominio**.

La administración de un dominio implica la creación, configuración y mantenimiento de los objetos del directorio que representan los recursos de la red (usuarios, grupos, equipos, impresoras, etc.), así como las relaciones entre ellos (por ejemplo, qué usuarios pertenecen a qué grupos, o qué grupos tienen acceso a determinados recursos compartidos).


## Elementos estructurales de Active Directory

La organización de los objetos en Active Directory se basa en una jerarquía lógica que facilita la administración y el control de acceso. Los principales elementos son:

| Elemento               | Descripción |
|------------------------|-------------|
| **Dominio**            | Unidad estructural básica que agrupa todos los objetos de la red. Se identifica mediante notación DNS (por ejemplo, `empresa.local`). |
| **Unidad Organizativa (OU)** | Contenedor lógico que permite agrupar objetos relacionados, como usuarios, equipos o impresoras de un mismo departamento. Equivale a una carpeta en un sistema de archivos. |
| **Grupo**              | Conjunto de objetos del mismo tipo, generalmente usuarios. Se utilizan para asignar permisos de forma colectiva, simplificando la administración. |
| **Objeto**             | Representación individual de un recurso de red: usuario, equipo, carpeta compartida, impresora, etc. Cada objeto tiene atributos específicos según su tipo. |

!!!note "Nota técnica:"

    Los dominios en Active Directory se identifican mediante la notación **DNS (Domain Name System)**, lo que permite una integración más sencilla con servicios de red y resolución de nombres.

## Identificación de objetos en Active Directory

Cada objeto en Active Directory se identifica mediante un nombre, que puede expresarse de tres formas distintas:

### Ejemplo: Usuario `sergio` en el dominio `microsoft.com`

1. **Nombre completo relativo LDAP**  
   ```plaintext
   CN=sergio
   ```
   Donde `CN` significa *Common Name* (nombre común).

2. **Nombre completo LDAP**  
   ```plaintext
   CN=sergio, DC=microsoft, DC=com
   ```
   Incluye el dominio y todos los contenedores jerárquicos.

3. **Nombre canónico**  
   ```plaintext
   microsoft.com/sergio
   ```

### Ejemplo avanzado: Usuario `srey` en una estructura jerárquica

Supongamos que el usuario `srey` se encuentra dentro de la unidad organizativa `sor`, que a su vez está dentro de `SMR`, todo ello en el dominio `aula224.lan`. Las formas de nombrarlo serían:

- **Nombre completo LDAP**  
  ```plaintext
  CN=srey, OU=sor, OU=SMR, DC=aula224, DC=lan
  ```

- **Nombre canónico**  
  ```plaintext
  aula224.lan/SMR/sor/srey
  ```

!!!note "Equivalencia técnica:" 

    Esta estructura es comparable a una ruta absoluta en un sistema de archivos, lo que facilita la localización precisa de cualquier objeto dentro del dominio.



!!!example "Ejemplo: Unidad Organizativa `MiUnidadOrganizativa` en `microsoft.com`"

  | Forma de nombre | Ejemplo |
  |------------------|---------|
  | **Relativo LDAP** | `OU=MiUnidadOrganizativa` |
  | **Completo LDAP** | `OU=MiUnidadOrganizativa, DC=microsoft, DC=com` |
  | **Canónico**      | `microsoft.com/MiUnidadOrganizativa` |


## Atributos de los objetos

Cada objeto en Active Directory posee una serie de **atributos** que lo describen. Estos atributos varían según el tipo de objeto:

- **Usuario**: nombre, apellidos, dirección de correo electrónico, nombre de inicio de sesión, etc.
- **Grupo**: nombre del grupo, tipo de grupo (seguridad o distribución), miembros asociados, etc.
- **Equipo**: nombre del equipo, sistema operativo, ubicación, etc.

!!!note "Herramienta principal:" 

    La gestión de estos objetos y sus atributos se realiza principalmente mediante la consola **Usuarios y Equipos de Active Directory**, accesible desde los controladores de dominio.

