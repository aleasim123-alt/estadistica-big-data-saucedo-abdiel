# Actividad 4.1 - Modelado no supervisado con Covertype

**Autor:** Abdiel Eliasim Saucedo Sosa\
**Asignatura:** Estadística / Big Data\
**Dataset:** Covertype (`sklearn.datasets.fetch_covtype`)\
**Técnicas:** PCA + MiniBatch K-Means

## Pregunta de análisis

¿Qué estructuras de agrupación pueden identificarse a partir de las
características cartográficas y qué tan confiables y útiles resultan los
grupos obtenidos?

## Datos

El conjunto Covertype contiene 581,012 observaciones y 55 variables en
la carga original. La variable `Cover_Type` se separó antes de cualquier
transformación y se reservó exclusivamente para validación externa.

Para el agrupamiento se utilizaron 54 características:

-   10 variables cuantitativas.
-   44 indicadores binarios de áreas silvestres y tipos de suelo.
-   No se encontraron valores faltantes ni registros duplicados.
-   Memoria inicial aproximada de la matriz de características
    procesada: 239.37 MB.

## Preprocesamiento

Las diez variables cuantitativas se estandarizaron mediante
`StandardScaler`. Las variables binarias se conservaron en escala 0/1
para mantener su interpretación.

`Cover_Type` no participó en el escalamiento, PCA, selección del número
de grupos ni entrenamiento de MiniBatch K-Means.

## Reducción dimensional

Se aplicó PCA a la representación procesada. La varianza acumulada fue:

    Componentes   Varianza explicada
  ------------- --------------------
              7              83.46 %
             10              92.24 %
             14              95.51 %

Se seleccionaron inicialmente 10 componentes por superar el umbral
previamente establecido de aproximadamente 90 % de varianza explicada.
La representación pasó de 54 a 10 dimensiones y de aproximadamente
239.37 MB a 44.33 MB.

Aunque 7 componentes obtuvo mejores métricas internas, su estabilidad
frente a cambios de semilla fue considerablemente menor. Por ello se
mantuvieron 10 componentes como compromiso entre conservación de
información, calidad y reproducibilidad.

## Selección del número de clusters

Se evaluaron valores de `k` entre 2 y 8 utilizando MiniBatch K-Means. La
selección no se basó en una única métrica.

Resultados destacados:

-   `k=2`: Silhouette = 0.2059; Calinski-Harabasz = 4757.09.
-   `k=6`: Davies-Bouldin = 1.6367 y mejor estabilidad frente a
    semillas.
-   Estabilidad de `k=2`: ARI medio = 0.6305, desviación = 0.4445,
    mínimo = 0.1068.
-   Estabilidad de `k=6`: ARI medio = 0.7852, desviación = 0.0889,
    mínimo = 0.6485.

Se seleccionó `k=6`.

## Configuración final

-   PCA: 10 componentes.
-   Varianza explicada: 92.24 %.
-   MiniBatch K-Means.
-   Número de clusters: 6.
-   `random_state`: 99.
-   `batch_size`: 4096.
-   `n_init`: 10.
-   Inercia: 3,364,636.10.

Distribución final:

    Cluster   Registros   Porcentaje
  --------- ----------- ------------
          0     168,233      28.96 %
          1     127,299      21.91 %
          2      52,034       8.96 %
          3      58,535      10.07 %
          4      75,512      13.00 %
          5      99,399      17.11 %

## Comparación PCA vs. referencia sin PCA

  --------------------------------------------------------------------------------------------------------
  Configuración     Dimensiones Memoria MB   Silhouette   Davies-Bouldin   Calinski-Harabasz           ARI
                                                                                               estabilidad
  --------------- ------------- ---------- ------------ ---------------- ------------------- -------------
  Sin PCA                    54     239.37       0.1419           1.7584             2862.31        0.5808

  PCA                        10      44.33       0.1588           1.6321             3322.18        0.7852
  --------------------------------------------------------------------------------------------------------

La representación PCA presentó mejores métricas internas, mayor
estabilidad y una reducción aproximada de 81.5 % en la memoria de la
matriz de características.

## Sensibilidad al número de componentes

  ---------------------------------------------------------------------------------
     Componentes     Silhouette   Davies-Bouldin   Calinski-Harabasz   ARI medio de
                                                                        estabilidad
  -------------- -------------- ---------------- ------------------- --------------
               7         0.1787           1.5308             3819.92         0.4664

              10         0.1588           1.6321             3322.18         0.7852

              14         0.1351           1.8640             2805.63         0.5390
  ---------------------------------------------------------------------------------

