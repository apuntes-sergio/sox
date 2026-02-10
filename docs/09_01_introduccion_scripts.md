---
title: Introducción y scripts básicos
description: Introducción al scripting en Bash - Variables, lectura de usuario y parámetros
---

## 📖 ¿Qué es un Script?

Un **script** es un archivo de texto que contiene comandos que se ejecutan automáticamente, uno tras otro. Es como una "receta" que le das al ordenador.

**¿Para qué sirve?**
- Automatizar tareas repetitivas
- Hacer backups automáticos
- Crear múltiples usuarios de una vez
- Generar informes del sistema
- Cualquier cosa que hagas manualmente, puedes automatizarla

---

## 🚀 Tu Primer Script

Crear el archivo

```bash
nano hola.sh
```

Escribir el contenido

!!! example "Mi primer Script"

    ```bash
    #!/bin/bash
    # Mi primer script

    echo "¡Hola! Este es mi primer script"
    echo "Hoy es: $(date)"
    echo "Soy el usuario: $USER"
    ```

Dar permisos de ejecución

```bash
chmod +x hola.sh
```

Ejecutar el script

```bash
./hola.sh
```

**Salida esperada:**
```
¡Hola! Este es mi primer script
Hoy es: jue 13 feb 2026 10:30:45 UTC
Soy el usuario: sergio
```

Explicación línea por línea

- `#!/bin/bash` → **Shebang**: indica que es un script de Bash
- `# Mi primer script` → **Comentario**: se ignora, sirve para documentar
- `echo "texto"` → Muestra texto en pantalla
- `$(date)` → Ejecuta el comando `date` y pone el resultado
- `$USER` → Variable del sistema con tu nombre de usuario

---

## 📦 Variables

Las variables son como "cajas" donde guardamos información para usarla después.

Crear y usar variables

!!! Example "Scripts con variables"
    ```bash
    #!/bin/bash

    # Crear variables
    nombre="Sergio"
    edad=20
    ciudad="Valencia"

    # Usar variables (con $)
    echo "Me llamo $nombre"
    echo "Tengo $edad años"
    echo "Vivo en $ciudad"
    ```

**Salida:**
```
Me llamo Sergio
Tengo 20 años
Vivo en Valencia
```

!!! note "⚠️ Reglas importantes"

    1. **NO poner espacios** alrededor del `=`
        - ✅ Correcto: `nombre="Sergio"`
        - ❌ Incorrecto: `nombre = "Sergio"`

    2. Para **usar** una variable, poner `$` delante
        - `echo $nombre`

    3. Los nombres distinguen mayúsculas
        - `nombre` es diferente de `NOMBRE`

    4. Sin espacios en los valores (o usar comillas)
        - ✅ `nombre="Sergio Rey"`
        - ❌ `nombre=Sergio Rey`

---

## 🎤 Pedir Información al Usuario (`read`)

El comando `read` permite que el script pida información al usuario.

!!! example "Ejemplo básico"

    ```bash
    #!/bin/bash

    echo "¿Cómo te llamas?"
    read nombre

    echo "¿Cuántos años tienes?"
    read edad

    echo "Hola $nombre, tienes $edad años"
    ```

El comando `echo` muestra por pantalla, mientras que el comando `read` queda a la espera de una entrada, de forma que la variable que tiene el `read` tendrá esa información.

Podemos hacer el mismo ejemplo de otra manera, y nos ahorramos el `echo`

!!!example "Usando `-p` (más compacto)"

    ```bash
    #!/bin/bash

    read -p "¿Cómo te llamas? " nombre
    read -p "¿Cuántos años tienes? " edad

    echo "Hola $nombre, tienes $edad años"
    ```

Ejecución:**
```
¿Cómo te llamas? Sergio
¿Cuántos años tienes? 20
Hola Sergio, tienes 20 años
```

---

## 📋 Parámetros del Script

Los parámetros son valores que se pasan al script al ejecutarlo.

### Variables especiales de parámetros

- `$0` → Nombre del script
- `$1` → Primer parámetro
- `$2` → Segundo parámetro
- `$3` → Tercer parámetro
- `$#` → Número total de parámetros
- `$@` → Todos los parámetros

!!! example "Ejemplo básico"

    ```bash
    #!/bin/bash
    # Script: saludo.sh

    echo "Nombre del script: $0"
    echo "Primer parámetro: $1"
    echo "Segundo parámetro: $2"
    ```

Ejecutar:
```bash
./saludo.sh Sergio Valencia
```

Salida:
```
Nombre del script: ./saludo.sh
Primer parámetro: Sergio
Segundo parámetro: Valencia
```

!!!example "Ejemplo práctico 2: Script de saludo"

    ```bash
    #!/bin/bash
    # Script: saludar.sh
    # Uso: ./saludar.sh NOMBRE EDAD

    nombre=$1
    edad=$2

    echo "Hola $nombre, tienes $edad años"
    ```

Ejecutar:
```bash
./saludar.sh María 22
```

Salida:
```
Hola María, tienes 22 años
```

### Verificar número de parámetros

Podemos conocer cuantos parámetros nos han pasado y más datos:

- `$0`: Indica el nombre del script
- `$@`: Variable que contiene todos los parámetros en formato lista, separados por espacios en blanco
- `$#`: Variable que contiene el número de parámetros

!!! example "Ejemplo demostración"
    ```bash
    #!/bin/bash
    # Script: info.sh

    echo "Nombre del script: $0"
    echo "Número de parámetros: $#"
    echo "Todos los parámetros: $@"
    ```

