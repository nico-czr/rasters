# Tratamiento de los rásteres



## Flujos de lava: Q-LavHA

Este es el procedimiento usado para convertir los rásteres determinísticos, entregados por Q-LavHA decreasing probability, en mapas de peligro.

Q-lavHA es un plugin que puede ser usado en QGIS, desde la versión de QGIS 3 en adelante.

Primero, se hacen las 236 simulaciones de Q-LavHA en QGIS, y se obtienen 236 rásters en formato `.asc`.

Se aplica la función `r.null` a cada raster con la caja de herramientas GRASS, ya esto permite: (1) convertir todas las celdas del raster con valor `no data` en celdas con valor `0`, (2) convierte el ráster desde el formato `.asc` al formato `.tif`, y (3) aplica la compresión `lzw` al raster en formato `.tif`, lo que logra reducir mucho el peso del archivo en formato `.asc`, desde unos 200 MB a 2 MB. Lo cual es ideal para no trabajar con archivos con tanto peso en Google Drive y en Google Colab.

Se crea un modelo gráfico en QGIS donde sólo se utiliza la función `calculadora raster` con el siguiente código:
```
'input-raster' > 0.001 = 1
```
Este codigo hace que, dentro del raster, todas las celdas con valores de probabilidad superiores a `0.001` se conviertan en `1`, y todas las celdas con valores de probabilidad inferiores a `0.001` se convierten en celdas con valor `0`. 

El modelo gráfico se ejecuta como "proceso por lotes" (batch process) se va tratando los 236 rasters en grupos de 26 rasters. 
Finalmente se obtienen 236 rásteres, donde las celdas por donde no pasan flujos de lava tienen un valor de `0`, y las celdas por donde pasan flujos de lava tienen un valor de `1`.

Los 236 rásteres son cargados a Google Drive, e importados desde un documento de Jupyter Notebook en Google Colab, usando la librería `rasterio`. La cual se puede usar en Google Colab con el siguiente código:
```
pip install rasterio
```
Dentro del documento de jupyter notebook se utiliza también la librería `tensorflow` de python para poder ejecutar las distribuciones Beta y Dirichlet de cada nodo.

```
import tensorflow as tf
import tensorflowprobability as tfp

N1_a01_prior = 1 # alpha hyperparameter
N1_a02_prior = 1 # beta hyperparameter

n = 161                        # n
N1_a01_y = 50                  # alpha hyperparameter likelihood value
N1_a02_y = n - N1_a01_y        # beta hyperparameter likelihood value (= 111)

N1_a01_post = N1_a01_prior + N1_a01_y  # alpha hyperparameter a posteriori Beta
N1_a02_post = N1_a02_prior + N1_a02_y  # beta hyperparameter a posteriori Beta

N1_post = tfp.distributions.Beta(N1_a01_post, N1_a02_post)
N1_a01 = N1_post_distribution.sample(100000)
N1_a02 = 1 - N1_a01_post
```

Luego, se extrae la media del tensor (array de 100000 valores de probabilidad posibles), y se obitene un valor numérico.

```
N1_a01_mean = tf.reduce_mean(N1)
```
Tambien se extraen los intervalos de credibilidad de la media por medio de los percentiles 10° y 90°
```
N1_a01_quantiles = tfp.stats.quantiles(N1_a01, 98)

N1_a01_perc10 = tf.slice(N1_a01_quantile, begin=[9], size=[1])

N1_a01_perc90 = tf.slice(N1_a01_quantile, begin=[89], size=[1])
```