Siete componentes mejora la separación interna, pero diez componentes
produce una partición mucho más estable. Esto muestra que conservar más
varianza o maximizar una métrica aislada no garantiza una solución más
reproducible.

## Validación externa

Solo después de seleccionar el modelo final se utilizó `Cover_Type` como
referencia externa.

-   Adjusted Rand Index (ARI): **0.0251**
-   Normalized Mutual Information (NMI): **0.0561**

La correspondencia con las clases conocidas es baja. Esto no convierte
el procedimiento en clasificación: `Cover_Type` no intervino en la
construcción del modelo.

Los clusters no deben interpretarse como sustitutos de las clases de
cobertura forestal.

## Perfiles de los clusters

-   **Cluster 0 - Terreno moderado y accesible:** pendiente baja,
    distancias a hidrología, carreteras y puntos de fuego inferiores al
    promedio.
-   **Cluster 1 - Terreno elevado de orientación occidental:** elevación
    algo superior, pendiente relativamente baja y mayor `Aspect`.
-   **Cluster 2 - Terreno bajo y escarpado:** elevación baja, pendiente
    alta y `Hillshade_9am` muy inferior al promedio.
-   **Cluster 3 - Terreno elevado alejado de hidrología:** mayor
    elevación y grandes distancias horizontal y vertical respecto a
    hidrología.
-   **Cluster 4 - Terreno bajo, inclinado y sombreado:** elevación baja,
    pendiente alta y valores reducidos de `Hillshade_Noon` y
    `Hillshade_3pm`.
-   **Cluster 5 - Terreno remoto de Wilderness Area 0:** grandes
    distancias a carreteras y puntos de fuego; 97.17 % pertenece a
    `Wilderness_Area_0`.

Los nombres son descriptivos y no representan categorías naturales,
definitivas ni causales.

## Evaluación crítica

La solución seleccionada presenta estructura cartográfica interpretable
y estabilidad razonable, pero la separación matemática es limitada
(`Silhouette = 0.1588`) y la concordancia con `Cover_Type` es muy baja.
Su utilidad principal es exploratoria y descriptiva: segmentar
observaciones con perfiles cartográficos semejantes, no predecir
cobertura forestal.

Entre las limitaciones se encuentran el uso de distancia euclidiana
sobre una combinación de variables cuantitativas y binarias, la
sensibilidad a la inicialización y al número de componentes, el
solapamiento observado en la proyección bidimensional y la baja
correspondencia externa.

Un procedimiento adicional para fortalecer la confiabilidad sería
comparar los resultados con otros algoritmos de clustering y medidas de
distancia adecuadas para datos mixtos, además de estudiar estabilidad
mediante remuestreo.

## Estructura del repositorio

``` text
estadistica-big-data-saucedo-abdiel/
├── README.md
├── requirements.txt
├── notebooks/
│   └── actividad_4_1_modelado.ipynb
└── figures/
    ├── varianza_acumulada_pca.png
    ├── silhouette_por_k.png
    ├── davies_bouldin_por_k.png
    ├── clusters_pca_2d.png
    └── perfiles_clusters.png
```

## Reproducción

1.  Crear un entorno con Python 3.13.9.
2.  Instalar las dependencias de `requirements.txt`.
3.  Abrir `notebooks/actividad_4_1_modelado.ipynb`.
4.  Ejecutar las celdas en orden.
5.  El dataset se obtiene directamente mediante
    `fetch_covtype(as_frame=True)`, por lo que no es necesario almacenar
    el conjunto original en el repositorio.

## Entorno utilizado

-   Python 3.13.9 (Anaconda)
-   NumPy 2.3.5
-   Pandas 2.3.3
-   scikit-learn 1.7.2
-   Matplotlib 3.10.6

## Conclusión

La configuración PCA de 10 componentes y MiniBatch K-Means con seis
grupos ofrece el mejor compromiso encontrado entre reducción
dimensional, métricas internas y estabilidad. Los grupos poseen perfiles
cartográficos diferenciables, pero presentan separación limitada y
escasa correspondencia con `Cover_Type`. Por tanto, su valor es
principalmente exploratorio y descriptivo y no deben considerarse clases
naturales, definitivas o equivalentes a los tipos conocidos de
cobertura.
