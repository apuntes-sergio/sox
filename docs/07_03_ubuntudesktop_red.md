---
title: Ubuntu Desktop. Configuración de red con Netplan y NetworkManager
description: Gestión de sistemas Linux. Integración de Ubuntu Desktop y Server en entornos Windows
---

En este apartado vamos a ver cómo configurar la red de nuestro equipo Ubuntu Desktop. Para que todo fucione correctamente:

- El servidor Windows Server debe estar iniciado
- La red en virtual box debes configurarla para comunicarte con el servidor

## Introducción a la gestión de red en Ubuntu Desktop

La configuración de red en Ubuntu Desktop funciona de manera diferente a Ubuntu Server, y es importante entender por qué y cuándo usar cada sistema. Ubuntu Desktop viene configurado con **NetworkManager** por defecto, mientras que Ubuntu Server usa **Netplan** con `systemd-networkd`. Ambos sistemas pueden coexistir, pero es fundamental entender sus diferencias y cuándo usar cada uno.

### NetworkManager vs Netplan: ¿cuál usar?

**NetworkManager** está diseñado para equipos de escritorio donde:

- La configuración de red cambia frecuentemente (WiFi de casa, cable en la oficina, hotspot del móvil, etc.)
- El usuario necesita cambiar configuraciones sin ser administrador
- Se valora la comodidad de una interfaz gráfica
- La máquina se mueve entre diferentes redes

**Netplan** con `systemd-networkd` está diseñado para servidores donde:

- La configuración de red es estática y raramente cambia
- Se prefiere la configuración mediante archivos de texto para poder versionarla y automatizarla
- No hay interfaz gráfica disponible
- Se requiere configuración avanzada de red (bridges, VLANs, bonding, etc.)

### Configuración por defecto en Ubuntu Desktop

Por defecto, Ubuntu Desktop delega toda la gestión de red en NetworkManager. Si miramos la configuración de Netplan:

```bash
cat /etc/netplan/*.yaml
```

Probablemente veremos algo así:

```yaml
network:
  version: 2
  renderer: NetworkManager
```

Esta configuración significa: "Netplan, no configures nada tú. Deja que NetworkManager gestione toda la red".

Esta es la configuración óptima para un equipo Desktop de uso general, y **no deberíamos cambiarla** a menos que tengamos una razón específica.

---

## NetworkManager: configuración gráfica

Para la mayoría de situaciones en un equipo Desktop, NetworkManager es la herramienta adecuada. Vamos a ver cómo configurar los aspectos más importantes desde su interfaz gráfica.

### Acceso a la configuración

Para acceder a la configuración de red:

1. Hacemos clic en el **icono de red** del panel superior (esquina superior derecha)
2. En el menú desplegable, seleccionamos **Configuración** o **Configuración de red**
3. Se abre la aplicación de Configuración del sistema en la sección de Red

Alternativamente, podemos:
- Buscar "Red" o "Network" en Actividades
- Abrir Configuración y buscar la sección Red

### Configuración DHCP automática (por defecto)

Esta es la configuración que viene por defecto y es adecuada para la mayoría de situaciones:

1. En la ventana de configuración de red, vemos nuestra conexión "Conexión cableada" o "Wired"
2. Junto a ella hay un icono de **engranaje** ⚙️ 
3. Hacemos clic en el engranaje para ver los detalles

En la pestaña **IPv4**, veremos:

- **Método IPv4**: Automático (DHCP)
- **Direcciones automáticas**: Activado
- **DNS automático**: Activado
- **Rutas automáticas**: Activado

Con esta configuración, el equipo obtiene automáticamente:
- Dirección IP
- Máscara de red
- Puerta de enlace (gateway)
- Servidores DNS

Esta configuración es perfecta si estamos en una red con servidor DHCP y no necesitamos IP fija.

### Configuración manual (IP estática)

