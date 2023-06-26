# Análisis de muestras

## Creación de un nuevo proyecto

En el menú desplegable de la esquina superior derecha ![](<../.gitbook/assets/image (4).png>), pulsar el icono <img src="../.gitbook/assets/image (13).png" alt="" data-size="line"> para crear un nuevo proyecto.

Aparecerá una ventana emergente donde se escribirá, como mínimo, el título del proyecto y, a mayores, el grupo que lo solicita (_Group_) y las personas implicadas en su análisis.

<figure><img src="../.gitbook/assets/image (10).png" alt="" width="563"><figcaption></figcaption></figure>

## Importación de metadatos (pacientes y muestras)

Todos los datos asociados al paciente y sus muestras se registran en una hoja modelo de Excel.

Para importar los datos, hacer _click_ en el recuadro azul (click here to register new metadata), que abrirá una nueva ventana emergente.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

En la ventana emergente, se puede descargar la plantilla de Excel en la que se incluirán todos los datos de pacientes y muestras del proyecto. Para tenerla lo más actualizada posible, se recomienda descargarla para cada nuevo proyecto, haciendo _click_ en _Get form_.

<figure><img src="../.gitbook/assets/image (6).png" alt="" width="563"><figcaption></figcaption></figure>

La plantilla se descarga con el nombre "SubmissionTemplate\_aaaa-mm-dd" y se recomienda completar el máximo de campos posibles (una fila por muestra), e ineludiblemente todos aquellos donde se incluya el término _(required)_ tanto sobre datos del individuo como de la muestra.&#x20;

Una vez se halla cumplimentado la plantilla, subir el archivo haciendo _click_ en el icono <img src="../.gitbook/assets/image (12).png" alt="" data-size="line"> (o soltar el archivo sobre la pantalla emergente) y pulsar _Commit submission_.



## Importar datos

Para importar los datos, desde la carpeta que contiene los fastq's, se corre la siguiente secuencia: comando de importar (isabl-import-data) + identificador que vamos a usar para asociar fastq y muestra (-id sample.identifier) + --ignore-ownership + filtro por el número del proyecto que nos interesa + directorio en el que se encuentran los archivos fastq

El número de proyecto se obtiene de la columna key:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
^/isabl-import-data -id sample.identifier --ignore-ownership -fi projects.pk N -di /path/to/isabl_demo/assets/staging/fastqs
```
{% endcode %}

Ejemplo:

{% code overflow="wrap" %}
```
^/(isabl-cli) raquel@pinchinX:~/isabl/fastqs$ isabl-import-data -id sample.identifier --ignore-ownership -fi projects.pk 4 -di /isabl/isabl_demo/assets/staging/fastqs
```
{% endcode %}

Tras este comando, el programa comprobará si se puede llevar a cabo y, de ser así, aparecerá en pantalla el siguiente mensaje:

```
/add --commit to proceed.
```

Para ello, se copia la última línea que se ha escrito y al final se añade --commit:

{% code overflow="wrap" %}
```
^/isabl-import-data -id sample.identifier --ignore-ownership -fi projects.pk 4 -di /directorio/subdirectorio --commit
```
{% endcode %}



## Resultados

Una vez se ha comenzado a correr el programa, para ver el estado en el que se encuentran los distintos programas/análisis de la muestra, se abre la pestaña "Analyses" donde aparece el estado (_status_) en el que se encuentran:

* ![](<../.gitbook/assets/image (9).png>):
* ![](<../.gitbook/assets/image (8).png>): el programa se está corriendo.
* ![](<../.gitbook/assets/image (3).png>): el programa está esperando una acción del usuario.
* ![](<../.gitbook/assets/image (15).png>): el programa se ha completado con éxito.
* ![](<../.gitbook/assets/image (14).png>): el programa ha fallado y se tiene que volver a correr.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**Battenberg** se mostrará "in progress" y solicitará una acción para continuar. Antes de avanzar, se comprobará que es creíble el resultado que devuelve el programa, para lo que se deben comprobar, manualmente, las imágenes sobre ploidía de los cromosomas.&#x20;

Se debe comprobar que el BAF y el logR no presentan nada extraño (dentro de las alteraciones de ganancia y pérdida esperadas, el orden de magnitud tiene que tener sentido). Se comparan las magnitudes de las alteraciones vistas en logR del `tumor profile` con las del `non rounded profile` para ver si coinciden.

En las imágenes `non rounded profile` de cada cromosoma se muestran las alteraciones <mark style="color:green;">clonales en verde</mark> (la mayoría) y las <mark style="color:red;">subclonales en rojo</mark> (menos numerosas). Si todas las alteraciones son subclonales, se debe repetir el análisis.

Para verificar que el resultado que nos muestra battenberg de forma automática es correcto, se observará en el segmento con la alteración clonal de mayor tamaño (no subclonal) si realmente es correcto el cálculo automático.

1.  **Resultado original correcto**: si no se tiene que realizar ninguna modificación, se correrá el siguiente comando.

    * tras correr el programa `isabl`
    * correr: isabl + apps-grch38 + battenberg-3.5.2\_finalise + -p muestra (usaremos el código _Experiment System ID_ recogido en el programa) + --subclones-original
    * El programa comprobará el comando y si puede continuar nos indicará que introduzcamos --commit al final de la última línea

    Ejemplo:  `isabl apps-grch38 battenberg-3.5.2_finalise -p DEM_H000017_T01_01_WG01 DEM_H000017_N01_01_WG01 --subclones-original --commit`
2. <mark style="background-color:orange;">**Resultado original incorrecto**</mark>: si se quiere modificar algún parámetro se pueden consultar los comandos pertinentes para según cuál se desee modificar en `isabl apps-grch38` apareciendo los siguientes:

```
battenberg-3.5.2_finalise Finalise a cgpBattenberg run.
battenberg-3.5.2_forcecn Force copy number for a cgpBattenberg run
battenberg-3.5.2_refitcn Refit copy number for a cgpBattenberg run
battenberg-3.5.2_subclones Battenberg pipeline for the discovery of...
```

Nota: Para consultar cómo correr un comando escribir --help tras el comando que queramos, por ejemplo, `isabl apps-grch38 battenberg-3.5.2_finalise --help` y se mostrará cómo ejecutarlo.





