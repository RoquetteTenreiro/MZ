# Uncertainties associated with the delineation of management zones in precision agriculture

![Image description](cover_MZ.jpeg)

> Author: Tomás Roquette Tenreiro

> Institute for Sustainable Agriculture (IAS-CSIC) - Córdoba, 2020

#### 1. Get started
### 1.1 R-Markdown script

This is an R-Markdown script presentation. R-Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R-Markdown see <http://rmarkdown.rstudio.com>. This document includes both general instructions and comments by the author as well as the script code developed and the output of any embedded R code chunks within this document.

### 1.2 General introduction

The following R-script aims to describe an analytical procedure that combined multiple spatial data in order to delineate management zones for sampling and soil moisture probe installation. The main goal is to distinguish management zones based on geo-physical (i.e. orientation, elevation, texture, ECa), chemical (pH), and biological (plant vigor) properties within a crop field. The selected field is about 9.5 ha, located in the arable region of Cordoba. This document aims also to deliver a script that can be adjusted to similar analyses or function as a guide to conduct geospatial analysis with R. The reader can use this document both as a dissemination and a decision supporting tool.

### 1.3 Necessary material - R libraries    

The first step consists on installing all necessary libraries for this analysis: 

```{r}
install.packages("rmarkdown")
install.packages("dplyr")
install.packages("plyr")
install.packages("reshape2")
install.packages("agricolae")
install.packages("quantreg")
install.packages("ploty")
install.packages("sf")
install.packages("raster")
install.packages("spData")
install.packages('spDataLarge', repos='https://nowosad.github.io/drat/', type='source')
install.packages("gridExtra")
install.packages("RColorBrewer")
install.packages("root.dir")
install.packages("tidyverse")
install.packages("ggplot2")
install.packages("wesanderson")
install.packages("ggpmisc")
install.packages("knitr")
install.packages("installr")
install.packages("lmtest", repos = "http://cran.us.r-project.org")
install.packages("tinytex")
```
and updating all the installed libraries in R-studio.

```{r}
library(knitr)
library(sf)
library(dplyr)  
library(plyr)
library(reshape2)
library(agricolae)
library(quantreg)
library(plotly)
library(sp)
library(spDataLarge)
library(spData)
library(tmap)
library(raster)
library(gridExtra)
library(quantreg)
library(ggplot2)
library(RColorBrewer)
library(tidyverse)
library(gridExtra)
library(wesanderson)
library(ggpmisc)
library(markdown)
library(tinytex)
library(lmtest)
```

### 1.4 Initial details - working directory

In this section we set initial details to specify the working directory; in this particular case the analysis was linked to the internal folder "Experimental_Catchment_Cordoba_19_20" where input and output data is saved. To run this code please specify the working directory where your input files are saved. 

```{r setup, include=FALSE}
rm(list=ls())
getwd()
knitr::opts_chunk$set(echo = TRUE)
knitr::opts_knit$set(root.dir = "C:/Users/Tomas R. Tenreiro/Desktop/Experimental_Catchment_Cordoba_19_20")
```

#### 2 Historical remote sensing data

### 1.1 Input material