Los resultados de cada nodo se convierten en valores constantes:
```
#reference BET

#mean of node 1-2-3$\alpha$ (eruption in 2023)
N1_a01_mean = tf.constant(0.31, shape=None, type=tf.float32)

#mean of node 4$\alpha_{13}$ (eruption in the main crater)
N4_a13_mean = tf.constant(0.60, shape=None, type=tf.float32)

#mean of node 5$\alpha_{5}$ (VEI 3 eruption)
N5_a02_mean = tf.constant(0.80, shape=None, type=tf.float32)

#mean of node 6\alpha.3 (lava flow occurrence given a VEI 3 eruption)
N63_mean = tf.constant(0.89, shape=None, type=tf.float32)

#proposed BET

#mean of node 1-2-3$\alpha$ (eruption in 2023)
N1_mean = tf.constant(0.31, shape=None, type=tf.float32)

#mean of node 4$\alpha_{13}$ (eruption in the main crater)
N4_a13_mean = tf.constant(0.60, shape=None, type=tf.float32)

#mean of node R$\alpha_{5}$ (lava flow occurrence given any eruption)
NR_mean = tf.constant(0.91, shape=None, type=tf.float32)

#mean of node S$\alpha_{5}$ (lava flow of length magnitude 5)
NS_a05_mean = tf.constant(0.89, shape=None, type=tf.float32)
```

Estos valores son multiplicados entre si para obtener la probabilidad media de un evento terminal del árbol de eventos. 

```
#reference BET (Mean)
N1_a01_N4_a13_N5_a02_N63_a01_mean = N1_a01_mean * N4_a13_mean * N5_a02_mean * N63_a01_mean

#proposed BET (Mean)
N1_a01_N4_a13_NR_a01_NS_a05_mean = N1_a01_mean * N4_a13_mean * NRa01_mean * NS_a05_mean

#reference BET (10th Percentile)
N1_a01_N4_a13_N5_a02_N63_a01_perc10 = N1_a01_perc10 * N4_a13_perc10 * N5_a02_perc10 * N63_a01_perc10

#proposed BET (90th Percentile)
N1_a01_N4_a13_NR_a01_NS_a05_perc10 = N1_a01_perc10 * N4_a13_perc10 * NRa01_perc10 * NS_a05_perc10

#reference BET (10th Percentile)
N1_a01_N4_a13_N5_a02_N63_a01_perc90 = N1_a01_perc90 * N4_a13_perc90 * N5_a02_perc90 * N63_a01_perc90

#proposed BET (90th Percentile)
N1_a01_N4_a13_NR_a01_NS_a05_perc90 = N1_a01_perc90 * N4_a13_perc90 * NRa01_perc90 * NS_a05_perc90
```
Esto se hace para todos los eventos terminales que desencadenan un evento de flujo de lava.

Los rasteres son multiplicados por las constantes finales entregadas por el árbol de eventos propuesto (es decir, los valores de probabilidad media, el 10° percentil y el 90° percentil).

Los rásteres ponderados se suman y se dividen por la sumatoria de los valores de todos los eventos ponderados para formar tres rásteres integrados, un ráster final con la probabilidad media, un ráster final con la probabilidad del 10° percentil, y un ráster final con la probabilidad del 90° percentil.

Usando `rasterio` Los nuevos rásteres ponderados son exportados a Google Drive, en formato `.tif`  y con una compresión `lzw`, por lo que no superan unos 7 MB de tamaño.

Luego, son descargados desde Google Drive, e importados nuevamente a QGIS para generar las Figuras.

Cabe destacar, que en el caso de la metodología referencial del árbol de eventos, se hace un paso previo donde se promedian todos los rásteres que representan eventos que ocurren desde cada uno de los 26 sectores del sistema volcánico.


## Flujos de agua-sedimento: Laharz-py

Este es el procedimiento usado para convertir los rásteres determinísticos, entregados por Laharz-py, en mapas de peligro.

Se hacen las 312 simulaciones de Laharz-py en ArcGIS ArcMap.

Al exportar se usa la función `copy raster` como `batch` o `proceso por lotes`, y se exportan los rásteres en formato `.tif`, donde el formato de los píxel de los rásteres es `float32`. Cada uno de estos rásteres puede contener más de un evento según la forma en la que se hagan las simulaciones.

