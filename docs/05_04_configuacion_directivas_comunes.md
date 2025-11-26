---
title: Configuración de directivas comunes
description:  Gestión del dominio en Windows Server. Configuración de directivas comunes
---


En esta sección veremos las configuraciones más habituales que se aplican mediante GPO en entornos empresariales. Cada configuración incluirá la ruta dentro del Editor de directivas y ejemplos prácticos.

## Estructura del Editor de directivas de grupo

Antes de comenzar, es importante conocer las dos secciones principales:

### Configuración del equipo (Computer Configuration)

* Se aplica a los **ordenadores**, independientemente del usuario que inicie sesión
* Se procesa durante el **arranque** del sistema
* Afecta a todos los usuarios que utilicen ese equipo

**Subsecciones principales:**

* **Directivas → Configuración de software**: Instalación de aplicaciones
* **Directivas → Configuración de Windows**: Seguridad, scripts, redirección
* **Directivas → Plantillas administrativas**: Configuraciones del sistema operativo
* **Preferencias**: Configuraciones que el usuario puede modificar

### Configuración de usuario (User Configuration)

* Se aplica a los **usuarios**, independientemente del equipo donde inicien sesión
* Se procesa durante el **inicio de sesión**
* Las configuraciones "siguen" al usuario

**Subsecciones principales:**

* **Directivas → Configuración de software**: Instalación de aplicaciones para el usuario
* **Directivas → Configuración de Windows**: Scripts, redirección de carpetas
* **Directivas → Plantillas administrativas**: Configuraciones de escritorio y aplicaciones
* **Preferencias**: Configuraciones personalizables por el usuario

<figure markdown="span" align="center">
    ![](./imgs/directivas/editor_directiva_grupo_2.png){ width="70%" }
    <figcaption>Editor de directivas de grupo.</figcaption>
</figure>

!!! tip "¿Equipo o Usuario?"
    * Si la configuración debe aplicarse **independientemente de quién inicie sesión** → Configuración del equipo
    * Si la configuración debe **seguir al usuario** en cualquier equipo → Configuración de usuario

---


## Configuración de directivas comunes

### Directivas predeterminadas del dominio

Al crear un dominio de Active Directory, se generan automáticamente **dos GPO predeterminadas** con configuraciones de seguridad esenciales. Es importante conocerlas antes de crear nuevas directivas.

#### Default Domain Policy

La **Default Domain Policy** es una GPO vinculada automáticamente a la raíz del dominio y se aplica a todos los usuarios y equipos del dominio (excepto los controladores de dominio).

**Propósito:** Contiene las configuraciones de seguridad básicas a nivel de dominio, principalmente:

* **Directivas de contraseñas**: Longitud mínima, complejidad, vigencia
* **Directivas de bloqueo de cuentas**: Umbral de bloqueos, duración
* **Directivas Kerberos**: Configuración de autenticación del dominio

**¿Qué configuraciones incluye por defecto?**

| Directiva | Valor predeterminado |
|-----------|---------------------|
| Longitud mínima de contraseña | 7 caracteres |
| Vigencia máxima de contraseña | 42 días |
| Vigencia mínima de contraseña | 1 día |
| Requisitos de complejidad | Habilitada |
| Historial de contraseñas | 24 contraseñas |
| Umbral de bloqueo de cuenta | No definido (deshabilitado) |

!!!tip "**Buenas prácticas:**"

    * ✅ **Editar esta GPO** para ajustar políticas de contraseña según requisitos de la organización
    * ✅ **No eliminarla nunca**: Es crítica para el funcionamiento del dominio
    * ⚠️ **No añadir configuraciones no relacionadas con seguridad de cuentas**: Crea GPO específicas para otras configuraciones
    * ✅ **Realizar copias de seguridad** antes de modificarla

<figure markdown="span" align="center">
    ![](./imgs/directivas/default_domain_policy.png){ width="85%" }
    <figcaption>Editor de directivas de grupo.</figcaption>
</figure>

!!! warning "Modificar con cuidado"
    Esta GPO afecta a **todos los usuarios del dominio**. Cambios incorrectos en políticas de contraseña o Kerberos pueden bloquear el acceso a todo el dominio.


#### Default Domain Controllers Policy