Para equipos que formarán parte de un dominio Active Directory, necesitamos configuración manual para asegurar que la IP no cambie y que usamos el DNS correcto.

#### Escenario: Equipo que se unirá a Active Directory

Configuración necesaria para nuestro entorno ISCASOX:

- **IP fija**: 192.168.100.50 (elegid una libre en vuestra red)
- **Máscara**: 255.255.255.0 (equivalente a /24)
- **Puerta de enlace**: 192.168.100.1 (el Windows Server)
- **DNS primario**: 192.168.100.1 (el Windows Server - CRÍTICO)
- **DNS secundario**: 1.1.1.1 (Cloudflare, como backup)
- **Dominios de búsqueda**: DOMXXX.local

<figure markdown="span" align="center">
  ![](./imgs/ubuntu/desktop/NetworkManager.png){ width="80%" }
  <figcaption>Configuración de red por NetworkManager</figcaption>
</figure>


**Verificación de la configuración**:

Abrimos una terminal y ejecutamos:

```bash
ip addr show
```

Deberíamos ver nuestra interfaz (normalmente `enp0s3` o `ens33`) con la IP que configuramos:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.100.50/24 brd 192.168.100.255 scope global dynamic noprefixroute enp0s3
```

Verificamos el gateway:

```bash
ip route show
```

Deberíamos ver una línea como:

```
default via 192.168.100.1 dev enp0s3 proto static metric 100
```

Verificamos el DNS:

```bash
resolvectl status
```

En la sección de nuestra interfaz (`enp0s3` o similar) deberíamos ver:

```
Link 2 (enp0s3)
    Current DNS Server: 192.168.100.1
           DNS Servers: 192.168.100.1
                        1.1.1.1
            DNS Domain: DOMXXX.local
```

Si todo coincide, la configuración es correcta.

**Probar la conectividad**:

Ping al gateway:

```bash
ping -c 3 192.168.100.1
```

Resolución de nombres del dominio:

```bash
nslookup DOMXXX.local
```

Debería responder con la IP del Windows Server (192.168.100.1).

Ping a Internet para verificar que tenemos salida:

```bash
ping -c 3 google.com
```

Si todos estos comandos funcionan, la red está perfectamente configurada.

---

## Netplan: configuración avanzada

Aunque NetworkManager es suficiente para la mayoría de casos en Desktop, hay situaciones donde necesitamos o preferimos usar Netplan directamente:

- **Configuración reproducible**: podemos versionar archivos YAML y aplicarlos automáticamente
- **Configuraciones complejas**: bridges, VLANs, bonding, múltiples IPs en una interfaz
- **Automatización**: scripting y despliegue automatizado de configuraciones
- **Preferencia personal**: algunos administradores prefieren editar archivos de texto
- **Consistencia**: si gestionamos tanto Servers como Desktops, usar la misma herramienta simplifica

### Cómo funciona Netplan

Netplan es una **capa de abstracción** sobre los sistemas de red de Linux. No gestiona la red directamente, sino que lee archivos de configuración YAML y genera la configuración para el "renderer" (gestor de red) que hayamos elegido.

Los dos renderers principales son:

- **NetworkManager**: para Desktop (soporta WiFi, interfaces gráficas, configuración dinámica)
- **systemd-networkd**: para Server (más ligero, sin GUI, configuración estática)

```
┌─────────────────────┐
│  Archivos Netplan   │  ← Editamos estos (.yaml en /etc/netplan/)
│   (formato YAML)    │
└──────────┬──────────┘
           │
           ↓
      ┌────────┐
      │Netplan │  ← Procesa el YAML y genera configuración
      └────┬───┘
           │
           ↓
  ┌────────────────┐
  │    Renderer    │  ← NetworkManager o systemd-networkd
  │ (gestor real)  │
  └────────────────┘
