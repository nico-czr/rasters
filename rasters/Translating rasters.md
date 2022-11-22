## Probability rasters

Each cell of the probability rasters has values that range from 0 to 1 that correspond to the conditional probabilities of a given volcanic phenomenon reaching a particular location (or the likelihood of spatial distribution of a given phenomenon). The probabilistic model-toolbox Tephra2-TephraProb outputs probabilistic rasters, therefore the mean of these rasters is easily done with the statistical mean formula shown in Equation 1, widely used in volcanic hazard assessments ([Sandri et al., 2016](); [Biass et al., 2016b](); [Charbonnier et al., 2020](), among others). However, the deterministic models (i.e., VolcFlow and LaharFlow) output thickness rasters that must be corrected by converting all their thickness values above zero to a value of one—also known as the ‘hazard footprint’ ([Loughlin et al., 2015]())—to then obtain the mean of all the simulations through Equation 1. In the case of the Decreasing Probability model, as it is not a geophysical model, but a statistical/empirical model, its results have been treated as if they were from a deterministic model, this means that all probabilities above zero have been converted to one, and then the mean has been obtained through **Equation 1**. This agrees with how the probabilities of empirical/statistical models have been estimated in other assessments, for example, for the Energy Cone model ([Tierz et al. 2018](); [Clarke et al. 2020]()). Finally, we give a uniform temporal occurrence probability to each simulated intra-scenario.

**Equation 1**

$$x = \frac{\sum^{n}_{i=1}{x_{i}}}{n}$$

Tephra fallout. The probability rasters of each tephra fallout scenario are obtained by exporting the intra-scenario probability ESRI ASCII (.asc) rasters from MATLAB, by using the TephraProb toolbox. We used the probability rasters with a tephra accumulation threshold of 10 kg m–2. Then, these rasters are imported to QGIS and interpolated with the GRASS function (r.resamp.interp) from a resolution of 6.0, 4.0, and 2.5 km to 2.0 km. For instance, the small-size tephra fallout scenario has an eruptive magnitude between 3.0–4.0, and five intra-scenarios. To obtain the probability raster of this scenario, we add the probability rasters of its five intra-scenarios and divide them by the total number of small size intra-scenarios (n = 5) by using Equation 1 with the GDAL raster calculator. The probability rasters of the medium and large size scenarios are obtained following the same procedure.

Concentrated PDCs. The probability raster of concentrated PDCs is obtained by exporting ESRI ASCII (.asc) rasters from MATLAB by using the ArcGridWrite function. Then, these rasters are loaded in ArcGIS ArcMap and corrected by flipping them upside down along the horizontal axis by using the ArcMap function (flip raster), because the ArcGridWrite function wrongfully flips the horizontal axis of the ASCII raster matrices. These raster matrices are then loaded in QGIS and all thickness values above zero are converted to a value of one with the GDAL raster calculator (Code 1). The mean probabilities are obtained by using Equation 1 with the GDAL raster calculator (n = 30). Then, the values are normalized with the SAGA function (raster normalization), so that all cells had values ranging from zero to one. Finally, we smoothed the values of the rasters with the GRASS function (r.neighbors).

```
'raster_file'> 0 = 1   
```

> **Code 1**

Lahars. The probability raster of lahars is obtained by downloading the ‘maximum height’ vector data files (.kml) from LaharFlow and importing them to QGIS. These files have an error in the thickness column as it is populated with text values (e.g., ‘h = 0.234’) instead of numbers. To correct this, a new column labeled ‘thickness’ with only the thickness value must be created. To eliminate the ‘h = ‘ portion we created a new column that included  code 2.

```
replace(Name, ‘h_max =’, ‘ ‘)
```

> **Code (2)**

Then, these vector files are converted to raster format by using the GDAL function (rasterizer), with the ‘thickness’ column as the selected field value that will populate the cells of the raster. After, all thickness values above zero are converted to one with the GDAL raster calculator. The mean probabilities are obtained by using  Equation 1 with the GDAL raster calculator (n = 56). The same normalization and smoothing procedures described above are applied to the LaharFlow rasters.

Blocky lava flows. The probability raster of blocky lava flows is obtained by filling with zeros the no-data cell values of the 32 resulting raster matrices using the GRASS function (r.null). Then, all values above zero are converted to a value of one with the GDAL raster calculator. The mean probabilities are obtained by using  Equation 1 with the GDAL raster calculator (n = 32). Then, the values are normalized with the SAGA function (raster normalization), so that all cells have values ranging from zero to one. Finally, the values of the rasters are smoothed with the GRASS function (r.neighbors).

## References 

Sandri, L., Costa, A., Selva, J., Tonini, R., Macedonio, G., Folch, A., and Sulpizio, R. (2016). Beyond eruptive scenarios: Assessing tephra fallout hazard from Neapolitan volcanoes. Scientific Reports, 6(1), 24271. https://doi.org/10.1038/srep24271

Biass, S., Bonadonna, C., Connor, L., and Connor, C. (2016b). TephraProb: A Matlab package for probabilistic hazard assessments of tephra fallout. Journal of Applied Volcanology, 5(1), 10. https://doi.org/10.1186/s13617-016-0050-5

Charbonnier, S. J., Thouret, J.–C., Gueugneau, V., and Constantinescu, R. (2020). New Insights Into the 2070 cal yr BP pyroclastic currents at El Misti volcano (Peru) from field investigations, satellite imagery and probabilistic modeling. Frontiers in Earth Science, 8, 557788. https://doi.org/10.3389/feart.2020.557788

Clarke, B., Tierz, P., Calder, E., and Yirgu, G. (2020). Probabilistic volcanic hazard assessment for pyroclastic density currents from pumice cone eruptions at Aluto Volcano, Ethiopia. Frontiers in Earth Science, 8, 348. https://doi.org/10.3389/feart.2020.00348

Loughlin, S. C., Sparks, S., Brown, S. K., Jenkins, S. F., and Vye-Brown, C. (Eds.). (2015). Global volcanic hazards and risk (1st ed.). Cambridge University Press. https://doi.org/10.1017/CBO9781316276273

Tierz, P., Stefanescu, E. R., Sandri, L., Sulpizio, R., Valentine, G. A., Marzocchi, W., and Patra, A. K. (2018). Towards Quantitative Volcanic Risk of Pyroclastic Density Currents: Probabilistic Hazard Curves and Maps Around Somma-Vesuvius (Italy). Journal of Geophysical Research: Solid Earth. https://doi.org/10.1029/2017JB015383