This section uploads all input material. In this particular case, we will work with satellite data (Landsat-8 and Sentinel-2), atmospherically corrected (and cloud cover < 4%), that was downloaded from EO-browser (https://apps.sentinel-hub.com/). The script considers imagery from five different growing seasons (i.e. 2015, 2016, 2017, 2018 and 2019). In order to explore spatial correlation between plant vigor and soil properties under rainfed conditions, we focused on late phenological stages before crop senescence. The two consecutive images taken imediatly after flowering stage were selected for each year. 2015 and 2017 imagery corresponds to a sunflower crop, 2016 and 2018 to a winter wheat crop and 2019 imagery corresponds to a canola crop. 

| Year | Crop             | Date 1     | Date 2     | Satellite   | Sp.Resolution |
|------|------------------|------------|------------|-------------|---------------|
| 2015 | Sunflower        | 05/6/2015  | 07/7/2015  | Landsat-8   | 30x30 m       |
| 2016 | Wheat (durum)    | 07/6/2016  | 23/6/2016  | Landsat-8   | 30x30 m       |
| 2017 | Sunflower        | 10/6/2017  | 26/6/2017  | Landsat-8   | 30x30 m       |
| 2018 | Wheat (durum)    | 22/5/2018  | 03/6/2018  | Sentinel-2  | 10x10 m       |
| 2019 | Canola           | 14/4/2019  | 27/4/2019  | Sentinel-2  | 10x10 m       |

```{r}
# Date 2015 -1
Red.2015.1 <- raster("SAT Imagery/Red_05.06.2015.tiff")
NIR.2015.1 <- raster("SAT Imagery/NIR_05.06.2015.tiff")

# Date 2015 -2
Red.2015.2 <- raster("SAT Imagery/Red_07.07.2015.tiff")
NIR.2015.2 <- raster("SAT Imagery/NIR_07.07.2015.tiff")

# Date 2016 -1
Red.2016.1 <- raster("SAT Imagery/Red_07.06.2016.tiff")
NIR.2016.1 <- raster("SAT Imagery/NIR_07.06.2016.tiff")

# Date 2016 -2
Red.2016.2 <- raster("SAT Imagery/Red_23.06.2016.tiff")
NIR.2016.2 <- raster("SAT Imagery/NIR_23.06.2016.tiff")

# Date 2017 -1
Red.2017.1 <- raster("SAT Imagery/Red_10.06.2017.tiff")
NIR.2017.1 <- raster("SAT Imagery/NIR_10.06.2017.tiff")

# Date 2017 -2
Red.2017.2 <- raster("SAT Imagery/Red_26.06.2017.tiff")
NIR.2017.2 <- raster("SAT Imagery/NIR_26.06.2017.tiff")

# Date 2018 -1
Red.2018.1 <- raster("SAT Imagery/Red_22.05.2018.tiff")
NIR.2018.1 <- raster("SAT Imagery/NIR_22.05.2018.tiff")

# Date 2018 -2
Red.2018.2 <- raster("SAT Imagery/Red_03.06.2018.tiff")
NIR.2018.2 <- raster("SAT Imagery/NIR_03.06.2018.tiff")

# Date 2019 -1
Red.2019.1 <- raster("SAT Imagery/Red_14.04.2019.tiff")
NIR.2019.1 <- raster("SAT Imagery/NIR_14.04.2019.tiff")

# Date 2019 -2
Red.2019.2 <- raster("SAT Imagery/Red_27.04.2019.tiff")
NIR.2019.2 <- raster("SAT Imagery/NIR_27.04.2019.tiff")
```

## 2.2 Estimate NDVI 

This section estimates NDVI considering that Landsat-8 satellite B4 corresponds to Red wave length and B5 to NIR wave length; Sentinel-2 satellite B4 corresponds to Red wave length and B8 to NIR wave length. Data is in raster format with a spatial resolution of 30x30m (Landsat-8) or 10x10m (Sentinel-2).

```{r}
# NDVI 2015
NDVI_2015.1 <- (NIR.2015.1 - Red.2015.1) / (NIR.2015.1 + Red.2015.1)
NDVI_2015.2 <- (NIR.2015.2 - Red.2015.2) / (NIR.2015.2 + Red.2015.2)

# NDVI 2016
NDVI_2016.1 <- (NIR.2016.1 - Red.2016.1) / (NIR.2016.1 + Red.2016.1)
NDVI_2016.2 <- (NIR.2016.2 - Red.2016.2) / (NIR.2016.2 + Red.2016.2)

# NDVI 2017
NDVI_2017.1 <- (NIR.2017.1 - Red.2017.1) / (NIR.2017.1 + Red.2017.1)
NDVI_2017.2 <- (NIR.2017.2 - Red.2017.2) / (NIR.2017.2 + Red.2017.2)

# NDVI 2018
NDVI_2018.1 <- (NIR.2018.1 - Red.2018.1) / (NIR.2018.1 + Red.2018.1)
NDVI_2018.2 <- (NIR.2018.2 - Red.2018.2) / (NIR.2018.2 + Red.2018.2)

# NDVI 2019
NDVI_2019.1 <- (NIR.2019.1 - Red.2019.1) / (NIR.2019.1 + Red.2019.1)
NDVI_2019.2 <- (NIR.2019.2 - Red.2019.2) / (NIR.2019.2 + Red.2019.2)
```
