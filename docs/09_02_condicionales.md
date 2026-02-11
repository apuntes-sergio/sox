---
title: Condicionales y Verificación de Archivos
description: Tomar decisiones en scripts con if/else y comprobar existencia de archivos
---


Los condicionales permiten que el script **tome decisiones** y ejecute diferentes comandos según las condiciones.

**Ejemplo del mundo real:**
```
SI tengo más de 18 años
    ENTONCES puedo votar
SI NO
    ENTONCES no puedo votar
```

En scripting es igual: el script puede verificar condiciones y actuar en consecuencia.

---

## Estructura Básica del `if`

Sintaxis

```bash
if [ condición ]; then
    # Comandos si la condición es VERDADERA
fi
```

**Elementos importantes:**

- Espacios alrededor de los corchetes: `[ condición ]`
- Punto y coma antes de `then`: `]; then`
- Cerrar con `fi` (if al revés)

!!!example "Ejemplo: Mayor de edad"

    ```bash
    #!/bin/bash

    read -p "¿Cuántos años tienes? " edad

    if [ $edad -ge 18 ]; then
        echo "Eres mayor de edad"
    fi
    ```

    **Explicación:**

    - `$edad -ge 18` → si edad es mayor o igual a 18
    - `-ge` significa "greater or equal" (mayor o igual)


## Operadores de Comparación Numérica

Para comparar números, usamos estos operadores:

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `-eq` | igual (equal) | `[ $edad -eq 18 ]` |
| `-ne` | distinto (not equal) | `[ $edad -ne 18 ]` |
| `-gt` | mayor que (greater than) | `[ $edad -gt 18 ]` |
| `-lt` | menor que (less than) | `[ $edad -lt 18 ]` |
| `-ge` | mayor o igual | `[ $edad -ge 18 ]` |
| `-le` | menor o igual | `[ $edad -le 18 ]` |

!!!example "Ejemplo completo"

    ```bash
    #!/bin/bash

    read -p "Introduce un número: " num

    if [ $num -eq 10 ]; then
        echo "El número es 10"
    fi

    if [ $num -gt 10 ]; then
        echo "El número es mayor que 10"
    fi

    if [ $num -lt 10 ]; then
        echo "El número es menor que 10"
    fi
    ```


## if/else - Si no...

Cuando queremos hacer algo diferente si la condición NO se cumple:

```bash
if [ condición ]; then
    # Si la condición es verdadera
else
    # Si la condición es falsa
fi
```

!!!example "Ejemplo: Mayor o menor de edad"

    ```bash
    #!/bin/bash

    read -p "¿Cuántos años tienes? " edad

    if [ $edad -ge 18 ]; then
        echo "Eres mayor de edad"
    else
        echo "Eres menor de edad"
    fi
    ```


## if/elif/else - Múltiples Condiciones

Para comprobar varias condiciones:

```bash
if [ condición1 ]; then
    # Si se cumple condición1
elif [ condición2 ]; then
    # Si se cumple condición2
elif [ condición3 ]; then
    # Si se cumple condición3
else
    # Si no se cumple ninguna
fi
```

!!!example "Ejemplo: Clasificador de notas"

    ```bash
    #!/bin/bash

    read -p "Introduce tu nota (0-10): " nota

    if [ $nota -ge 9 ]; then
        echo "Sobresaliente"
    elif [ $nota -ge 7 ]; then
        echo "Notable"
    elif [ $nota -ge 5 ]; then
        echo "Aprobado"
    else
        echo "Suspenso"
    fi
    ```

    **Ejecución:**
    ```
    Introduce tu nota (0-10): 8
    Notable
    ```


## Comparar Textos

Para comparar textos usamos `=` y `!=`

Sintaxis

```bash
if [ "$texto" = "valor" ]; then
    # Si son iguales
fi

if [ "$texto" != "valor" ]; then
    # Si son diferentes
fi
```

!!!tip "⚠️ IMPORTANTE:"
    Siempre poner las variables entre comillas: `"$variable"`

!!!example "Ejemplo: Verificar respuesta"

    ```bash
    #!/bin/bash

    read -p "¿Quieres continuar? (si/no): " respuesta

    if [ "$respuesta" = "si" ]; then
        echo "Continuando..."
    else
        echo "Operación cancelada"
    fi
    ```

