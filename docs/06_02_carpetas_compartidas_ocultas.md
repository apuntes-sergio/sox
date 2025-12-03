---
title: Recursos Compartidos Ocultos
description:  Gestión del dominio en Windows Server. Recursos Compartidos Ocultos
---


## Carpetas Compartidas Ocultas y Administrativas

Los recursos compartidos son una función esencial de Windows Server para el acceso a datos en red. Sin embargo, en entornos administrados, a menudo es necesario ocultar la existencia de ciertos recursos a la vista de los usuarios comunes para fines de limpieza o para mantener accesos críticos fuera del alcance no intencionado. Este apartado detalla el mecanismo para configurar recursos compartidos ocultos y explica la naturaleza de los recursos administrativos que Windows Server crea automáticamente.

### Recursos Compartidos Ocultos (Hidden Shares)

La funcionalidad de recursos compartidos ocultos permite a los administradores controlar la visibilidad de un *share* sin necesidad de cambiar los permisos de acceso. Es crucial entender que esta configuración no es una medida de seguridad que reemplace a los permisos NTFS, sino una técnica de **ofuscación**.

* **Mecanismo de Ocultación:** El recurso se hace invisible al navegar por el servidor añadiendo el símbolo de dólar (**$**) al final de su nombre. Por ejemplo, una carpeta llamada `Almacen` se comparte como `Almacen$`.
* **Procedimiento de Configuración:** La ocultación se configura en las propiedades de la carpeta, a través de la opción de **Uso Compartido Avanzado**, al incluir el `$` en el campo **Nombre del recurso compartido**.
* **Acceso Remoto:** Un cliente solo puede acceder al recurso si conoce y especifica la ruta **UNC (Universal Naming Convention)** completa, incluyendo el símbolo de dólar. Ejemplo: `\\NombreServidor\Almacen$`.

<figure markdown="span" align="center">
    ![](./imgs/adicional/carpeta_oculta.png){ width="85%" }
    <figcaption>Carpeta compartida oculta.</figcaption>
</figure>

### Recursos Compartidos Administrativos

Windows Server crea de forma predeterminada ciertos recursos compartidos ocultos que son fundamentales para la administración remota. Estos recursos siguen la misma convención de nomenclatura (terminan en `$`) y están reservados exclusivamente para los usuarios que pertenecen al grupo local de **Administradores**.

* **Definición:** Son *shares* que mapean las unidades de disco y directorios clave del sistema operativo, permitiendo la gestión remota de archivos y configuraciones.
* **Ejemplos Relevantes:**
    * **`C$`**, **`D$`**, etc.: Mapean las unidades de disco raíz del servidor.
    * **`ADMIN$`**: Mapea el directorio raíz del sistema operativo (`C:\Windows`).
    * **`IPC$`**: Recurso especial para la comunicación entre procesos (Interprocess Communication).
* **Acceso:** El acceso se realiza mediante la ruta UNC que incluye el nombre del recurso administrativo, usando el nombre del servidor o su dirección IP. Ejemplo: `\\NombreServidor\ADMIN$`.