```

### Ubicación y estructura de archivos

Los archivos de configuración de Netplan están en `/etc/netplan/` y tienen extensión `.yaml`.

```bash
ls /etc/netplan/
```

El nombre del archivo determina el orden de aplicación. Los archivos se procesan en orden numérico/alfabético:

- `00-installer-config.yaml` - configuración del instalador
- `01-network-manager-all.yaml` - configuración para NetworkManager
- `50-cloud-init.yaml` - configuración de cloud-init (si existe)

Si hay múltiples archivos, sus configuraciones se combinan. En caso de conflicto, el último archivo tiene prioridad.

**Buena práctica**: usar nombres descriptivos con números:

```
00-base.yaml           (configuración básica)
10-static-ip.yaml      (configuración estática)
50-advanced.yaml       (configuraciones avanzadas)
```

!!!tip "Permisos de archivos de Netplan"

    Los archivos de configuración de Netplan contienen información sensible (contraseñas WiFi, configuraciones de red). Debemos protegerlos:

    ```bash
    sudo chmod 600 /etc/netplan/*.yaml
    ```

    Esto asegura que solo root puede leer/escribir estos archivos.

### Sintaxis básica de YAML para Netplan

YAML es muy estricto con la sintaxis. Estos son los puntos críticos:

**Indentación con espacios, NUNCA tabuladores**: Cada nivel de indentación son exactamente 2 espacios.

```yaml
# CORRECTO
network:
  version: 2
  
# INCORRECTO (tabuladores)
network:
	version: 2
```

**Los dos puntos (`:`) son obligatorios** después de cada clave:

```yaml
network:
  ethernets:
    enp0s3:
```

**Las listas se indican con guiones (`-`)**:

```yaml
addresses:
  - 192.168.100.50/24
  - 192.168.100.51/24
```

**Las comillas no son obligatorias** para strings simples, pero sí para valores con caracteres especiales:

```yaml
# Sin comillas (preferido para simplicidad)
renderer: networkd

# Con comillas (necesario si hay espacios o caracteres especiales)
comment: "Esta es una configuración de red"
```

**Booleanos**: `true/false` o `yes/no` (ambos válidos):

```yaml
dhcp4: false
dhcp6: no
```


## Ejemplos de configuración de Netplan

A continuación se presentan 3 configuraciones diferentes de Netplan. 

Para editar el fichero de configuración, ejecutamos un comando como:

```bash
sudo nano /etc/netplan/01-static-config.yaml
```

Y para probar o aplicar la configuración:

```bash
# Verificar sintaxis
sudo netplan try

# Aplicar configuración
sudo netplan aply
```

El comando `netplan try` es MUY IMPORTANTE: aplica la configuración temporalmente y revierte automáticamente si no confirmamos en 120 segundos. Esto evita que nos quedemos sin acceso de red por un error de configuración.

Si la red funciona, pulsamos Enter para aceptar. Si algo falla o no respondemos, la configuración se revierte automáticamente.

!!!example "Ejemplo 1: Delegar todo a NetworkManager (configuración por defecto)"

    Esta es la configuración más simple y es la que viene por defecto en Ubuntu Desktop:

    ```yaml
    network:
      version: 2
      renderer: NetworkManager
    ```

    !!!note "Explicación"

        - `network`: raíz de toda configuración de Netplan
        - `version: 2`: versión del formato YAML de Netplan (siempre usamos 2)
        - `renderer: NetworkManager`: delega toda la gestión a NetworkManager

        Con esta configuración, Netplan no hace nada más. NetworkManager gestiona todo mediante su interfaz gráfica.

        **Cuándo usarla**: Siempre que queramos que NetworkManager gestione la red completamente mediante GUI.

!!!example "Ejemplo 2: IP estática básica con systemd-networkd"

    Configuración simple de IP estática usando `systemd-networkd` como renderer:

    ```yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s3:
          dhcp4: false
          dhcp6: false
          addresses:
            - 192.168.100.50/24
          routes:
            - to: default
              via: 192.168.100.1
          nameservers:
            addresses:
              - 192.168.100.1
              - 1.1.1.1
    ```

    !!!note "Explicación línea por línea"

        - `renderer: networkd`: usar systemd-networkd en lugar de NetworkManager
        - `ethernets:`: sección para configuración de interfaces Ethernet cableadas
        - `enp0s3:`: nombre de nuestra interfaz de red (verificar con `ip addr`)
        - `dhcp4: false`: no usar DHCP para IPv4
        - `dhcp6: false`: no usar DHCP para IPv6
        - `addresses:`: lista de direcciones IP a asignar
          - `- 192.168.100.50/24`: IP estática con máscara /24 (255.255.255.0)
        - `routes:`: rutas de red
          - `to: default`: ruta por defecto (equivale a 0.0.0.0/0)
          - `via: 192.168.100.1`: gateway (puerta de enlace)
        - `nameservers:`: configuración de DNS
          - `addresses:`: lista de servidores DNS
            - `192.168.100.1`: DNS primario (nuestro Windows Server)
            - `1.1.1.1`: DNS secundario (Cloudflare)


!!!example "Ejemplo 3: Configuración para equipo en dominio Active Directory"

    Esta es la configuración óptima para un equipo Ubuntu Desktop que se unirá a un dominio Windows:

    ```yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s3:
          dhcp4: false
          dhcp6: false
          addresses:
            - 192.168.100.50/24
          routes:
            - to: default
              via: 192.168.100.1
          nameservers:
            search:
              - DOMSERGIO.local
            addresses:
              - 192.168.100.1
              - 1.1.1.1
    ```

    !!!note "Diferencias clave respecto al ejemplo anterior"

        - `search:`: dominios de búsqueda DNS
          - `- DOMSERGIO.local`: nuestro dominio Active Directory

        Esta línea `search` es CRÍTICA para Active Directory. Permite:

        1. Resolver nombres cortos: escribir `ping servidor` en lugar de `ping servidor.domsergio.local`
        2. Facilitar la unión al dominio
        3. Permitir que las aplicaciones encuentren servicios del dominio

        **El orden de los DNS es importante**:

        ```yaml
        nameservers:
          addresses:
            - 192.168.100.1    # DNS PRIMARIO: el controlador de dominio
            - 1.1.1.1          # DNS SECUNDARIO: backup para Internet
        ```

        Siempre ponemos el controlador de dominio primero porque:

        - Active Directory requiere que los clientes usen su DNS
        - El DNS de AD contiene registros SRV necesarios para localizar servicios del dominio
        - Si pusiera 1.1.1.1 primero, muchas consultas de dominio fallarían


**Verificación específica para dominio**:

```bash
# Verificar que el dominio de búsqueda está configurado
resolvectl status

# Probar resolución del dominio
nslookup DOMSERGIO.local

```

<figure markdown="span" align="center">
  ![](./imgs/ubuntu/desktop/nslookup.png){ width="80%" }
  <figcaption>Verificación de la res</figcaption>
</figure>


### Comandos útiles de Netplan

**Verificar sintaxis sin aplicar**:

```bash
sudo netplan generate
```

Genera la configuración pero no la aplica. Si hay errores de sintaxis, los muestra.

**Aplicar configuración con seguridad (recomendado)**:

```bash
sudo netplan try
```

Aplica temporalmente. Si no confirmamos en 120 segundos, revierte automáticamente. Evita quedarnos sin conexión.

**Aplicar configuración inmediatamente**:

```bash
sudo netplan apply
```

Aplica la configuración sin pedir confirmación. Solo usar cuando estemos seguros.

**Ver la configuración efectiva**:

```bash
sudo netplan get
```

Muestra la configuración final después de combinar todos los archivos YAML.

**Depurar problemas**:

```bash
sudo netplan --debug apply
```

Muestra información detallada sobre qué está haciendo Netplan.



