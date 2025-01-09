---
layout: single
title: "Magic Numbers"
date: 2025-01-09
classes: wide
header:
  teaser: /assets/images/htb-writeup-smasher/smasher.png
categories:
  - Pentesting
tags:
  - Pentesting
---
### Índice y Estructura Principal

- [Autor](#autor)
- [Que son los magic  numbers](#que-son-los-magic-numbers)
- [Manipulacion de  los  magic numbers](#manipulacion-de--los-magic--numbers)

## Autor
**IRVING ST (Comandre-ex)**

### ¿Que son los magic numbers?.

Los ```magic numbers``` son una secuencia de los primeros 1 a 8 bytes de un archivo. Estos bytes sirven como una "firma" única que permite identificar el formato real del archivo, independientemente de la extensión que este tenga. Existe una creencia común de que el tipo de archivo se define únicamente por su extensión (como `.jpg` o `.pdf`), pero esto no es cierto. La extensión de un archivo no siempre es confiable, ya que puede ser manipulada o no reflejar el contenido real del archivo.

Para verificar el tipo real de un archivo, podemos utilizar herramientas como el comando `file` o `stat` en sistemas Linux, que examinan los ```magic numbers```  de un archivo para determinar su tipo. En Windows, también es posible usar un editor hexadecimal para inspeccionar los primeros bytes del archivo. Estos primeros bytes contienen la secuencia de ```magic numbers```, que son comparados con una base de datos interna que contiene las firmas conocidas de diferentes tipos de archivos.

Al manipular estos ```magic numbers```, es posible ocultar o cambiar el tipo real de un archivo. Por ejemplo, podríamos modificar los primeros bytes de un archivo de imagen `.jpg` para que parezca un archivo `.pdf` o viceversa, lo que podría permitir ocultar el archivo o engañar a las herramientas de análisis de archivos.

Este comportamiento subraya la importancia de no depender únicamente de la extensión del archivo para determinar su tipo. Los ```magic numbers``` ofrecen una forma mucho más confiable de identificar los archivos y también permiten modificar su comportamiento a nivel de bytes, lo cual es útil tanto para la seguridad como para la ocultación de archivos. La manipulación de estos números es una técnica utilizada en diversos contextos, desde pruebas de penetración hasta análisis forense y actividades maliciosas.

* Lista de  magic numbers. 

| Formato de Archivo | Magic Number (Hexadecimal) | Descripción                           |
|--------------------|---------------------------|---------------------------------------|
| **JPEG**           | `FF D8 FF`                | Imagen JPEG                          |
| **PNG**            | `89 50 4E 47 0D 0A 1A 0A` | Imagen PNG                           |
| **GIF**            | `47 49 46 38`            | Imagen GIF (version 87a o 89a)      |
| **PDF**            | `25 50 44 46`            | Documento PDF                        |
| **ZIP**            | `50 4B 03 04`            | Archivo ZIP                          |
| **RAR**            | `52 61 72 21 1A 07 00`    | Archivo RAR                          |
| **MP3**            | `FF FB`                   | Archivo de audio MP3                 |
| **WAV**            | `52 49 46 46`            | Archivo de audio WAV                 |
| **EXE**            | `4D 5A`                   | Archivo ejecutable de Windows        |
| **HTML**           | `3C 21 44 15`             | Archivo HTML (puede comenzar con `<!DOCTYPE html>`) |
| **TAR**            | `75 73 74 61 72`          | Archivo TAR                          |


###  Manipulacion de  los magic  numbers.

La manipulación de bytes se puede realizar de diversas maneras, dependiendo del entorno y las herramientas disponibles. Una de las formas más comunes de hacerlo en sistemas Linux es utilizando el comando ```dd```. Este comando es una utilidad poderosa que permite copiar y convertir archivos, y también es capaz de modificar los bytes de un archivo en bloque. ```dd``` se utiliza ampliamente en administración de sistemas, forense digital y pruebas de penetración debido a su capacidad para manejar datos a nivel de bytes y hacer manipulaciones precisas de los archivos.

El comando ```dd``` permite especificar detalles como el tamaño del bloque de datos a leer y escribir, lo que lo hace flexible y adecuado para tareas de manipulación de bytes. Por ejemplo, se puede usar para modificar solo una parte específica de un archivo, como cambiar los ```magic numbers``` (los primeros bytes de un archivo) para alterar su formato o hacer que se vea como otro tipo de archivo.


* Manipulacion de  magic  numbers  con  ```dd```.

```shell
 $ echo -ne '<magic number>' | dd of=<archivo de entrada> bs=<tamaño de bloque>  count=<cantidad de bloques>
```

* Scripting  de manipulacion  de magic numbers.


```shell
 #!/usr/bin/env bash
 # Colores
 greenColour="\e[0;32m\033[1m"
 endColour="\033[0m\e[0m"
 redColour="\e[0;31m\033[1m"
 blueColour="\e[0;34m\033[1m"
 yellowColour="\e[0;33m\033[1m"
 purpleColour="\e[0;35m\033[1m"
 turquoiseColour="\e[0;36m\033[1m"
 grayColour="\e[0;37m\033[1m"
 
 # Magic numbers para distintos tipos de archivos
 declare -A magic_numbers_list=(
    ["JPEG"]="FFD8FF"
    ["PNG"]="89504E470D0A1A0A"
    ["GIF"]="47494638"
    ["PDF"]="25504446"
    ["ZIP"]="504B0304"
    ["RAR"]="526172211A0700"
    ["MP3"]="FFFB"
    ["WAV"]="52494646"
    ["EXE"]="4D5A"
    ["HTML"]="3C214415"
    ["TAR"]="7573746172"
 )

 trap ctrl_c INT
 tput civis

 function ctrl_c() {
    printf "%bExiting...%b\n" "${grayColour}" "${endColour}"
    tput cnorm
    exit 0
 }

 function ModificacionMagicNumbers() {
     if [[ -n "${magic_numbers_list[$file_type]}" ]]; then
         hex_string="${magic_numbers_list[$file_type]}"
         formatted_hex=""
         for (( i=0; i<${#hex_string}; i+=2 )); do
             formatted_hex+="\x${hex_string:i:2}"
         done

         echo -ne "$formatted_hex" | dd of="$name_file" bs=1 count=$((${#hex_string} / 2)) conv=notrunc 2>/dev/null &1>2
         echo -ne "${greenColour}Magic number cambiado para $file_type en el archivo $name_file${endColour}"
     else
         echo -ne "${redColour}Tipo de archivo no soportado: $file_type${endColour}"
         helpPanel
     fi
 } 

 function helpPanel() {
     printf "%bAuthor%b %b:%b %bIRVING SA (Comandre-ex)%b\n" "${grayColour}" "${endColour}" "${yellowColour}" "${endColour}" "${grayColour}"  "${endColour}"
     printf "\t%b-t%b ) %bTipo de archivo (JPEG, PNG, GIF, PDF, ZIP, RAR, MP3, WAV, EXE, HTML, TAR)%b\n" "${purpleColour}" "${endColour}" "${grayColour}" "${endColour}"
     printf "\t%b-n%b ) %bNombre del archivo%b\n" "${purpleColour}" "${endColour}" "${grayColour}" "${endColour}"
     printf "\t%b-h%b ) %bPanel de ayuda%b\n" "${purpleColour}" "${endColour}" "${grayColour}" "${endColour}"
     printf "\t%bUso%b: %b./ChangueMagicNumbers.sh -t <Tipo de archivo> -n <Nombre de archivo>%b\n" "${grayColour}" "${endColour}" "${purpleColour}" "${endColour}"
 }

 declare -i parameter_counter=0
 while getopts ":t:n:h" arg; do
     case $arg in
         t) file_type=$OPTARG; let parameter_counter+=1;;
         n) name_file=$OPTARG; let parameter_counter+=1;;
         h) helpPanel;;
     esac
 done

 if [[ $parameter_counter -ne 2 ]]; then
     helpPanel
 else
     ModificacionMagicNumbers
 fi 
```