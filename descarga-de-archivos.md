---
description: Descargar fastq a partir de enlace
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Descarga de archivos

### Descargar un solo archivo

Descargar en la carpeta `/isabl/isabl_demo/assets/staging/fastqs` los fastq contenidos en el enlace que nos llega de la empresa de secuenciación (#Macrogen).&#x20;

Copiaremos el enlace y lo pegaremos con el comando `wget` tal que así:

<pre data-overflow="wrap"><code><strong>/isabl/isabl_demo/assets/staging/fastqs$ wget https://data.macrogen.com/~macro4/hWGS/202311/HN00206785/2357_01_R1.fastq.gz
</strong></code></pre>

Una vez hagamos esto, el contenido del enlace se descarga en el directorio seleccionado.&#x20;



### Descargar varios archivos consecutivamente

Para descargar varios archivos (cada uno con su propio enlace web) de forma consecutiva y no tener que estar pendiente de cuándo hay que descargar el siguiente, podemos hacer lo siguiente:

1. Desde la terminal de linux creamos un archivo con el comando nano:
   * &#x20;`nano enlaces.txt`
2. Se abrirá un editor de texto donde introduciremos un enlace de descarga por línea. Una vez esté listo, cerramos el documento con **Ctrl + X** y nos preguntaré si deseamos guardar en la parte inferior (Y/N).
3. Usaremos un script en la terminal para leer cada enlace desde el archivo que hemos creado, usando el comando wget para descargar los archivos. Primero se crea un archivo  como antes:
   * `nano descargar_archivos.sh`
   * Cuando se abra el editor, pegaremos el siguiente script:
     *   `#!/bin/bash`

         `#Leer cada enlace del archivo enlaces.txt y descargar los archivos secuencialmente`

         <mark style="color:blue;">`while`</mark>` ``IFS=`` `<mark style="color:orange;">`read`</mark>` ``-r enlace || [[ -n`` `<mark style="color:green;">`"`</mark><mark style="color:red;">`$enlace`</mark><mark style="color:green;">`"`</mark>` ``]];`` `<mark style="color:blue;">`do`</mark> \
         &#x20;  `nombre_archivo=$(`<mark style="color:orange;">`basename`</mark> <mark style="color:green;">`"`</mark><mark style="color:red;">`$enlace`</mark><mark style="color:green;">`"`</mark>`) # Obtener el nombre del archivo desde el enlace` \
         &#x20;  <mark style="color:orange;">`echo`</mark>` `<mark style="color:green;">`"Descargando`</mark> <mark style="color:red;">`$nombre_archivo`</mark><mark style="color:green;">`..."`</mark> \
         &#x20;  `wget`` `<mark style="color:green;">`"`</mark><mark style="color:red;">`$enlace`</mark><mark style="color:green;">`"`</mark>` ``-O`` `<mark style="color:green;">`"`</mark><mark style="color:red;">`$nombre_archivo`</mark><mark style="color:green;">`"`</mark> \
         &#x20;  <mark style="color:orange;">`echo`</mark>` `<mark style="color:green;">`"`</mark><mark style="color:red;">`$nombre_archivo`</mark> <mark style="color:green;">`descargado correctamente."`</mark> \
         <mark style="color:blue;">`done`</mark>` ``< enlaces.txt`
     *

         <figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>
4. Guardaremos el archivo al salir del editor y ejecutamos el siguiente comando para otorgarle permisos de ejecución:&#x20;
   1. `chmod +x descargar_archivos.sh`
5. Ejecutamos el script:
   1. `./descargar_archivos.sh`