La **Default Domain Controllers Policy** es una GPO vinculada automáticamente a la OU **"Domain Controllers"** y se aplica únicamente a los controladores de dominio.

**Propósito:** Contiene configuraciones de seguridad específicas para proteger los controladores de dominio, incluyendo:

* **Asignación de derechos de usuario**: Quién puede iniciar sesión localmente, hacer copias de seguridad, etc.
* **Directivas de auditoría**: Registro de eventos de seguridad en los DC
* **Opciones de seguridad**: Configuraciones especiales para servicios de dominio

**¿Qué configuraciones incluye?**

* Permisos para iniciar sesión en los controladores de dominio
* Derechos de copia de seguridad de archivos y directorios
* Configuración de auditoría de acceso a objetos de AD
* Restricciones de seguridad del protocolo LDAP

**¿Dónde se aplica?**

```
Dominio: empresa.local
└── Domain Controllers (OU)
    ├── DC01 (Controlador de dominio)
    └── DC02 (Controlador de dominio)
    
    ↑ Default Domain Controllers Policy se aplica aquí
```

!!!tip "**Buenas prácticas:**"

    * ✅ **No eliminarla nunca**: Esencial para la seguridad de los DC
    * ✅ **Editar solo para reforzar seguridad**: Añadir más auditoría, restricciones de acceso
    * ⚠️ **No añadir configuraciones de usuario**: Los DC no deben usarse como estaciones de trabajo
    * ✅ **Mantener respaldos actualizados**

<figure markdown="span" align="center">
    ![](./imgs/directivas/default_domain_controller_policy.png){ width="85%" }
    <figcaption>Editor de directivas de grupo.</figcaption>
</figure>

!!! danger "Crítico"
    Los controladores de dominio son el núcleo de Active Directory. Modificaciones incorrectas en esta GPO pueden comprometer la seguridad de todo el dominio o impedir que los DC funcionen correctamente.




#### Comparación entre ambas directivas

| Característica | Default Domain Policy | Default Domain Controllers Policy |
|----------------|----------------------|----------------------------------|
| **Ámbito** | Todo el dominio | Solo controladores de dominio |
| **Ubicación** | Vinculada a la raíz del dominio | Vinculada a OU "Domain Controllers" |
| **Contenido principal** | Políticas de contraseñas y cuentas | Derechos de usuario y auditoría |
| **Se puede eliminar** | ❌ NO (crítica) | ❌ NO (crítica) |
| **Se puede modificar** | ✅ Sí (con cuidado) | ✅ Sí (con cuidado) |
| **Afecta a usuarios** | ✅ Sí | ❌ No (solo equipos DC) |



#### ¿Crear GPO nuevas o modificar las predeterminadas?

**Recomendación general:**

Para la mayoría de las configuraciones, es mejor **crear GPO nuevas** específicas en lugar de modificar las predeterminadas.

**Cuándo modificar las predeterminadas:**

* ✅ **Default Domain Policy**: Para cambiar políticas de contraseñas y bloqueo de cuentas
* ✅ **Default Domain Controllers Policy**: Para reforzar auditoría y seguridad de los DC

**Cuándo crear GPO nuevas:**

* ✅ Configuraciones de escritorio y entorno de usuario
* ✅ Instalación de software
* ✅ Redirección de carpetas
* ✅ Restricciones de aplicaciones
* ✅ Scripts de inicio/cierre
* ✅ Configuraciones específicas de departamentos

!!! tip "Organización recomendada"
    Estructura sugerida de GPO:
    
    * **Default Domain Policy**: Solo políticas de contraseñas/cuentas
    * **SEC-Seguridad-Base**: Auditoría, firewall, antivirus
    * **CONFIG-Escritorio-Corporativo**: Fondo, restricciones UI
    * **SOFT-Aplicaciones-Departamento**: Instalación de software específico
    * **REDIR-Carpetas-Usuarios**: Redirección de perfiles



#### Restaurar directivas predeterminadas

Si accidentalmente se modifican o eliminan las directivas predeterminadas, se pueden restaurar:

**Usando dcgpofix (desde el DC):**

1. Abrir **CMD o PowerShell** como administrador en el controlador de dominio
2. Ejecutar:

```cmd
dcgpofix /ignoreschema /target:domain
```

Esto restaura la **Default Domain Policy**

```cmd
dcgpofix /ignoreschema /target:dc
```