!!!example "Ejemplo: Verificar color favorito"

    ```bash
    #!/bin/bash

    read -p "¿Cuál es tu color favorito? " color

    if [ "$color" = "rojo" ]; then
        echo "El rojo es un color energético"
    elif [ "$color" = "azul" ]; then
        echo "El azul es relajante"
    elif [ "$color" = "verde" ]; then
        echo "El verde es el color de la naturaleza"
    else
        echo "Buen color: $color"
    fi
    ```


## Verificar Archivos y Directorios (⭐)

Esta es una de las funcionalidades más útiles en scripts de administración.

Operadores de Archivos

| Operador | Significado |
|----------|-------------|
| `-f archivo` | Existe y es un archivo regular |
| `-d directorio` | Existe y es un directorio |
| `-e ruta` | Existe (archivo o directorio) |
| `-r archivo` | Es legible |
| `-w archivo` | Es escribible |
| `-x archivo` | Es ejecutable |
| `-s archivo` | No está vacío |

!!!example "Ejemplo: Verificar si existe un archivo"

    ```bash
    #!/bin/bash

    read -p "Introduce nombre de archivo: " archivo

    if [ -f "$archivo" ]; then
        echo "El archivo existe"
    else
        echo "El archivo NO existe"
    fi
    ```

!!!example "Ejemplo: Verificar si es archivo o directorio"

    ```bash
    #!/bin/bash

    read -p "Introduce una ruta: " ruta

    if [ -f "$ruta" ]; then
        echo "$ruta es un ARCHIVO"
    elif [ -d "$ruta" ]; then
        echo "$ruta es un DIRECTORIO"
    else
        echo "$ruta NO EXISTE"
    fi
    ```

!!!example "Ejemplo: Comprobar permisos ⭐"

    ```bash
    #!/bin/bash

    archivo=$1

    if [ ! -e "$archivo" ]; then
        echo "El archivo NO existe"
        exit 1
    fi

    echo "Información de: $archivo"

    if [ -f "$archivo" ]; then
        echo "Tipo: Archivo"
    elif [ -d "$archivo" ]; then
        echo "Tipo: Directorio"
    fi

    if [ -r "$archivo" ]; then
        echo "Permiso de lectura: SÍ"
    else
        echo "Permiso de lectura: NO"
    fi

    if [ -w "$archivo" ]; then
        echo "Permiso de escritura: SÍ"
    else
        echo "Permiso de escritura: NO"
    fi

    if [ -x "$archivo" ]; then
        echo "Permiso de ejecución: SÍ"
    else
        echo "Permiso de ejecución: NO"
    fi
    ```

!!!note "Nota:"
    `!` invierte la condición (`-e` = existe, `! -e` = NO existe)


## Operadores Lógicos

### AND (&&) 

Ambas condiciones deben cumplirse

```bash
if [ condición1 ] && [ condición2 ]; then
    # Solo si AMBAS son verdaderas
fi
```

!!!example "Ejemplo:"
    ```bash
    #!/bin/bash

    read -p "Edad: " edad
    read -p "¿Tienes carnet? (si/no): " carnet

    if [ $edad -ge 18 ] && [ "$carnet" = "si" ]; then
        echo "Puedes conducir"
    else
        echo "No puedes conducir"
    fi
    ```

### OR (||) 

Al menos una debe cumplirse

```bash
if [ condición1 ] || [ condición2 ]; then
    # Si CUALQUIERA de las dos es verdadera
fi
```

!!!example "Ejemplo:"
    ```bash
    #!/bin/bash

    read -p "¿Es fin de semana? (si/no): " finde
    read -p "¿Estás de vacaciones? (si/no): " vacaciones

    if [ "$finde" = "si" ] || [ "$vacaciones" = "si" ]; then
        echo "¡A descansar!"
    else
        echo "Toca trabajar"
    fi
    ```


## Ejercicios Prácticos

