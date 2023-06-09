Áreas quemadas en el parque nacional José del Carmen Ramírez,
marzo-abril 2023
================
José Ramón Martínez Batlle. <br> Última actualización: 9 de abril de
2023

Versión HTML (más legible e interactiva),
[aquí](https://geofis.github.io/fuego-jose-del-carmen-ramirez-2023/README.html)

``` r
# Paquetes
library(raster)
library(sf)
library(kableExtra)
library(tidyverse)
estilo_kable <- function(df, titulo = '', cubre_anchura = T) {
  df %>% kable(format = 'html', escape = F, booktabs = T, digits = 2, caption = titulo) %>%
  kable_styling(bootstrap_options = c("hover", "condensed"), full_width = cubre_anchura)
}

#Todo el parque. Límite mínimo del intervalo
dnbr <- raster('fuentes/dNBR-jose-del-carmen-ramirez-2023-masked-reclass-majority.tif')
esa <- raster('fuentes/esa-jose-del-carmen-ramirez-2023.tif')
dnbr <- resample(dnbr, esa)
dnbr[dnbr==0] <- NA
if(interactive()) plot(dnbr)
april_8 <- st_read('fuentes/mascara-quemado-inferido-imputado-from-landsat-april-8.gpkg', quiet = T)
dnbr_april8 <- rasterize(as(april_8, 'Spatial'), y = dnbr, field = 'DN', update = T)
if(interactive()) plot(dnbr_april8)
hotspots <- st_read('fuentes/mascara-quemado-inferido-imputado-from-hotspots.gpkg', quiet = T)
dnbr_april8_hotspots <- rasterize(as(hotspots, 'Spatial'),
                                  y = dnbr_april8, field = 'DN', update = T)
if(interactive()) plot(dnbr_april8_hotspots)
cross_tab_parque_min <- crosstab(dnbr_april8_hotspots, esa)
cross_tab_parque_min <- cross_tab_parque_min[2, 2:ncol(cross_tab_parque_min)]
names(cross_tab_parque_min) <- c("Trees", "Shrubland", "Grassland", "Cropland",
                      "Built-up", "Barren / sparse vegetation", "Open water")
cross_tab_parque_min <- cross_tab_parque_min[c("Trees", "Shrubland",
                                                 "Grassland", "Cropland")]
pct_parque_min <- round(prop.table(cross_tab_parque_min)*100, 2)
km2_parque_min <- cross_tab_parque_min * prod(res(dnbr_april8_hotspots))/1000000
total_km2_parque_min <- sum(km2_parque_min)

#Todo el parque. Límite máximo del intervalo
planet_april9 <- st_read('fuentes/mascara-quemado-inferido-imputado-from-planet-april-9.gpkg', quiet = T)
dnbr_april8_april9 <- rasterize(as(planet_april9, 'Spatial'),
                                  y = dnbr_april8, field = 'DN', update = T)
if(interactive()) plot(dnbr_april8_april9)
cross_tab_parque_max <- crosstab(dnbr_april8_april9, esa)
cross_tab_parque_max <- cross_tab_parque_max[2, 2:ncol(cross_tab_parque_max)]
names(cross_tab_parque_max) <- c("Trees", "Shrubland", "Grassland", "Cropland",
                      "Built-up", "Barren / sparse vegetation", "Open water")
cross_tab_parque_max <- cross_tab_parque_max[c("Trees", "Shrubland",
                                                 "Grassland", "Cropland")]
pct_parque_max <- round(prop.table(cross_tab_parque_max)*100, 2)
km2_parque_max <- cross_tab_parque_max * prod(res(dnbr_april8))/1000000
total_km2_parque_max <- sum(km2_parque_max)

#Área de alarma social (próxima al pico Duarte). Límite mínimo del intervalo
aoi <- st_read('fuentes/sector-norte-frontera-agropecuaria.gpkg', quiet = T)
dnbr_april8_hotspots_aoi <- mask(crop(dnbr_april8_hotspots, extent(aoi)), aoi)
esa_aoi <- mask(crop(esa, extent(aoi)), aoi)
cross_tab_aoi_min <- crosstab(dnbr_april8_hotspots_aoi, esa_aoi)
cross_tab_aoi_min <- cross_tab_aoi_min[, 2:4]
names(cross_tab_aoi_min) <- c("Trees", "Shrubland", "Grassland")
pct_aoi_min <- round(prop.table(cross_tab_aoi_min)*100, 2)
km2_aoi_min <- cross_tab_aoi_min * prod(res(dnbr_april8_hotspots_aoi))/1000000
total_km2_aoi_min <- sum(km2_aoi_min)

#Área de alarma social (próxima al pico Duarte). Límite máximo del intervalo
dnbr_april8_april9_aoi <- mask(crop(dnbr_april8_april9, extent(aoi)), aoi)
cross_tab_aoi_max <- crosstab(dnbr_april8_april9_aoi, esa_aoi)
cross_tab_aoi_max <- cross_tab_aoi_max[, 2:4]
names(cross_tab_aoi_max) <- c("Trees", "Shrubland", "Grassland")
pct_aoi_max <- round(prop.table(cross_tab_aoi_max)*100, 2)
km2_aoi_max <- cross_tab_aoi_max * prod(res(dnbr_april8_april9_aoi))/1000000
total_km2_aoi_max <- sum(km2_aoi_max)
```

> Contribuye a enmarcar adecuadamente la narrativa sobre este incendio.
> No se trata del “incendio del pico Duarte”, se trata de una
> desproporcionada actividad de quema DENTRO del parque nacional José
> del Carmen Ramírez, una de las áreas protegidas más afectadas en
> nuestro país por la agricultura migratoria y la agricultura de
> subsistencia.

![Áreas quemadas en el parque nacional José del Carmen Ramírez,
marzo-abril de 2023](img/superficie-quemada-conjunto.jpg)

## Resultados

### El dato (en desarrollo)

Un incendio forestal que inició en marzo dentro del parque nacional José
del Carmen Ramírez, cordillera Central de República Dominicana, quemó
superficies considerables de bosque de pinar. El hecho de que algunos
focos se acercaran al pico Duarte (máxima elevación del Caribe insular),
popularizó este desafortunado episodio de la historia del parque.

Es probable que hayas llegado hasta aquí buscando “cuánto se quemó”,
pues usamos las cifras para establecer comparaciones y, en definitiva,
nos gustan los números. Pero no te quedes sólo con la cifra, pues este
incendio visibiliza un problema aún más complejo: la intensa agricultura
migratoria y de subsistencia dentro del área protegida.

> “A los mayores les gustan las cifras”. Antoine de Saint-Exupéry, *El
> principito*

No puedo darte una cifra cerrada, pues se trata de un evento aún en
desarrollo. En su defecto, te ofrezco un intervalo probable dentro del
cual se encuentra el valor definitivo:
**15.52 $\leq$ $x$ $\leq$ 31.1 $km^2$** (consulta la sección
[Método](#metodo) para obtener detalles sobre el procedimiento de
cálculo). Prácticamente todo lo quemado es bosque (más de un 90%) de
distintas densidades, pero también hay herbazales (cf. pajonales). Este
intervalo contiene, con cierto nivel de confianza, **la cantidad final
de kilómetros cuadrados de área boscosa (en general, pinar) quemados
durante marzo en el “área de mayor preocupación”, que es el borde norte
del parque nacional José del Carmen Ramírez en su aproximación hacia el
pico Duarte**. Como aclaré, esta área se quemó producto de un foco que
inició a finales de marzo en cobertura no boscosa (e.g. frontera
agrícola), y que remontó hacia el norte, adentrándose en el bosque y
acercándose bastante al pico Duarte, donde produjo “alarma social”.

¿Por qué es tan amplio el intervalo? Porque para obtenerlo por medio de
imágenes satelitales que usan sensores ópticos, dependemos de que las
nubes “nos dejen brechar” el terreno (los sensores SAR superan esta
limitación, pero las imágenes gratuitas que usan esta tecnología, son
poco frecuentes y más complejas de usar). De hecho, las estimaciones de
superficie quemada realizadas en tierra, son más precisas en los
primeros momentos del incendio, pero tan pronto se dispone de buenas
escenas (sin nubes, y no mucho tiempo después de la fecha de extinción),
las estimaciones por medio de imágenes satelitales suelen ser mejores.

Pero volvamos al contexto que envuelve a este incendio. Es marzo, las
precipitaciones son muy reducidas y la preparación de tierras usando
quemas está en su momento más álgido; el parque es, literalmente, un
fogón. Es por esta razón que me interesa darle el contexto requerido a
este incendio: todo comenzó dentro de un parque nacional que exhibe una
incesante actividad agrícola, donde la quema se usa de manera regular
(DENTRO del parque, reitero). Por esta razón, si analizamos lo que
ocurrió fuera del área de “alarma social”, no nos debería sorprender que
las cifras de superficies quemadas sean enormes. Veamos el dato, también
como intervalo: **47.59 $\leq$ $x$ $\leq$ 63.22 $km^2$**. Este intervalo
contiene **la cantidad de terreno quemado a partir del 21 de febrero
hasta abril dentro del área protegida** (en enero también se quemó otro
poco). Mucho fue pastizal, matorral o bosque secundario; pero recuerda,
estamos hablando de un parque nacional. La distribución, según
coberturas, es como sigue: en el escenario de menor área quemada, sería
27.36 $km^2$ de bosque, y en el escenario de mayor área quemada 42.39
$km^2$.

![Áreas quemadas en el parque nacional José del Carmen Ramírez,
marzo-abril de 2023. El detalle muestra amplía el borde norte que enlaza
con el PN Armando Bermúdez. Los polígonos sombreados en naranja son
áreas quemadas calculadas por el método dNBR, que ofrece una alta
confiabilidad. Los polígonos de trazo discontinuo son áreas que se han
imputado como quemadas. Los de trazo discontinuo amarillo proceden de
interpretación visual de imagen Landsat 9 de 8 abril (confiable), y
envolventes convexas de los puntos de calor / anomalías térmicas de
FIRMS. El de trazo de discontinuo naranja, encierra parcialmente (con
islas incluidas) la superficie visiblemente quemada en una imagen de
Planet de 9 de abril de
2023.](img/dnbr-interpretacion-visual-8-abril-hotspots.jpg)

No nos hemos “sentado” a tener una discusión a nivel de país sobre
muchas de nuestras áreas protegidas, pero esta es la norma respecto del
conjunto del sistema: hemos discutido poco. Es relevante que ni tan
siquiera las áreas más carismáticas, como Los Haitises, sierra de
Bahoruco o Valle Nuevo, han podido salir de este círculo de degradación
ambiental. De éstas últimas se habla, “salen” en los medios, pero el
parque nacional José del Carmen Ramírez “no suena”.

Hay algunas señales preocupantes sobre lo que está ocurriendo en el
parque nacional José del Carmen Ramírez. Año tras año, la frontera
agrícola avanza más y más hacia el norte y al oeste, que son los únicos
cuadrantes todavía con bosque del área protegida. En concreto, en la
loma El Picacho (oeste del parque) un incendio consumió varias hectáreas
de bosque.

![Áreas de bosque afectadas por fuego en la loma el
Picacho](img/loma-el-picacho.jpg)

Si buscas la importancia de esta área protegida, encontrarás referencias
sobre su papel en la captación de agua, o sobre su destacada
biodiversidad. Sin embargo, te lo voy a poner fácil: sin José del Carmen
Ramírez no habría Armando Bermúdez y, sin este último, “se pué cuidá el
Cibao (y to’ el país)”. No perdamos el foco, esto es serio, el país
necesita una discusión profunda sobre el presente y el futuro de este
parque nacional.

## Método

### Obtención de los intervalos

Usé escenas Sentinel 2 para caracterizar la superficie quemada durante
todo el mes de marzo y parte de abril. La fuente prefuego fue una
composición promediada de escenas 22/02/2023 y 27/02/2023; la fuente
posfuego fue una única escena, de fecha 3/04/2023. Para extraer la
superficie quemada, apliqué una versión modificada del [*script* Google
Earth Engine de UN-Spider “BURN SEVERITY MAPPING USING THE NORMALIZED
BURN RATIO
(NBR)”](https://un-spider.org/advisory-support/recommended-practices/recommended-practice-burn-severity/burn-severity-earth-engine),
que consiste en calcular la severidad de quemado usando el diferencial
del índice normalizado de quema (dNBR). Este método ofrece el cálculo
más consistente y preciso de superficie quemada disponible actualmente.
A esta superficie le imputé áreas quemadas adicionales extraídas desde
otras fuentes para construir los límites inferior y superior del
intervalo, respectivamente.

Para obtener el límite inferior agregué, al cómputo base, áreas
interpretadas como quemadas por medio de inspección visual de la escena
Landsat 9 adquirida el 8/04/2023 (ID:
LC09_L1TP_008047_20230408_20230408_02_T1). También agregué las
envolventes convexas de los puntos de calor / anomalías térmicas de
FIRMS correspondientes a los 7 días anteriores al 8/04/2023. Por otra
parte, para obtener el límite superior del intervalo agregué, al computo
base, áreas quemadas interpretadas como tal por medio de inspección
visual de escenas Planet de 9 de abril (ID: 20230409_142601_27_24b9 y
20230409_142603_61_24b9).

En futuras entregas, en la medida en la que nuevas escenas se hagan
disponibles, aplicaré el método dNBR de forma consistente a toda el área
protegida para obtener un cómputo fiable. De esta manera, encogeré la
anchura del intervalo en próximas actualizaciones. “Mantengan la
sintonía”.

### La cobertura “bosque”

La superficie de bosque, que se trata sobre todo de pinar, parte del
dato ofrecido por la ESA (Zanaga, Daniele et al. 2022). Realicé un
simple chequeo cruzado de esta fuente, y noté un error muy común en
estas clasificaciones: las áreas situadas en sombra de relieve, sobre
todo en contextos muy intervenidos o de cobertura variable, suelen
aparecer clasificadas como bosque cuando probablemente son sólo
pastizales o matorrales. Es decir, en general, la cobertura “*tree
cover*” de Zanaga, Daniele et al. (2022) está sobrestimada en áreas
intervenidas. Una alternativa sería usar la capa de cobertura forestal
dominicana del estudio de 2019, siempre que esté disponible sin previa
petición, que no es el caso (ya estoy muy viejo para estar escribiendo
la típica carta). Hay más alternativas, como [Copernicus Global Land
Cover
Layers](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_Landcover_100m_Proba-V-C3_Global)
o [Hansen Global Forest Change v1.9
(2000-2021)](https://developers.google.com/earth-engine/datasets/catalog/UMD_hansen_global_forest_change_2021_v1_9),
pero no encontraremos grandes diferencias, al menos en la parte más
boscosa del parque. No obstante, la fuente suele ser consistente en
áreas donde el bosque es generalizado, lo cual en el parque nacional
José del Carmen Ramírez, ocurre hacia el norte y al noroeste.

## Referencias

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-zanagadaniele2022" class="csl-entry">

Zanaga, Daniele, Van De Kerchove, Ruben, Daems, Dirk, De Keersmaecker,
Wanda, Brockmann, Carsten, Kirches, Grit, Wevers, Jan, et al. 2022. “ESA
WorldCover 10 m 2021 V200,” October.
<https://doi.org/10.5281/ZENODO.7254221>.

</div>

</div>