Esto restaura la **Default Domain Controllers Policy**

```cmd
dcgpofix /ignoreschema /target:both
```

Esto restaura **ambas** directivas

!!! danger "Precaución con dcgpofix"
    `dcgpofix` sobrescribe completamente las GPO predeterminadas con sus valores de fábrica. **Perderás todas las modificaciones** que hayas hecho. Úsalo solo como último recurso.

**Alternativa: Restaurar desde copia de seguridad**

Si realizaste copias de seguridad previas (recomendado):

1. GPMC → Clic derecho sobre la GPO
2. **"Restaurar desde copia de seguridad..."**
3. Seleccionar la copia adecuada


### Directivas de seguridad personalizadas

Una vez comprendidas las directivas predeterminadas, procedemos a configurar directivas personalizadas según las necesidades de la organización.

Las directivas de seguridad son fundamentales para proteger el entorno de red.

#### 1. Directivas de contraseñas

Controlan los requisitos de las contraseñas de los usuarios del dominio.

**Ubicación:**
```
Configuración del equipo
└── Directivas
    └── Configuración de Windows
        └── Configuración de seguridad
            └── Directivas de cuenta
                └── Directiva de contraseñas
```

**Configuraciones principales:**

| Directiva | Descripción | Valor recomendado |
|-----------|-------------|-------------------|
| **Exigir el historial de contraseñas** | Número de contraseñas anteriores que no se pueden reutilizar | 24 contraseñas |
| **Vigencia máxima de la contraseña** | Días antes de que expire | 60-90 días |
| **Vigencia mínima de la contraseña** | Días antes de poder cambiarla | 1 día |
| **Longitud mínima de la contraseña** | Caracteres mínimos | 12-14 caracteres |
| **Debe cumplir los requisitos de complejidad** | Mayúsculas, minúsculas, números, símbolos | Habilitada |
| **Almacenar contraseñas con cifrado reversible** | Almacenamiento inseguro | Deshabilitada |

**Ejemplo práctico: Política de contraseñas robusta**

1. Editar una GPO vinculada al dominio (afecta a todos los usuarios)
2. Navegar a la ruta de directivas de contraseñas
3. Configurar:
    * Longitud mínima: **12 caracteres**
    * Vigencia máxima: **90 días**
    * Requisitos de complejidad: **Habilitada**
    * Historial: **24 contraseñas**

<figure markdown="span" align="center">
    ![](./imgs/directivas/directiva_contrasenya.png){ width="85%" }
    <figcaption>Panel del Editor mostrando las directivas de contraseñas con los valores configurados en la columna "Configuración de directiva"</figcaption>
</figure>

!!! warning "Solo una GPO de contraseñas"
    Las directivas de contraseñas deben configurarse en **una única GPO vinculada al dominio**. Si hay múltiples GPO con configuraciones diferentes, solo se aplicará una (la de mayor prioridad), lo que puede causar confusión.


#### 2. Directivas de bloqueo de cuentas

Protegen contra ataques de fuerza bruta bloqueando cuentas tras intentos fallidos.

**Ubicación:**
```
Configuración del equipo
└── Directivas
    └── Configuración de Windows
        └── Configuración de seguridad
            └── Directivas de cuenta
                └── Directiva de bloqueo de cuentas
```

**Configuraciones principales:**

| Directiva | Descripción | Valor recomendado |
|-----------|-------------|-------------------|
| **Duración del bloqueo de cuenta** | Minutos que permanece bloqueada | 30 minutos (o hasta que un administrador la desbloquee = 0) |
| **Umbral de bloqueo de cuenta** | Intentos fallidos antes del bloqueo | 5-10 intentos |
| **Restablecer el contador de bloqueo después de** | Minutos para resetear el contador de intentos | 30 minutos |

**Ejemplo práctico:**

1. Configurar umbral: **5 intentos fallidos**
2. Duración: **30 minutos**
3. Restablecer contador: **30 minutos**

Resultado: Tras 5 intentos fallidos, la cuenta se bloquea 30 minutos automáticamente.


#### 3. Directivas de auditoría

Registran eventos de seguridad en el Visor de eventos para detectar actividades sospechosas.

**Ubicación:**
```
Configuración del equipo
└── Directivas
    └── Configuración de Windows
        └── Configuración de seguridad
            └── Directivas locales
                └── Directiva de auditoría
```

