---
title: Bucles for y Procesamiento de Archivos
description: Automatizar tareas repetitivas con bucles for
---


Los bucles permiten **repetir acciones** sin escribir el mismo código muchas veces.

**Sin bucle (repetitivo y tedioso):**
```bash
echo "Procesando archivo1.txt"
echo "Procesando archivo2.txt"
echo "Procesando archivo3.txt"
echo "Procesando archivo4.txt"
# ... y así hasta 100 archivos
```

**Con bucle (eficiente):**
```bash
for i in {1..100}
do
    echo "Procesando archivo$i.txt"
done
```


## Bucle `for` Básico

Sintaxis

```bash
for variable in lista
do
    # Comandos a repetir
done
```

!!!example "Ejemplo: Lista de nombres"

    ```bash
    #!/bin/bash

    for nombre in Sergio Ana María Juan
    do
        echo "Hola $nombre"
    done
    ```

    **Salida:**
    ```
    Hola Sergio
    Hola Ana
    Hola María
    Hola Juan
    ```

    **Explicación:**

    - `for nombre in ...` → por cada elemento de la lista
    - `nombre` toma el valor de cada elemento (Sergio, luego Ana, etc.)
    - `do ... done` → comandos que se repiten


## Bucle for con Rangos Numéricos

Al hacer un bucle con un rango numérico, establecemos el primer elementos de la lista y el último, y en cada iteración del bucle, la variable definida se incrementa en 1 o también es posible indicar el incremento que queremos hacer:

**Sintaxis del rango:**

- `{inicio..fin}` → de inicio a fin de 1 en 1
- `{inicio..fin..incremento}` → con incremento personalizado

Veamos un ejemplo de un incremento sencillo de uno en uno

!!!example "Rango simple"

    ```bash
    #!/bin/bash

    for numero in {1..5}
    do
        echo "Número: $numero"
    done
    ```

    **Salida:**
    ```
    Número: 1
    Número: 2
    Número: 3
    Número: 4
    Número: 5
    ```

Y ahora de un incremento personalizado, en este caso, comenzamos por el 0 e incrementamos de 2 en 2 hasta el 10, o sea, los números pares entre 0 y 10:

!!!example "Rango con incremento"

    ```bash
    #!/bin/bash

    # De 2 en 2
    for numero in {0..10..2}
    do
        echo "Número: $numero"
    done
    ```

    **Salida:**
    ```
    Número: 0
    Número: 2
    Número: 4
    Número: 6
    Número: 8
    Número: 10
    ```


## Procesar Archivos con Bucles ⭐ 

Esta es una de las funcionalidades más útiles en administración de sistemas.

!!!example "Listar archivos por tipo"

    ```bash
    #!/bin/bash

    echo "Archivos .txt encontrados:"

    for archivo in *.txt
    do
        echo "- $archivo"
    done
    ```

    **Si tienes:** `documento1.txt`, `notas.txt`, `informe.txt`

    **Salida:**
    ```
    Archivos .txt encontrados:
    - documento1.txt
    - notas.txt
    - informe.txt
    ```

!!!example "Copiar múltiples archivos"

    ```bash
    #!/bin/bash

    # Crear directorio de backup si no existe
    if [ ! -d "backup" ]; then
        mkdir backup
    fi

    # Copiar todos los .txt
    for archivo in *.txt
    do
        cp "$archivo" backup/
        echo "Copiado: $archivo"
    done

    echo "Backup completado"
    ```

!!! example "Renombrar múltiples archivos"

    ```bash
    #!/bin/bash

    # Cambiar extensión .txt a .bak
    for archivo in *.txt
    do
        nuevo="${archivo%.txt}.bak"
        mv "$archivo" "$nuevo"
        echo "$archivo → $nuevo"
    done
    ```

    **Explicación de `${archivo%.txt}`:**

    - Elimina `.txt` del final del nombre
    - Ejemplo: `documento.txt` → `documento`
    - Luego añadimos `.bak` → `documento.bak`


## Combinar Bucles con Condicionales