!!! example "Ejercicio 1: Clasificador de Notas"

    **Objetivo:** Crear un script que clasifique notas.

    **Instrucciones:**
    1. Crear archivo `notas.sh`
    2. Pedir una nota (0-10)
    3. Mostrar la calificación según la nota

    ??? example "Inténtalo tu antes de mirar la solución"
        ```bash
        #!/bin/bash

        read -p "Introduce tu nota (0-10): " nota

        if [ $nota -ge 9 ]; then
            echo "Calificación: Sobresaliente"
        elif [ $nota -ge 7 ]; then
            echo "Calificación: Notable"
        elif [ $nota -ge 5 ]; then
            echo "Calificación: Aprobado"
        else
            echo "Calificación: Suspenso"
        fi
        ```

        **Probar:**
        ```bash
        chmod +x notas.sh
        ./notas.sh
        ```


!!! example "Ejercicio 2: Verificador de Archivos"

    **Objetivo:** Crear un script que verifique si existe un archivo y muestre información.

    **Instrucciones:**
    1. Crear archivo `verificar.sh`
    2. Recibir nombre de archivo como parámetro
    3. Verificar si existe
    4. Si existe, mostrar si es archivo o directorio
    5. Mostrar permisos

    ??? example "Inténtalo tu antes de mirar la solución"
        ```bash
        #!/bin/bash
        # Uso: ./verificar.sh ARCHIVO

        archivo=$1

        # Verificar que se pasó un parámetro
        if [ $# -eq 0 ]; then
            echo "Uso: $0 ARCHIVO"
            exit 1
        fi

        # Verificar si existe
        if [ ! -e "$archivo" ]; then
            echo "ERROR: $archivo no existe"
            exit 1
        fi

        echo "===================================="
        echo "Información de: $archivo"
        echo "===================================="

        # Tipo
        if [ -f "$archivo" ]; then
            echo "Tipo: Archivo regular"
        elif [ -d "$archivo" ]; then
            echo "Tipo: Directorio"
        fi

        # Permisos
        echo -n "Lectura: "
        if [ -r "$archivo" ]; then
            echo "SÍ"
        else
            echo "NO"
        fi

        echo -n "Escritura: "
        if [ -w "$archivo" ]; then
            echo "SÍ"
        else
            echo "NO"
        fi

        echo -n "Ejecución: "
        if [ -x "$archivo" ]; then
            echo "SÍ"
        else
            echo "NO"
        fi

        echo "===================================="
        ```

        **Probar:**
        ```bash
        chmod +x verificar.sh
        ./verificar.sh /etc/passwd
        ./verificar.sh /home
        ./verificar.sh archivo_inexistente
        ```

---

!!! example "Ejercicio 3: Creador de Directorio Seguro"

    **Objetivo:** Crear un script que cree un directorio solo si no existe.

    **Requisitos:**

    1. El script debe llamarse `crear_dir.sh`
    2. Debe recibir el nombre del directorio como parámetro
    3. Debe verificar si el directorio ya existe
    4. Si NO existe, crearlo y mostrar mensaje de éxito
    5. Si YA existe, mostrar mensaje de error y NO crear nada

    **Estructura esperada:**
    ```bash
    #!/bin/bash
    # Script: crear_dir.sh
    # Uso: ./crear_dir.sh NOMBRE_DIRECTORIO

    # TODO: Verificar que se pasó un parámetro
    # TODO: Capturar el parámetro en una variable
    # TODO: Verificar si el directorio existe
    # TODO: Si NO existe, crearlo
    # TODO: Si SÍ existe, mostrar error
    ```

    **Ejemplo de ejecución:**
    ```bash
    ./crear_dir.sh mi_carpeta
    ```

    **Salida esperada (si no existe):**
    ```
    ✓ Directorio 'mi_carpeta' creado exitosamente
    ```

    **Salida esperada (si ya existe):**
    ```
    ✗ ERROR: El directorio 'mi_carpeta' ya existe
    ```

    **Pistas:**
    - Usa `$#` para verificar número de parámetros
    - Usa `-d` para verificar si existe el directorio
    - Usa `mkdir` para crear el directorio
    - Usa `!` para invertir la condición

---

## 📝 Resumen de Comandos

### Estructura if
```bash
if [ condición ]; then
    comandos
fi

if [ condición ]; then
    comandos
else
    otros_comandos
fi

if [ condición1 ]; then
    comandos1
elif [ condición2 ]; then
    comandos2
else
    comandos3
fi
```