**Categorías principales:**

| Evento | Qué audita | Cuándo habilitar |
|--------|------------|------------------|
| **Auditar eventos de inicio de sesión de cuenta** | Intentos de autenticación en el dominio | Siempre (Correcto y Error) |
| **Auditar la administración de cuentas** | Creación, modificación, eliminación de cuentas | Siempre (Correcto y Error) |
| **Auditar el acceso al servicio de directorio** | Cambios en Active Directory | Entornos de alta seguridad |
| **Auditar eventos de inicio de sesión** | Inicios de sesión locales en equipos | Siempre (Correcto y Error) |
| **Auditar el acceso a objetos** | Acceso a archivos y carpetas | Carpetas sensibles |
| **Auditar cambios de directivas** | Modificaciones en GPO | Siempre (Correcto) |

**Configuración recomendada básica:**

1. Eventos de inicio de sesión de cuenta: **Correcto y Error**
2. Administración de cuentas: **Correcto y Error**
3. Cambios de directivas: **Correcto**
4. Inicio de sesión: **Error** (para detectar intentos fallidos)

!!! info "Ver eventos auditados"
    Los eventos se registran en:
    
    * **Visor de eventos** → Registros de Windows → Seguridad
    * Buscar Event ID específicos (ej: 4624 = inicio de sesión correcto, 4625 = inicio de sesión fallido)


### Directivas de escritorio y entorno de usuario

Estas directivas permiten personalizar y restringir el entorno de trabajo de los usuarios.

#### 1. Configuración del escritorio

**Ubicación:**
```
Configuración de usuario
└── Directivas
    └── Plantillas administrativas
        └── Escritorio
```

**Configuraciones comunes:**

| Directiva | Qué hace | Ruta específica |
|-----------|----------|-----------------|
| **Ocultar y deshabilitar todos los elementos del escritorio** | Escritorio vacío sin iconos | Escritorio |
| **Quitar el icono Papelera de reciclaje del escritorio** | Oculta la papelera | Escritorio |
| **Impedir cambios en el fondo de escritorio** | Bloquea cambio de wallpaper | Escritorio\Active Desktop |
| **Fondo de escritorio** | Establece una imagen corporativa | Escritorio\Active Desktop |

**Ejemplo: Establecer fondo corporativo**

1. Navegar a: `Configuración de usuario → Plantillas administrativas → Escritorio → Active Desktop`
2. Editar **"Fondo de escritorio"**
3. **Habilitar** la directiva
4. Introducir ruta UNC: `\\servidor\compartido\fondocorporativo.jpg`
5. Seleccionar estilo: **Rellenar, Ajustar, Expandir, Mosaico, Centrar**
6. Aplicar


!!! tip "Compartir imágenes corporativas"
    Guarda las imágenes en una carpeta compartida en SYSVOL o en un servidor de archivos accesible para todos:
    
    ```
    \\dominio.local\SYSVOL\dominio.local\Recursos\Fondos\
    ```


#### 2. Menú Inicio y barra de tareas

**Ubicación:**
```
Configuración de usuario
└── Directivas
    └── Plantillas administrativas
        └── Menú Inicio y barra de tareas
```

**Restricciones comunes:**

| Directiva | Efecto |
|-----------|--------|
| **Quitar el menú Ejecutar del menú Inicio** | El usuario no puede usar Win+R |
| **Quitar Configuración del menú Inicio** | Oculta acceso a Configuración de Windows |
| **Quitar el icono Panel de control del menú Inicio** | No puede acceder al Panel de control |
| **Impedir cambios en la configuración de la barra de tareas y del menú Inicio** | Bloquea personalización |
| **Quitar la opción Apagar** | Impide apagar el equipo (útil en terminales compartidos) |

**Ejemplo: Bloquear Panel de control**

1. Navegar a la ruta de Menú Inicio
2. Buscar **"Prohibir el acceso al Panel de control y a Configuración de PC"**
3. **Habilitar** la directiva
4. Aplicar

El usuario no podrá acceder al Panel de control por ningún medio.

#### 3. Restricciones del Explorador de Windows

**Ubicación:**
```
Configuración de usuario
└── Directivas
    └── Plantillas administrativas
        └── Componentes de Windows
            └── Explorador de archivos
```