Ejecutar:
```bash
./info.sh uno dos tres
```

Salida:
```
Nombre del script: ./info.sh
Número de parámetros: 3
Todos los parámetros: uno dos tres
```

Veamos un ejemplo para crear archivos

!!!example "Ejemplo completo: Creador de archivos"

    ```bash
    #!/bin/bash
    # Script: crear_archivo.sh
    # Uso: ./crear_archivo.sh NOMBRE_ARCHIVO

    archivo=$1

    touch "$archivo"
    echo "Archivo creado: $archivo"
    echo "Fecha de creación: $(date)" > "$archivo"
    ```

Ejecutar:
```bash
./crear_archivo.sh mi_archivo.txt
cat mi_archivo.txt
```

Salida:
```
Archivo creado: mi_archivo.txt
Fecha de creación: jue 13 feb 2026 10:35:12 UTC
```

---

## 🔢 Operaciones Matemáticas Básicas

Para hacer cálculos, usamos `$(( ))`

!!!example "Operaciones básicas"

    ```bash
    #!/bin/bash

    num1=10
    num2=5

    suma=$((num1 + num2))
    resta=$((num1 - num2))
    multiplicacion=$((num1 * num2))
    division=$((num1 / num2))

    echo "$num1 + $num2 = $suma"
    echo "$num1 - $num2 = $resta"
    echo "$num1 × $num2 = $multiplicacion"
    echo "$num1 ÷ $num2 = $division"
    ```

Salida:
```
10 + 5 = 15
10 - 5 = 5
10 × 5 = 50
10 ÷ 5 = 2
```

!!! example "Ejemplo de operaciones matemáticas con parámetros"

    ```bash
    #!/bin/bash
    # Script: calculadora.sh
    # Uso: ./calculadora.sh NUM1 NUM2

    num1=$1
    num2=$2

    suma=$((num1 + num2))

    echo "$num1 + $num2 = $suma"
    ```

Ejecutar:
```bash
./calculadora.sh 8 3
```

Salida:
```
8 + 3 = 11
```

---

## 💻 Ejercicios Prácticos

### Ejercicio 1: Presentación Personal

**Objetivo:** Crear un script que pida información y la muestre.

**Instrucciones:**

1. Crear archivo `presentacion.sh`
2. Pedir nombre, edad y ciudad con `read`
3. Mostrar un mensaje con toda la información

??? example "Inténtalo tu antes de mirar la solución"
    ```bash
    #!/bin/bash

    read -p "¿Cómo te llamas? " nombre
    read -p "¿Cuántos años tienes? " edad
    read -p "¿De qué ciudad eres? " ciudad

    echo "=========================================="
    echo "Hola $nombre"
    echo "Tienes $edad años y eres de $ciudad"
    echo "=========================================="
    ```

Ejecutar y probar:
```bash
chmod +x presentacion.sh
./presentacion.sh
```

---

### Ejercicio 2: Script con Parámetros

**Objetivo:** Crear un script que use parámetros en lugar de `read`.

**Instrucciones:**

1. Crear archivo `info_usuario.sh`
2. Recibir nombre y ciudad como parámetros
3. Mostrar la información

??? example "Inténtalo tu antes de mirar la solución"
    ```bash
    #!/bin/bash
    # Script: info_usuario.sh
    # Uso: ./info_usuario.sh NOMBRE CIUDAD

    nombre=$1
    ciudad=$2

    echo "Usuario: $nombre"
    echo "Ciudad: $ciudad"
    echo "Directorio actual: $(pwd)"
    echo "Fecha: $(date +%d/%m/%Y)"
    ```

**Ejecutar:**
```bash
chmod +x info_usuario.sh
./info_usuario.sh Sergio Valencia
```

---

### Ejercicio 3: Calculadora Simple

**Objetivo:** Crear un script que sume 4 números pasados como parámetros.

**Requisitos:**

1. El script debe llamarse `sumar.sh`
2. Debe recibir dos números como parámetros: `./sumar.sh 5 8 20 1`
3. Debe mostrar el resultado: `5 + 8 + 20 + 1 = 34`


Estructura esperada:
```bash
#!/bin/bash
# Script: sumar.sh
# Uso: ./sumar.sh NUM1 NUM2 NUM3 NUM4

# TODO: Capturar parámetros en variables
# TODO: Calcular la suma
# TODO: Mostrar el resultado
```

---

## 📝 Resumen de Comandos

Crear y ejecutar scripts
```bash
nano script.sh          # Crear/editar script
chmod +x script.sh      # Dar permisos de ejecución
./script.sh             # Ejecutar script
```

Variables
```bash
variable="valor"        # Crear variable
echo $variable          # Usar variable
```

Lectura de usuario
```bash
read variable                      # Pedir input
read -p "Pregunta: " variable      # Con mensaje
```

Parámetros

| Parámetro | Significado |
| --- | --- |
| `$0` | # Nombre del script |
| `$1` | # Primer parámetro |
| `$2` | # Segundo parámetro |
| `$#` | # Número de parámetros |
| `$@` | # Todos los parámetros |


Operaciones
```bash
suma=$((num1 + num2))
resta=$((num1 - num2))
mult=$((num1 * num2))
div=$((num1 / num2))
```

Comandos útiles en scripts
```bash
$(date)                 # Fecha y hora actual
$(date +%Y%m%d)         # Fecha formato YYYYMMDD
$(pwd)                  # Directorio actual
$USER                   # Usuario actual
```

