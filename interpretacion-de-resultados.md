# Interpretación de resultados

## PostprocessSV

Está en pruebas, se utilizan 3 programas para llamar SV (Gridss, Svaba y Manta) y no siempre las variantes reales son recogidas por más de uno (True).



## PostprocessCNV

Para seleccionar las CNVs que se incluirán en el informe, se prestará atención a las siguientes columnas:

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Se ignorarán los valores de pvalor, aún cuando este sea de 1.&#x20;

Las columnas nMaj1\_A y nMin1\_A hacen referencia al número de copias de cada alelo y la columna frac1\_A al % de la muestra al que afecta .&#x20;

Las siguientes columnas (nMaj2\_A, nMin2\_A, frac2\_A) hacen referencia a los mismos parámetros pero, en este caso, subclonales.

De cara al informe, también es interesante comprobar el número de genes a los que afecta la CNV y si alguno está implicado en cáncer.



## VEP-annotation

Pendiente de la inclusión de oncoKB.

En la tabla 1 del informe se recogerán las variantes recogidas en la oncoKB y en la tabla 2 aquellas que estén presentes y sean relevantes en genes asociados a cáncer.

En el anexo, se incluirán otras variantes (tier 3) que cumplan X filtros (por determinar).