**Configuraciones útiles:**

| Directiva | Descripción |
|-----------|-------------|
| **Ocultar estas unidades especificadas en Mi PC** | Oculta unidades (A:, C:, D:, etc.) |
| **Impedir el acceso a las unidades desde Mi PC** | Bloquea acceso real a las unidades |
| **Quitar la ficha Seguridad** | Los usuarios no ven permisos de archivos |
| **Quitar el menú Herramientas (y acceso a Opciones de carpeta)** | Impide cambiar configuración del explorador |

**Ejemplo: Ocultar unidad C:**

1. Buscar **"Ocultar estas unidades especificadas en Mi PC"**
2. **Habilitar**
3. Seleccionar: **"Restringir solo las unidades C"**
4. Aplicar

!!! warning "Diferencia importante"
    * **Ocultar unidades**: No se ve en el explorador pero se puede acceder escribiendo la ruta
    * **Impedir acceso**: Bloquea acceso real incluso escribiendo la ruta


### Directivas de instalación y restricción de software

#### 1. Restricciones de software (AppLocker / Software Restriction Policies)

Controlan qué aplicaciones pueden ejecutarse en los equipos.

**Ubicación (AppLocker - Windows 7 y superior):**
```
Configuración del equipo
└── Directivas
    └── Configuración de Windows
        └── Configuración de seguridad
            └── Directivas de control de aplicaciones
                └── AppLocker
```

**Tipos de reglas:**

* **Reglas ejecutables**: .exe, .com
* **Reglas de Windows Installer**: .msi, .msp
* **Reglas de scripts**: .ps1, .bat, .cmd, .vbs
* **Reglas de aplicaciones empaquetadas**: Apps de Microsoft Store
* **Reglas de DLL**: Librerías (alto impacto en rendimiento)

**Ejemplo: Bloquear ejecución desde carpeta Descargas**

1. En AppLocker, clic derecho en **"Reglas ejecutables"** → **"Crear nueva regla..."**
2. Siguiente → Seleccionar **"Denegar"**
3. Seleccionar usuarios/grupos afectados
4. Siguiente → Condición: **"Ruta de acceso"**
5. Ruta: `%USERPROFILE%\Downloads\*`
6. Siguiente → Dar nombre → Crear

Los usuarios no podrán ejecutar archivos .exe desde su carpeta de Descargas.

!!! info "Activar servicio AppLocker"
    Para que AppLocker funcione, el servicio "Identidad de aplicación" debe estar iniciado. Se puede configurar mediante GPO:
    
    ```
    Configuración del equipo → Preferencias → Configuración del Panel de control → Servicios
    ```


#### 2. Instalación de software

Permite desplegar aplicaciones automáticamente a usuarios o equipos.

**Ubicación:**
```
Configuración del equipo (o usuario)
└── Directivas
    └── Configuración de software
        └── Instalación de software
```

**Métodos de implementación:**

| Método | Cuándo se instala | Descripción |
|--------|-------------------|-------------|
| **Asignado (equipo)** | Al arrancar el equipo | Instalación automática y obligatoria |
| **Asignado (usuario)** | Al iniciar sesión | Aparece en Programas, se instala al usar |
| **Publicado (solo usuario)** | Nunca automáticamente | Disponible en "Agregar o quitar programas" |

**Ejemplo: Desplegar 7-Zip en todos los equipos**

**Requisitos previos:**

1. Tener el archivo **.msi** de 7-Zip
2. Compartir en ubicación accesible: `\\servidor\software\7zip.msi`
3. Dar permisos de lectura a "Equipos del dominio"

**Pasos:**

1. Editar GPO vinculada a la OU con los equipos
2. Ir a: `Configuración del equipo → Instalación de software`
3. Clic derecho → **"Nuevo"** → **"Paquete..."**
4. Navegar usando ruta UNC (no letra de unidad) hasta el .msi
5. Seleccionar el archivo
6. Elegir **"Asignado"**
7. Aceptar

Al reiniciar, los equipos instalarán 7-Zip automáticamente.

!!! warning "Solo archivos .msi"
    La instalación de software mediante GPO solo admite archivos Windows Installer (.msi). Para ejecutables (.exe), usa scripts de inicio.


### Redirección de carpetas