!!! example "Ejemplo: Procesar solo archivos que existen"

    ```bash
    #!/bin/bash

    for archivo in *.txt
    do
        if [ -f "$archivo" ]; then
            echo "Procesando: $archivo"
            # Hacer algo con el archivo
        fi
    done
    ```

!!! example "Ejemplo: Copiar solo archivos no vacíos"

    ```bash
    #!/bin/bash

    mkdir -p backup

    for archivo in *.txt
    do
        if [ -s "$archivo" ]; then
            cp "$archivo" backup/
            echo "✓ Copiado: $archivo (no vacío)"
        else
            echo "✗ Omitido: $archivo (vacío)"
        fi
    done
    ```

    **Nota:** `-s` verifica que el archivo NO está vacío


## Ejemplos Prácticos Completos

!!! example "Ejemplo 1: Crear múltiples archivos"

    ```bash
    #!/bin/bash

    echo "Creando archivos de prueba..."

    for i in {1..10}
    do
        touch "archivo$i.txt"
        echo "Creado: archivo$i.txt"
    done

    echo "✓ 10 archivos creados"
    ```

!!! example "Ejemplo 2: Mostrar información de archivos"

    ```bash
    #!/bin/bash

    echo "=========================================="
    echo "INFORMACIÓN DE ARCHIVOS .SH"
    echo "=========================================="

    for script in *.sh
    do
        if [ -f "$script" ]; then
            tamano=$(du -h "$script" | cut -f1)
            
            echo "Archivo: $script"
            echo "  Tamaño: $tamano"
            
            if [ -x "$script" ]; then
                echo "  Ejecutable: SÍ"
            else
                echo "  Ejecutable: NO"
            fi
            
            echo "---"
        fi
    done
    ```

!!! example "Ejemplo 3: Backup selectivo"

    ```bash
    #!/bin/bash
    # Hacer backup solo de archivos modificados hoy

    fecha_hoy=$(date +%Y-%m-%d)
    mkdir -p "backup_$fecha_hoy"

    contador=0

    for archivo in *
    do
        if [ -f "$archivo" ]; then
            # Verificar si fue modificado hoy
            fecha_mod=$(date -r "$archivo" +%Y-%m-%d)
            
            if [ "$fecha_mod" = "$fecha_hoy" ]; then
                cp "$archivo" "backup_$fecha_hoy/"
                echo "✓ Respaldado: $archivo"
                contador=$((contador + 1))
            fi
        fi
    done

    echo "=========================================="
    echo "Total de archivos respaldados: $contador"
    echo "Ubicación: backup_$fecha_hoy/"
    echo "=========================================="
    ```


## Ejercicios Prácticos


!!! question "Ejercicio 1: Crear Múltiples Directorios"

    **Objetivo:** Crear un script que cree directorios de manera automática.

    **Instrucciones:**

    1. Crear archivo `crear_carpetas.sh`
    2. Crear 5 carpetas llamadas `carpeta1`, `carpeta2`, ..., `carpeta5`
    3. Mostrar mensaje por cada carpeta creada

    ??? success "Solución: Intenta resolver sin mirar solución"
        ```bash
        #!/bin/bash

        echo "Creando carpetas..."

        for i in {1..5}
        do
            mkdir "carpeta$i"
            echo "✓ Creada: carpeta$i"
        done

        echo "=========================================="
        echo "5 carpetas creadas exitosamente"
        ```

    **Probar:**
    ```bash
    chmod +x crear_carpetas.sh
    ./crear_carpetas.sh
    ls -d carpeta*
    ```

---

