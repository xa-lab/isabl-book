# Interpretación de resultados

## Comprobar Quality Control (Qc data)

Consultar el archivo "Quality Control Report"  y comprobar, al menos, que:

* Se han alineado más del <mark style="background-color:orange;">95%</mark> de las secuencias.
* El FastQC tiene buena calidad completa o mayoritariamente.



## PostprocessSV

Está en pruebas, se utilizan 3 programas para llamar SV (Gridss, Svaba y Manta) y no siempre las variantes reales son recogidas por más de uno (True).

Se recomienda comprobar manualmente todas aquellas variantes que vayan a salir reflejadas en el informe final.





## PostprocessCNV

Para seleccionar las CNVs que se incluirán en el informe, se prestará atención a las siguientes columnas:

<figure><img src=".gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

Se ignorarán los valores de pvalor, aún cuando este sea de 1.&#x20;

Las columnas nMaj1\_A y nMin1\_A hacen referencia al número de copias de cada alelo y la columna frac1\_A al % de la muestra al que afecta . &#x20;

Las siguientes columnas (nMaj2\_A, nMin2\_A, frac2\_A) hacen referencia a los mismos parámetros pero, en este caso, subclonales.

De cara al informe, también es interesante comprobar el número de genes a los que afecta la CNV y si alguno está implicado en cáncer.



## VEP-annotation

Se lleva a cabo una revisión en dos pasos:

* Si el programa ha fallado y aparece el mensaje de FAILED, se vuelve a lanzar con el comando _vep-annotation-1.1.0\_start_ que lanza la pipeline de anotación, corre OncoKB, el VEP, y el snapshot. Para ver qué opciones se tienen con dicho comando se puede usar el comando --help (apps-grch38 vep-annotation-1.1.0\_start --help). Normalmente, se utilizará el siguiente comando con la opción --force: apps-grch38 vep-annotation-1.1.0\_start -p Muestra\_tumor Muestra\_normal --force
  * Ejemplo: apps-grch38 vep-annotation-1.1.0\_start -p DEM\_H000022\_T01\_01\_WG01 DEM\_H000022\_N01\_01\_WG01 --force
  * Para continuar, se copia la última línea que se ha escrito y al final se añade --commit
* Si todo ha ido bien, el programa se queda como IN\_PROGRESS en Isabl, al igual que battenberg, y en este punto se pueden revisar las mutaciones manualmente. Para ello, se abre el archivo Snapshots (como muestra la imagen):

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Se abre una ventana donde se muestra el archivo bam con las variantes y sus lecturas (como la imagen inferior) tanto en tumor como en normal:

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

En la parte inferior aparecerán unos botones para aprobar o rechazar las variantes detectadas. Se deben revisar manualmente al menos el 70% de las mismas (sino fallará el programa). Una vez se haya llevado a cabo toda la revisión, se hará click en "Save session".

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Una vez se hayan guardado los cambios, se utilizará el comando _vep-annotation-1.1.0\_finalise,_ que cerrará el proceso de revisión, de la siguiente manera: isabl apps-grch38 vep-annotation-1.1.0\_finalise -p Muestra\_tumor Muestra\_normal

Para continuar, se copia la última línea que se ha escrito y al final se añade --commit

En este punto, ya se pueden ver en Isabl los archivos con todas las variantes que han pasado los filtros predefinidos en los archivos VEP: filtered y curated.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Si no hubiese ninguna variante que revisar, la revisión se tendría que cerrar desde la consola con el siguiente comando:&#x20;

`isabl apps-grch38 vep-annotation-1.1.0_finalise -p muestra/experimento_tumoral muestra/experimento_normal`&#x20;

y añadir, tras comprobar que está bien el comando, `--commit`.

Ejemplo: `isabl apps-grch38 vep-annotation-1.1.0_finalise -p DEM_H000041_T01_01_WG01 DEM_H000041_N01_01_WG01 --commit`





En la tabla 1 del informe se recogerán las variantes recogidas en la oncoKB y en la tabla 2 aquellas que estén presentes y sean relevantes en genes asociados a cáncer.

En el anexo, se incluirán otras variantes (tier 3) que cumplan X filtros (por determinar).