Redirige carpetas del perfil de usuario a ubicaciones de red centralizadas.

**Ubicación:**
```
Configuración de usuario
└── Directivas
    └── Configuración de Windows
        └── Redirección de carpetas
```

**Carpetas que se pueden redirigir:**

* Documentos
* Escritorio
* Menú Inicio
* Música, Imágenes, Vídeos
* Descargas
* Favoritos

**Ventajas:**

* ✅ **Copias de seguridad centralizadas**: Los datos están en el servidor
* ✅ **Movilidad**: El usuario accede a sus archivos desde cualquier equipo
* ✅ **Libera espacio**: No se almacena localmente en el equipo

**Ejemplo: Redirigir carpeta Documentos**

**Preparación:**

1. Crear carpeta compartida en servidor: `\\servidor\perfiles$\`
2. Compartir con permisos:
   * Usuarios autenticados: Control total
3. Permisos NTFS:
   * SYSTEM: Control total
   * Usuarios del dominio: Modificar

**Configuración:**

1. En GPO, navegar a Redirección de carpetas
2. Clic derecho en **"Documentos"** → **"Propiedades"**
3. Configuración: **"Básica: redirigir las carpetas de todos a la misma ubicación"**
4. Ubicación de carpeta de destino: **"Crear una carpeta para cada usuario en la ruta de acceso raíz"**
5. Ruta de acceso raíz: `\\servidor\perfiles$\%USERNAME%\Documentos`
6. Pestaña **"Configuración"**:
   * ✅ Conceder al usuario derechos exclusivos
   * ✅ Mover el contenido a la nueva ubicación
7. Aceptar

!!! tip "Variable %USERNAME%"
    La variable `%USERNAME%` se sustituye automáticamente por el nombre de cada usuario, creando carpetas individuales.


### Scripts de inicio y cierre de sesión

Permiten ejecutar scripts automáticamente en momentos específicos.

**Ubicación (Scripts de inicio/cierre de equipo):**
```
Configuración del equipo
└── Directivas
    └── Configuración de Windows
        └── Scripts (Inicio o apagado)
```

**Ubicación (Scripts de inicio/cierre de sesión de usuario):**
```
Configuración de usuario
└── Directivas
    └── Configuración de Windows
        └── Scripts (Inicio o cierre de sesión)
```

**Tipos de scripts:**

* **.bat / .cmd**: Scripts de MS-DOS/Windows
* **.ps1**: Scripts de PowerShell
* **.vbs**: Visual Basic Script

**Ejemplo: Mapear unidad de red al iniciar sesión**

**Crear el script:**

1. Crear archivo `mapear_unidad.bat`:
```batch
@echo off
net use Z: \\servidor\compartido /persistent:yes
```

2. Guardar en: `\\dominio.local\SYSVOL\dominio.local\scripts\`

**Configurar GPO:**

1. Editar GPO
2. Ir a: `Configuración de usuario → Scripts → Inicio de sesión`
3. Hacer clic en **"Agregar..."**
4. Examinar y seleccionar `mapear_unidad.bat`
5. Aceptar

Al iniciar sesión, se mapeará automáticamente la unidad Z:.

!!! info "Scripts PowerShell"
    Para ejecutar scripts .ps1, primero debes habilitar la ejecución de PowerShell:
    
    ```
    Configuración del equipo → Plantillas administrativas → Componentes de Windows 
    → Windows PowerShell → Activar la ejecución de scripts
    ```


### Plantillas administrativas adicionales

#### 1. Microsoft Edge / Internet Explorer

**Ubicación:**
```
Configuración de usuario (o equipo)
└── Plantillas administrativas
    └── Componentes de Windows
        └── Microsoft Edge (o Internet Explorer)
```

**Configuraciones útiles:**

* Establecer página principal predeterminada
* Deshabilitar modo InPrivate
* Configurar sitios de confianza
* Bloquear extensiones no autorizadas
* Configurar proxy


#### 2. Windows Update

**Ubicación:**
```
Configuración del equipo
└── Plantillas administrativas
    └── Componentes de Windows
        └── Windows Update
```

**Configuraciones importantes:**

* Configurar actualizaciones automáticas
* Especificar servidor WSUS interno
* Programar día y hora de instalación
* Habilitar actualizaciones de calidad/características

