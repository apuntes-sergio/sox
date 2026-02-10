---
title: Bucles for y Procesamiento de Archivos
description: Automatizar tareas repetitivas con bucles for
---

## 🎯 Objetivos de la Sesión

Al finalizar esta sesión serás capaz de:
- Usar bucles `for` para repetir acciones
- Procesar múltiples archivos automáticamente
- Recorrer listas de elementos
- Usar rangos numéricos en bucles
- Combinar bucles con condicionales
- Crear scripts que trabajen con muchos archivos a la vez

---

## 🔁 ¿Qué son los Bucles?

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

---

## 📋 Bucle for Básico

### Sintaxis

```bash
for variable in lista
do
    # Comandos a repetir
done
```

### Ejemplo: Lista de nombres

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

---

## 🔢 Bucle for con Rangos Numéricos

### Rango simple

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

### Rango con incremento

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

**Sintaxis del rango:**
- `{inicio..fin}` → de inicio a fin de 1 en 1
- `{inicio..fin..incremento}` → con incremento personalizado

---

## 📁 Procesar Archivos con Bucles ⭐ (MUY IMPORTANTE)

Esta es una de las funcionalidades más útiles en administración de sistemas.

### Listar archivos por tipo

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

### Copiar múltiples archivos

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

### Renombrar múltiples archivos

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

---

## 🔄 Combinar Bucles con Condicionales

### Ejemplo: Procesar solo archivos que existen

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

### Ejemplo: Copiar solo archivos no vacíos

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

---

## 🛠️ Ejemplos Prácticos Completos

### Ejemplo 1: Crear múltiples archivos

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

### Ejemplo 2: Mostrar información de archivos

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

### Ejemplo 3: Backup selectivo

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

---

## 💻 Ejercicios Prácticos

### Ejercicio 1: Crear Múltiples Directorios (EN CLASE - GUIADO)

**Objetivo:** Crear un script que cree directorios de manera automática.

**Instrucciones:**
1. Crear archivo `crear_carpetas.sh`
2. Crear 5 carpetas llamadas `carpeta1`, `carpeta2`, ..., `carpeta5`
3. Mostrar mensaje por cada carpeta creada

**Solución:**
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

### Ejercicio 2: Backup de Archivos de Texto (EN CLASE - GUIADO)

**Objetivo:** Crear un script que copie todos los archivos .txt a una carpeta de backup.

**Instrucciones:**
1. Crear archivo `backup_txt.sh`
2. Crear carpeta `backup_txt` si no existe
3. Copiar todos los .txt a esa carpeta
4. Contar cuántos archivos se copiaron

**Solución:**
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

---

### Ejercicio 3: Renombrador Masivo (PARA ENTREGAR)

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

# TODO: Verificar que se pasó un parámetro
# TODO: Capturar el prefijo
# TODO: Inicializar contador
# TODO: Bucle for por cada .txt
#   - Crear nuevo nombre con prefijo
#   - Renombrar archivo
#   - Incrementar contador
# TODO: Mostrar resumen
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

---

## 🎯 Ejercicio Avanzado (Opcional)

### Organizador de Archivos por Extensión

Crear un script que organice archivos en carpetas según su extensión:

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

---

## 📝 Resumen de Comandos

### Bucle for básico
```bash
for variable in lista
do
    comandos
done
```

### Rangos numéricos
```bash
for i in {1..10}; do          # Del 1 al 10
for i in {0..20..2}; do       # Del 0 al 20 de 2 en 2
for i in {10..1}; do          # Del 10 al 1 (descendente)
```

### Procesar archivos
```bash
for archivo in *.txt; do      # Todos los .txt
for archivo in *.sh; do       # Todos los .sh
for archivo in *; do          # Todos los archivos
```

### Manipulación de nombres
```bash
${archivo%.txt}               # Quita .txt del final
${archivo#prefijo_}           # Quita prefijo_ del inicio
```

### Incrementar contador
```bash
contador=0
contador=$((contador + 1))
```

---

## 🏠 Tarea para Casa

### Tarea 1: Generador de Estructura de Proyecto

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

**Pistas:**
```bash
proyecto=$1
mkdir -p "$proyecto"/{src,docs,tests,config}

for carpeta in src docs tests config
do
    echo "Carpeta para $carpeta" > "$proyecto/$carpeta/README.txt"
done
```

### Tarea 2: Informe de Archivos

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

---

## ✅ Checklist de la Sesión

Antes de terminar, verifica que puedes:

- [ ] Crear un bucle `for` básico
- [ ] Usar rangos numéricos `{1..10}`
- [ ] Procesar archivos con `*.txt`
- [ ] Combinar bucles con condicionales
- [ ] Usar contadores dentro de bucles
- [ ] Renombrar archivos con bucles
- [ ] Copiar múltiples archivos automáticamente
- [ ] Manipular nombres de archivos `${archivo%.txt}`

---

## 💡 Consejos y Buenas Prácticas

### 1. Siempre verificar que los archivos existen

```bash
for archivo in *.txt
do
    if [ -f "$archivo" ]; then
        # Procesar archivo
    fi
done
```

### 2. Usar comillas en nombres de archivos

```bash
# ✅ Bien (funciona con espacios en nombres)
cp "$archivo" backup/

# ❌ Mal (falla con espacios)
cp $archivo backup/
```

### 3. Inicializar contadores

```bash
contador=0

for item in lista
do
    # hacer algo
    contador=$((contador + 1))
done

echo "Total: $contador"
```

### 4. Mostrar progreso

```bash
for i in {1..100}
do
    echo "Procesando archivo $i de 100..."
    # hacer algo
done
```

### 5. Combinar con parámetros

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

---

## 🔧 Solución de Problemas

### Problema: "No such file or directory"
**Causa:** No hay archivos del tipo especificado
```bash
# Solución: Verificar antes
for archivo in *.txt
do
    if [ -f "$archivo" ]; then
        # El archivo existe, procesarlo
    fi
done
```

### Problema: Espacios en nombres de archivos
**Causa:** Falta de comillas
```bash
# ❌ Mal
for archivo in *.txt; do
    mv $archivo backup/
done

# ✅ Bien
for archivo in *.txt; do
    mv "$archivo" backup/
done
```

### Problema: El bucle se ejecuta una vez aunque no hay archivos
**Causa:** El patrón `*.txt` se toma literal si no hay coincidencias
```bash
# Solución
for archivo in *.txt
do
    # Verificar que no es el patrón literal
    if [ -f "$archivo" ]; then
        # Procesar
    fi
done
```

---

## 🎯 Próxima Sesión

En la próxima sesión aprenderemos:
- Uso profesional de `tar` para crear backups
- Comprimir y descomprimir archivos
- Excluir archivos innecesarios del backup
- Crear backups con fecha en el nombre
- Verificar integridad de backups

**Combinaremos lo aprendido hasta ahora para crear scripts de backup reales.**

---

## 🚀 Reto Extra (Para los Más Rápidos)

Crear un script que:
1. Busque todos los archivos `.sh` en el directorio actual
2. Por cada uno, verifique si tiene permisos de ejecución
3. Si NO los tiene, añadirlos automáticamente
4. Contar cuántos scripts se modificaron

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