!!! question "Ejercicio 2: Backup de Archivos de Texto"

    **Objetivo:** Crear un script que copie todos los archivos .txt a una carpeta de backup.

    **Instrucciones:**

    1. Crear archivo `backup_txt.sh`
    2. Crear carpeta `backup_txt` si no existe
    3. Copiar todos los .txt a esa carpeta
    4. Contar cuántos archivos se copiaron

    ??? success "Solución: Intenta resolver sin mirar solución"
        ```bash
        #!/bin/bash

        # Crear directorio de backup
        if [ ! -d "backup_txt" ]; then
            mkdir backup_txt
            echo "✓ Directorio backup_txt creado"
        fi

        # Contador
        contador=0

        # Copiar archivos
        echo "Iniciando backup..."

        for archivo in *.txt
        do
            # Verificar que existe (por si no hay .txt)
            if [ -f "$archivo" ]; then
                cp "$archivo" backup_txt/
                echo "✓ Copiado: $archivo"
                contador=$((contador + 1))
            fi
        done

        # Resumen
        echo "=========================================="
        if [ $contador -eq 0 ]; then
            echo "No se encontraron archivos .txt"
        else
            echo "Total de archivos copiados: $contador"
            echo "Ubicación: backup_txt/"
        fi
        echo "=========================================="
        ```

    **Probar:**
    ```bash
    # Crear algunos archivos de prueba
    touch archivo1.txt archivo2.txt archivo3.txt

    # Ejecutar script
    chmod +x backup_txt.sh
    ./backup_txt.sh

    # Verificar
    ls backup_txt/
    ```


!!! question "Ejercicio 3: Renombrador Masivo"

    **Objetivo:** Crear un script que añada prefijo a múltiples archivos.

    **Requisitos:**

    1. El script debe llamarse `renombrar.sh`
    2. Debe recibir como parámetro el prefijo a añadir
    3. Debe renombrar todos los archivos .txt añadiendo el prefijo
    4. Ejemplo: Si ejecutas `./renombrar.sh old_`, el archivo `archivo.txt` se renombra a `old_archivo.txt`
    5. Debe contar cuántos archivos se renombraron

    **Estructura esperada:**
    ```bash
    #!/bin/bash
    # Script: renombrar.sh
    # Uso: ./renombrar.sh PREFIJO

    # Verificar que se pasó un parámetro
    # Capturar el prefijo
    # Inicializar contador
    # Bucle for por cada .txt
    #   - Crear nuevo nombre con prefijo
    #   - Renombrar archivo
    #   - Incrementar contador
    # Mostrar resumen
    ```

    **Ejemplo de ejecución:**
    ```bash
    # Crear archivos de prueba
    touch doc1.txt doc2.txt doc3.txt

    # Ejecutar script
    ./renombrar.sh backup_
    ```

    **Salida esperada:**
    ```
    Renombrando archivos...
    ✓ doc1.txt → backup_doc1.txt
    ✓ doc2.txt → backup_doc2.txt
    ✓ doc3.txt → backup_doc3.txt
    ==========================================
    Total de archivos renombrados: 3
    ```

    **Pistas:**

    - Usa `$1` para capturar el prefijo
    - Usa `mv` para renombrar: `mv "$archivo" "$prefijo$archivo"`
    - Usa un contador: `contador=$((contador + 1))`


!!! question "Ejercicio Avanzado (Opcional). Organizador de Archivos por Extensión"

    Crear un script que organice archivos en carpetas según su extensión:

        ??? success "Solución: Intenta resolver sin mirar solución"

        ```bash
        #!/bin/bash
        # organizador.sh

        echo "Organizando archivos..."

        # Crear carpetas para cada tipo
        mkdir -p imagenes documentos scripts otros

        # Procesar cada archivo
        for archivo in *
        do
            # Solo procesar archivos (no directorios)
            if [ -f "$archivo" ]; then
                
                # Obtener extensión
                if [[ "$archivo" == *.jpg ]] || [[ "$archivo" == *.png ]]; then
                    mv "$archivo" imagenes/
                    echo "✓ $archivo → imagenes/"
                    
                elif [[ "$archivo" == *.txt ]] || [[ "$archivo" == *.pdf ]]; then
                    mv "$archivo" documentos/
                    echo "✓ $archivo → documentos/"
                    
                elif [[ "$archivo" == *.sh ]]; then
                    mv "$archivo" scripts/
                    echo "✓ $archivo → scripts/"
                    
                else
                    mv "$archivo" otros/
                    echo "✓ $archivo → otros/"
                fi
            fi
        done

        echo "=========================================="
        echo "Archivos organizados correctamente"
        ```


## Resumen de Comandos

!!! example "Bucle for básico"
    ```bash
    for variable in lista
    do
        comandos
    done
    ```

