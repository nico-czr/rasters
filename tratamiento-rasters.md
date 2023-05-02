# Tratamiento de los rásteres

## Flujos de lava: Q-LavHA

Este es el procedimiento usado para convertir los rásteres determinísticos, entregados por Laharz-py, en mapas de peligro.

Se hacen las 236 simulaciones de Q-LavHA en QGIS, y se obtiene un ráster en formato `.asc` por cada una de estas simulaciones.

Se aplica la función `r.null` a cada raster con la caja de herramientas GRASS, y al mismo tiempo se convierte el ráster desde el formato `.asc` al formato `.tif`.

A los rásteres se les aplica la función de la calculadora ráster para convertir a valores de `0` todas las celdas con valores de probabilidad inferiores a `0.001`, y convertir a valores de `1`, todas las celdas con valores de probabilidad superiores a `0.001`.

En cada uno de los 236 rásteres, las celdas por donde no pasan flujos de lava tienen un valor de `0`, y las celdas donde pasan flujos de lava tienen un valor de `1`.

Finalmente, se aplica la función `r.null` de la caja de herramientas GRASS, para comprimir los rásteres.

Los 236 rásteres son cargados a Google Drive, e importados desde un documento de Jupyter Notebook en Google Colab, con la librería Rasterio.

Los rasteres son multiplicados por las constantes finales entregadas por el árbol de eventos propuesto (es decir, los valores de probabilidad media, el 10° percentil y el 90° percentil).

Los rásteres ponderados se suman y se dividen por la sumatoria de los valores de todos los eventos ponderados para formar tres rásteres integrados, un ráster final con la probabilidad media, y otros dos rásteres con la probabilidad de los percentiles 10° y 90°.

Los nuevos rásteres ponderados son exportados en formato `.tif` con rasterio y con compresión `lzw`, descargados desde Google Drive, e importados nuevamente a QGIS.

En el caso de la metodología referencial del árbol de eventos, se hace un paso previo donde se promedian todos los rásteres que representan eventos que ocurren desde cada uno de los 26 sectores del sistema volcánicos.


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
