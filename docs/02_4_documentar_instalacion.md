---
title: Documentación
description:  Documentación de la instalación
---

# Documentación de la instalación del servidor

Durante el proceso de instalación de un servidor, es fundamental registrar toda la información relevante en un documento técnico. Esta documentación servirá como referencia para futuras intervenciones, auditorías, mantenimiento o restauraciones.

## Información que debe incluirse

- **Fecha y hora de la instalación**  
  Registro cronológico del inicio y finalización del proceso.

- **Versión y número de serie del sistema operativo**  
  Incluye también las licencias de cliente instaladas (CALs).

- **Especificaciones del hardware**  
  Detalles del equipo: procesador, memoria RAM, discos duros, tarjeta gráfica, tarjetas de red, etc.

- **Discos duros y particiones**  
  Nombre de cada volumen, tamaño, sistema de archivos (NTFS, ReFS, etc.), y uso previsto.

- **Identificación del equipo**  
  Nombre del equipo, contraseña del administrador, ubicación física (sala, rack, puesto).

- **Software adicional instalado**  
  Nombre del programa, versión, descripción, utilidad y fecha de instalación.

- **Configuración de red**  
  Dirección IP, máscara de subred, puerta de enlace, servidores DNS, número de roseta en el switch o rack, nombre del dominio o grupo de trabajo.

- **Actualizaciones instaladas**  
  Nombre de la actualización, utilidad y fecha de instalación.

- **Clientes conectados**  
  IP, sistema operativo, nombre del equipo, tipo de dispositivo, ubicación, usuario habitual, observaciones.

- **Otros datos relevantes**  
  Antivirus instalado, cortafuegos, gestor de base de datos, servidores a los que está conectado, servicios activos.

- **Configuraciones adicionales**  
  Detalles sobre la configuración de elementos como el cortafuegos, antivirus, gestor de base de datos, etc.

- **Impresoras conectadas**  
  Nombre, tipo, dirección IP o puerto, ubicación física.

## Consideraciones

- La documentación debe mantenerse actualizada cada vez que se realice un cambio en la configuración del servidor.
- Es recomendable utilizar una plantilla estructurada para facilitar la recogida de datos.
- Puedes consultar un ejemplo de plantilla en la documentación oficial o en recursos como [la propuesta de textcortex.com](https://textcortex.com/es/post/technical-documentation-template).

## Documento ejemplo

A continuación tienes enlace a documento tipo ejemplo para documentar:

> [Descargar documentación del equipo ISCA](files/Documentacion_Equipo_ISCA.docx)
> 
## Recomendación post-instalación

Una vez finalizada la instalación y configuración del servidor, es **muy recomendable crear una imagen del sistema**. Esta copia de seguridad permitirá restaurar el servidor rápidamente en caso de fallo grave, corrupción del sistema o pérdida de datos.