!!! example "Rangos numéricos"
    ```bash
    for i in {1..10}; do          # Del 1 al 10
    for i in {0..20..2}; do       # Del 0 al 20 de 2 en 2
    for i in {10..1}; do          # Del 10 al 1 (descendente)
    ```

!!! example "Procesar archivos"
    ```bash
    for archivo in *.txt; do      # Todos los .txt
    for archivo in *.sh; do       # Todos los .sh
    for archivo in *; do          # Todos los archivos
    ```

!!! example "Manipulación de nombres"
    ```bash
    ${archivo%.txt}               # Quita .txt del final
    ${archivo#prefijo_}           # Quita prefijo_ del inicio
    ```

!!! example "Incrementar contador"
    ```bash
    contador=0
    contador=$((contador + 1))
    ```

---

## Tareas extra

!!! question "Tarea 1: Generador de Estructura de Proyecto"

    Crear un script `proyecto.sh` que:

    1. Reciba como parámetro el nombre del proyecto
    2. Cree la siguiente estructura:
    ```
    nombre_proyecto/
    ├── src/
    ├── docs/
    ├── tests/
    ├── config/
    └── README.txt
    ```
    3. Dentro de cada carpeta, cree un archivo `README.txt` con el texto: "Carpeta para [nombre de la carpeta]"

    ??? tip "Pistas:"
        ```bash
        proyecto=$1
        mkdir -p "$proyecto"/{src,docs,tests,config}

        for carpeta in src docs tests config
        do
            echo "Carpeta para $carpeta" > "$proyecto/$carpeta/README.txt"
        done
        ```

!!! question "Tarea 2: Informe de Archivos"

    Crear un script `informe.sh` que:

    1. Cuente cuántos archivos hay de cada tipo (.txt, .sh, .pdf)
    2. Muestre el total de cada tipo
    3. Muestre el tamaño total de todos los archivos

    **Ejemplo de salida:**
    ```
    ========================================
    INFORME DE ARCHIVOS
    ========================================
    Archivos .txt: 5
    Archivos .sh: 3
    Archivos .pdf: 2
    ----------------------------------------
    Total de archivos: 10
    Tamaño total: 2.3M
    ========================================
    ```

!!! question "Reto Extra "

    Crear un script que:

    1. Busque todos los archivos `.sh` en el directorio actual
    2. Por cada uno, verifique si tiene permisos de ejecución
    3. Si NO los tiene, añadirlos automáticamente
    4. Contar cuántos scripts se modificaron

    ??? success "Intenta antes de mirar la solución"

        ```bash
        #!/bin/bash

        modificados=0

        for script in *.sh
        do
            if [ -f "$script" ]; then
                if [ ! -x "$script" ]; then
                    chmod +x "$script"
                    echo "✓ Permisos añadidos a: $script"
                    modificados=$((modificados + 1))
                else
                    echo "○ $script ya es ejecutable"
                fi
            fi
        done

        echo "=========================================="
        echo "Scripts modificados: $modificados"
        ```

## Consejos y Buenas Prácticas

!!! tip "1. Siempre verificar que los archivos existen"

    ```bash
    for archivo in *.txt
    do
        if [ -f "$archivo" ]; then
            # Procesar archivo
        fi
    done
    ```

!!! tip "2. Usar comillas en nombres de archivos"

    ```bash
    # ✅ Bien (funciona con espacios en nombres)
    cp "$archivo" backup/

    # ❌ Mal (falla con espacios)
    cp $archivo backup/
    ```

!!! tip "3. Inicializar contadores"

    ```bash
    contador=0

    for item in lista
    do
        # hacer algo
        contador=$((contador + 1))
    done

    echo "Total: $contador"
    ```

!!! tip "4. Mostrar progreso"

    ```bash
    for i in {1..100}
    do
        echo "Procesando archivo $i de 100..."
        # hacer algo
    done
    ```

!!! tip "5. Combinar con parámetros"

    ```bash
    #!/bin/bash
    # Procesar archivos con extensión pasada como parámetro

    extension=$1

    for archivo in *.$extension
    do
        if [ -f "$archivo" ]; then
            echo "Procesando: $archivo"
        fi
    done
    ```