Los rásteres se importan, y se les aplican funciónes de la calculadora ráster para extraer eventos individuales de cada ráster con más de un evento.

En el raster con más de un evento, los lugares por donde no pasan flujos tienen un valor de $x$; los rasters donde pasan flujos tienen valores superiores a $x$.

Por ejemplo, si en Laharz-py, desde una coordenada se hicieron 8 simulaciones con 8 volumenes distintos, entonces el raster tiene un valor por cada simulación.

Por ejemplo, la simulación de magnitud de clase 6 poseen el número 7 en el raster. La idea es extraer estos valores del raster principal y generar un raster individual para cada clase, y que tengan sólo valores de `0` (no ocurre) y `1` (ocurre).

Con la calculadora raster, los valores de $x$ se convierten a valores de `0` (0 %), y los valores superiores a $x$ se convierten a valores de `1` (100 %)

Finalmente, se aplica la función `r.null` de la caja de herramientas GRASS, para comprimir los rásteres.

Los 312 rásteres son cargados a Google Drive, e importados desde un documento de Jupyter Notebook en Google Colab, con la librería Rasterio.

Los rasteres son multiplicados por las constantes finales entregadas por el árbol de eventos propuesto (es decir, los valores de probabilidad media, el 10° percentil y el 90° percentil).

Los rásteres se suman para formar tres rásteres integrados, un ráster final con la probabilidad media, y otros dos rásteres con la probabilidad de los percentiles 10° y 90°.

Los nuevos rásteres ponderados son exportados en formato `.tif` con rasterio y con compresión `lzw`, descargados desde Google Drive, e importados nuevamente a QGIS.

En el caso de la metodología referencial del árbol de eventos, se hace un paso previo donde se promedian todos los rásteres que representan eventos que ocurren desde cada uno de los 26 sectores del sistema volcánicos.


  

## Caída de tefra: Tephra2-TephraProb


Este es el procedimiento usado para convertir los rásteres probabilísticos, entregados por Tephra2-TephraProb, en mapas de peligro.

Desde Tephra2-TephraProb se exportaron 28 rasters de cada simulación probabilística con um brales de acumulación de tefra de 10 $kg m^{-3}$.

Se importan los 28 rásteres con los resultados en formato texto (`grilla ASCII de ESRI`) a QGIS.

Se interpolan los ráster desde 3, 6, y 7 km a 1 km con la función `r.resamp.interp` de la caja de herramientas GRASS.

A los rásteres se les aplica  la función `r.null` para reemplazar las celdas de los límites del ráster con valores de `0`, ya que la interpolación deja a estas celdas como `sin datos`.

Esto rasteres se guardan, pero su extensión se modifica, para que sean iguales a las coordenadas del ráster de mayor tamaño (es decir, el raster con las erupciones de magnitud VEI 5), con la intención de poder sumar los rásteres en los pasos futuros.

Estos rasters se pasan por la función `r.null` nuevamente para reemplazar las celdas `sin datos` con valores de `0`, que fueron producidas al extender la extensión de los rásters de las magnitudes VEI 3 y VEI 4.

Los 28 rasters son importados a un documento de Jupyter Notebook, con la librería Rasterio.

Los rasteres son multiplicados por las constantes finales entregadas por el árbol de eventos (es decir, los valores de media, el 10° percentil y el 90° percentil).

Los rásteres ponderados se suman y se dividen por la sumatoria de los valores de todos los eventos ponderados para formar tres rásteres integrados, un ráster final con la probabilidad media, y otros dos rásteres con la probabilidad de los percentiles 10° y 90°.

También se generan rásteres según la magnitud eruptiva, y rásteres según la estación climática en la que se desarrolla la erupción (con rásteres que representan las probabilidades de los percentiles 10° y 90°).

Los nuevos rásteres ponderados son exportados en formato `.tif` con rasterio, e importados nuevamente a QGIS para crear las figuras.