### Comparación numérica
```bash
[ $num -eq 5 ]    # Igual
[ $num -ne 5 ]    # Distinto
[ $num -gt 5 ]    # Mayor que
[ $num -lt 5 ]    # Menor que
[ $num -ge 5 ]    # Mayor o igual
[ $num -le 5 ]    # Menor o igual
```

### Comparación de texto
```bash
[ "$texto" = "hola" ]     # Igual
[ "$texto" != "hola" ]    # Distinto
```

### Verificación de archivos
```bash
[ -f archivo ]      # Es un archivo
[ -d directorio ]   # Es un directorio
[ -e ruta ]        # Existe
[ -r archivo ]     # Es legible
[ -w archivo ]     # Es escribible
[ -x archivo ]     # Es ejecutable
[ ! -e archivo ]   # NO existe
```

### Operadores lógicos
```bash
[ cond1 ] && [ cond2 ]    # AND - ambas verdaderas
[ cond1 ] || [ cond2 ]    # OR - alguna verdadera
```

---

## 🏠 Tarea para Casa

### Tarea 1: Verificador de Espacio en Disco

Crear un script llamado `verificar_espacio.sh` que:
1. Compruebe el espacio libre en el disco
2. Si hay menos del 80% usado, mostrar "OK: Espacio suficiente"
3. Si hay más del 80% usado, mostrar "ADVERTENCIA: Poco espacio"

**Pistas:**
```bash
# Obtener porcentaje de uso del disco raíz
uso=$(df -h / | tail -n 1 | awk '{print $5}' | sed 's/%//')

# Comparar con 80
if [ $uso -gt 80 ]; then
    echo "ADVERTENCIA"
fi
```

### Tarea 2: Script de Backup Condicional

Crear un script llamado `backup_condicional.sh` que:
1. Reciba como parámetro un directorio a respaldar
2. Verifique que el directorio existe
3. Si NO existe, mostrar error y salir
4. Si SÍ existe, crear un backup con tar:
   - Nombre: `backup_YYYYMMDD.tar.gz`
   - Mostrar mensaje de éxito con el tamaño del backup

**Estructura:**
```bash
#!/bin/bash
# Uso: ./backup_condicional.sh DIRECTORIO

directorio=$1

# Verificar parámetro

# Verificar que existe el directorio

# Crear backup
fecha=$(date +%Y%m%d)
tar -czf "backup_$fecha.tar.gz" "$directorio"

# Mostrar mensaje de éxito
```

---

## ✅ Checklist de la Sesión

Antes de terminar, verifica que puedes:

- [ ] Usar `if/then/fi` correctamente
- [ ] Usar `if/else/fi`
- [ ] Usar `if/elif/else/fi`
- [ ] Comparar números con `-eq`, `-gt`, `-lt`, etc.
- [ ] Comparar textos con `=` y `!=`
- [ ] Verificar si existe un archivo con `-f`
- [ ] Verificar si existe un directorio con `-d`
- [ ] Comprobar permisos con `-r`, `-w`, `-x`
- [ ] Usar operadores lógicos `&&` y `||`
- [ ] Invertir condiciones con `!`

---

## 💡 Errores Comunes y Soluciones

### Error: "unary operator expected"
**Causa:** Variable vacía o sin comillas
```bash
# ❌ Mal
if [ $variable = "texto" ]; then

# ✅ Bien
if [ "$variable" = "texto" ]; then
```

### Error: Espacios en los corchetes
```bash
# ❌ Mal
if [$num -eq 5]; then

# ✅ Bien
if [ $num -eq 5 ]; then
```

### Error: Usar = en lugar de -eq para números
```bash
# ❌ Mal
if [ $num = 5 ]; then

# ✅ Bien
if [ $num -eq 5 ]; then
```

### Error: Olvidar el `fi`
```bash
# ❌ Mal
if [ condición ]; then
    echo "algo"

# ✅ Bien
if [ condición ]; then
    echo "algo"
fi
```

---

## 🎯 Próxima Sesión

En la próxima sesión aprenderemos:
- Bucles `for` para repetir acciones
- Procesar múltiples archivos automáticamente
- Crear archivos en masa
- Hacer backups de múltiples directorios

**Prepárate para automatizar tareas repetitivas de forma eficiente.**