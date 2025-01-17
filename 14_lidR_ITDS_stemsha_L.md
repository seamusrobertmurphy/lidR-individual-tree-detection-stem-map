Script Draft: ‘stems_grids_2020125’
================
LQ-SRM
22/02/2022

-   [Action](#action)
-   [1 Import normalized LAS data](#import-normalized-las-data)
-   [2 Individual Tree Detection](#individual-tree-detection)
-   [3 Individual Tree Segmentation
    (treeID-proofed)](#individual-tree-segmentation-treeid-proofed)
-   [4 Derive ‘stemsha_L’ Predictor From Segmented Tree Count: ITDS
    Route](#derive-stemsha_l-predictor-from-segmented-tree-count-itds-route)
-   [5 Derive ‘stemsha_L’ Predictor from CHMs: Raster Resolutions
    Route](#derive-stemsha_l-predictor-from-chms-raster-resolutions-route)
    -   [5.1 Variable Window Size
        Functions](#variable-window-size-functions)
    -   [5.2 Custom Function for 95th% Tree Metrics
        Extraction](#custom-function-for-95th-tree-metrics-extraction)
        -   [5.2.1 Stem Mapping](#stem-mapping)

## Action

This following includes R Markdown documentation of steps to derive
stems/ha predictor from LiDAR point cloud for the TCC-EFI predictive
ecosystem model. This pipeline was adopted from the original script
shared by LQuan and reformatted to use inputs from the normalized,
cleaned and classified point cloud produced in previous deliverable
[‘13_lidR_PointCloud_Processing’](https://github.com/seamusrobertmurphy/13_lidR_PointCloud_Processing.git).All
package lists, markdown outputs, and virtual environment settings were
stored in the github repository titled
[‘14_lidR_ITDS_stemha_L’](https://github.com/seamusrobertmurphy/14_lidR_ITDS_stemsha_L.git).

# 1 Import normalized LAS data

Since, the output from previous deliverable ‘13_lidR_Processing’ that is
needed here is in point-form, there are challenges with how to share the
full collection remotely due to file size and internet connection here.
Instead, I’ve saved all system files from that pipeline environment in a
.RData file in the U:drive folder ‘EFI Project - SRM 2021-2022’ titled
‘lidar-processing-pipeline.RData’. File paths to ‘AOI’ in the original
script were replaced with the normalized LAScatolog
‘las_ctg_ahbau_csf_sor_norm’ and normalized tile
’las_tile_ahbau_csf_sor_norm. In effort to get a sample ready for you
meeting tomorrow, I’ve tested this pipeline with the normalized tile
chunk only and rendered in 2D using base plot functions. This tile chunk
can also be processed, classified, and normalized quickly enough on your
end with functions below which were copied and pasted from the previous
output. Make sure to unnormalize if using the .RData files.

``` r
AOI<-"Meldrum"
AOIpath<-paste0("H:\\",AOI,"\\")
GRIDS<-paste0(AOIpath,"GRIDS\\")
TREES<-paste0(AOIpath,"TREES\\")
```

…replaced with:

``` r
defaultCRS<-CRS("+init=EPSG:3153")
las_tile_ahbau = readLAS("./Data/Ahbau/Las_v12_ASPRS/093g030122ne.las", select = 'xyzr')
las_tile_ahbau = filter_duplicates(las_tile_ahbau)
las_tile_ahbau_csf = classify_ground(las_tile_ahbau, csf(sloop_smooth=TRUE, 0.5, 1))
las_tile_ahbau_csf_so = classify_noise(las_tile_ahbau_csf, sor(k=10, m=3))
```

    ## Metrics computation: [=-------------------------------------------------] 3% (1 threads)Metrics computation: [==------------------------------------------------] 4% (1 threads)Metrics computation: [==------------------------------------------------] 5% (1 threads)Metrics computation: [===-----------------------------------------------] 6% (1 threads)Metrics computation: [===-----------------------------------------------] 7% (1 threads)Metrics computation: [====----------------------------------------------] 8% (1 threads)Metrics computation: [====----------------------------------------------] 9% (1 threads)Metrics computation: [=====---------------------------------------------] 10% (1 threads)Metrics computation: [=====---------------------------------------------] 11% (1 threads)Metrics computation: [======--------------------------------------------] 12% (1 threads)Metrics computation: [======--------------------------------------------] 13% (1 threads)Metrics computation: [=======-------------------------------------------] 14% (1 threads)Metrics computation: [=======-------------------------------------------] 15% (1 threads)Metrics computation: [========------------------------------------------] 16% (1 threads)Metrics computation: [========------------------------------------------] 17% (1 threads)Metrics computation: [=========-----------------------------------------] 18% (1 threads)Metrics computation: [=========-----------------------------------------] 19% (1 threads)Metrics computation: [==========----------------------------------------] 20% (1 threads)Metrics computation: [==========----------------------------------------] 21% (1 threads)Metrics computation: [===========---------------------------------------] 22% (1 threads)Metrics computation: [===========---------------------------------------] 23% (1 threads)Metrics computation: [============--------------------------------------] 24% (1 threads)Metrics computation: [============--------------------------------------] 25% (1 threads)Metrics computation: [=============-------------------------------------] 26% (1 threads)Metrics computation: [=============-------------------------------------] 27% (1 threads)Metrics computation: [==============------------------------------------] 28% (1 threads)Metrics computation: [==============------------------------------------] 29% (1 threads)Metrics computation: [===============-----------------------------------] 30% (1 threads)Metrics computation: [===============-----------------------------------] 31% (1 threads)Metrics computation: [================----------------------------------] 32% (1 threads)Metrics computation: [================----------------------------------] 33% (1 threads)Metrics computation: [=================---------------------------------] 34% (1 threads)Metrics computation: [=================---------------------------------] 35% (1 threads)Metrics computation: [==================--------------------------------] 36% (1 threads)Metrics computation: [==================--------------------------------] 37% (1 threads)Metrics computation: [===================-------------------------------] 38% (1 threads)Metrics computation: [===================-------------------------------] 39% (1 threads)Metrics computation: [====================------------------------------] 40% (1 threads)Metrics computation: [====================------------------------------] 41% (1 threads)Metrics computation: [=====================-----------------------------] 42% (1 threads)Metrics computation: [=====================-----------------------------] 43% (1 threads)Metrics computation: [======================----------------------------] 44% (1 threads)Metrics computation: [======================----------------------------] 45% (1 threads)Metrics computation: [=======================---------------------------] 46% (1 threads)Metrics computation: [=======================---------------------------] 47% (1 threads)Metrics computation: [========================--------------------------] 48% (1 threads)Metrics computation: [========================--------------------------] 49% (1 threads)Metrics computation: [=========================-------------------------] 50% (1 threads)Metrics computation: [=========================-------------------------] 51% (1 threads)Metrics computation: [==========================------------------------] 52% (1 threads)Metrics computation: [==========================------------------------] 53% (1 threads)Metrics computation: [===========================-----------------------] 54% (1 threads)Metrics computation: [===========================-----------------------] 55% (1 threads)Metrics computation: [============================----------------------] 56% (1 threads)Metrics computation: [============================----------------------] 57% (1 threads)Metrics computation: [=============================---------------------] 58% (1 threads)Metrics computation: [=============================---------------------] 59% (1 threads)Metrics computation: [==============================--------------------] 60% (1 threads)Metrics computation: [==============================--------------------] 61% (1 threads)Metrics computation: [===============================-------------------] 62% (1 threads)Metrics computation: [===============================-------------------] 63% (1 threads)Metrics computation: [================================------------------] 64% (1 threads)Metrics computation: [================================------------------] 65% (1 threads)Metrics computation: [=================================-----------------] 66% (1 threads)Metrics computation: [=================================-----------------] 67% (1 threads)Metrics computation: [==================================----------------] 68% (1 threads)Metrics computation: [==================================----------------] 69% (1 threads)Metrics computation: [===================================---------------] 70% (1 threads)Metrics computation: [===================================---------------] 71% (1 threads)Metrics computation: [====================================--------------] 72% (1 threads)Metrics computation: [====================================--------------] 73% (1 threads)Metrics computation: [=====================================-------------] 74% (1 threads)Metrics computation: [=====================================-------------] 75% (1 threads)Metrics computation: [======================================------------] 76% (1 threads)Metrics computation: [======================================------------] 77% (1 threads)Metrics computation: [=======================================-----------] 78% (1 threads)Metrics computation: [=======================================-----------] 79% (1 threads)Metrics computation: [========================================----------] 80% (1 threads)Metrics computation: [========================================----------] 81% (1 threads)Metrics computation: [=========================================---------] 82% (1 threads)Metrics computation: [=========================================---------] 83% (1 threads)Metrics computation: [==========================================--------] 84% (1 threads)Metrics computation: [==========================================--------] 85% (1 threads)Metrics computation: [===========================================-------] 86% (1 threads)Metrics computation: [===========================================-------] 87% (1 threads)Metrics computation: [============================================------] 88% (1 threads)Metrics computation: [============================================------] 89% (1 threads)Metrics computation: [=============================================-----] 90% (1 threads)Metrics computation: [=============================================-----] 91% (1 threads)Metrics computation: [==============================================----] 92% (1 threads)Metrics computation: [==============================================----] 93% (1 threads)Metrics computation: [===============================================---] 94% (1 threads)Metrics computation: [===============================================---] 95% (1 threads)Metrics computation: [================================================--] 96% (1 threads)Metrics computation: [================================================--] 97% (1 threads)Metrics computation: [=================================================-] 98% (1 threads)Metrics computation: [=================================================-] 99% (1 threads)Metrics computation: [==================================================] 100% (1 threads)

``` r
las_tile_ahbau_csf_sor = filter_poi(las_tile_ahbau_csf_so, Classification != LASNOISE)
las_tile_ahbau_csf_sor_norm = normalize_height(las_tile_ahbau_csf_sor, knnidw())
```

    ## Inverse distance weighting: [=====---------------------------------------------] 10% (1 threads)Inverse distance weighting: [=====---------------------------------------------] 11% (1 threads)Inverse distance weighting: [======--------------------------------------------] 12% (1 threads)Inverse distance weighting: [======--------------------------------------------] 13% (1 threads)Inverse distance weighting: [=======-------------------------------------------] 14% (1 threads)Inverse distance weighting: [=======-------------------------------------------] 15% (1 threads)Inverse distance weighting: [========------------------------------------------] 16% (1 threads)Inverse distance weighting: [========------------------------------------------] 17% (1 threads)Inverse distance weighting: [=========-----------------------------------------] 18% (1 threads)Inverse distance weighting: [=========-----------------------------------------] 19% (1 threads)Inverse distance weighting: [==========----------------------------------------] 20% (1 threads)Inverse distance weighting: [==========----------------------------------------] 21% (1 threads)Inverse distance weighting: [===========---------------------------------------] 22% (1 threads)Inverse distance weighting: [===========---------------------------------------] 23% (1 threads)Inverse distance weighting: [============--------------------------------------] 24% (1 threads)Inverse distance weighting: [============--------------------------------------] 25% (1 threads)Inverse distance weighting: [=============-------------------------------------] 26% (1 threads)Inverse distance weighting: [=============-------------------------------------] 27% (1 threads)Inverse distance weighting: [==============------------------------------------] 28% (1 threads)Inverse distance weighting: [==============------------------------------------] 29% (1 threads)Inverse distance weighting: [===============-----------------------------------] 30% (1 threads)Inverse distance weighting: [===============-----------------------------------] 31% (1 threads)Inverse distance weighting: [================----------------------------------] 32% (1 threads)Inverse distance weighting: [================----------------------------------] 33% (1 threads)Inverse distance weighting: [=================---------------------------------] 34% (1 threads)Inverse distance weighting: [=================---------------------------------] 35% (1 threads)Inverse distance weighting: [==================--------------------------------] 36% (1 threads)Inverse distance weighting: [==================--------------------------------] 37% (1 threads)Inverse distance weighting: [===================-------------------------------] 38% (1 threads)Inverse distance weighting: [===================-------------------------------] 39% (1 threads)Inverse distance weighting: [====================------------------------------] 40% (1 threads)Inverse distance weighting: [====================------------------------------] 41% (1 threads)Inverse distance weighting: [=====================-----------------------------] 42% (1 threads)Inverse distance weighting: [=====================-----------------------------] 43% (1 threads)Inverse distance weighting: [======================----------------------------] 44% (1 threads)Inverse distance weighting: [======================----------------------------] 45% (1 threads)Inverse distance weighting: [=======================---------------------------] 46% (1 threads)Inverse distance weighting: [=======================---------------------------] 47% (1 threads)Inverse distance weighting: [========================--------------------------] 48% (1 threads)Inverse distance weighting: [========================--------------------------] 49% (1 threads)Inverse distance weighting: [=========================-------------------------] 50% (1 threads)Inverse distance weighting: [=========================-------------------------] 51% (1 threads)Inverse distance weighting: [==========================------------------------] 52% (1 threads)Inverse distance weighting: [==========================------------------------] 53% (1 threads)Inverse distance weighting: [===========================-----------------------] 54% (1 threads)Inverse distance weighting: [===========================-----------------------] 55% (1 threads)Inverse distance weighting: [============================----------------------] 56% (1 threads)Inverse distance weighting: [============================----------------------] 57% (1 threads)Inverse distance weighting: [=============================---------------------] 58% (1 threads)Inverse distance weighting: [=============================---------------------] 59% (1 threads)Inverse distance weighting: [==============================--------------------] 60% (1 threads)Inverse distance weighting: [==============================--------------------] 61% (1 threads)Inverse distance weighting: [===============================-------------------] 62% (1 threads)Inverse distance weighting: [===============================-------------------] 63% (1 threads)Inverse distance weighting: [================================------------------] 64% (1 threads)Inverse distance weighting: [================================------------------] 65% (1 threads)Inverse distance weighting: [=================================-----------------] 66% (1 threads)Inverse distance weighting: [=================================-----------------] 67% (1 threads)Inverse distance weighting: [==================================----------------] 68% (1 threads)Inverse distance weighting: [==================================----------------] 69% (1 threads)Inverse distance weighting: [===================================---------------] 70% (1 threads)Inverse distance weighting: [===================================---------------] 71% (1 threads)Inverse distance weighting: [====================================--------------] 72% (1 threads)Inverse distance weighting: [====================================--------------] 73% (1 threads)Inverse distance weighting: [=====================================-------------] 74% (1 threads)Inverse distance weighting: [=====================================-------------] 75% (1 threads)Inverse distance weighting: [======================================------------] 76% (1 threads)Inverse distance weighting: [======================================------------] 77% (1 threads)Inverse distance weighting: [=======================================-----------] 78% (1 threads)Inverse distance weighting: [=======================================-----------] 79% (1 threads)Inverse distance weighting: [========================================----------] 80% (1 threads)Inverse distance weighting: [========================================----------] 81% (1 threads)Inverse distance weighting: [=========================================---------] 82% (1 threads)Inverse distance weighting: [=========================================---------] 83% (1 threads)Inverse distance weighting: [==========================================--------] 84% (1 threads)Inverse distance weighting: [==========================================--------] 85% (1 threads)Inverse distance weighting: [===========================================-------] 86% (1 threads)Inverse distance weighting: [===========================================-------] 87% (1 threads)Inverse distance weighting: [============================================------] 88% (1 threads)Inverse distance weighting: [============================================------] 89% (1 threads)Inverse distance weighting: [=============================================-----] 90% (1 threads)Inverse distance weighting: [=============================================-----] 91% (1 threads)Inverse distance weighting: [==============================================----] 92% (1 threads)Inverse distance weighting: [==============================================----] 93% (1 threads)Inverse distance weighting: [===============================================---] 94% (1 threads)Inverse distance weighting: [===============================================---] 95% (1 threads)Inverse distance weighting: [================================================--] 96% (1 threads)Inverse distance weighting: [================================================--] 97% (1 threads)Inverse distance weighting: [=================================================-] 98% (1 threads)Inverse distance weighting: [=================================================-] 99% (1 threads)Inverse distance weighting: [==================================================] 100% (1 threads)

``` r
# plot normalization using derived chm
las_tile_ahbau_chm = grid_canopy(las_tile_ahbau_csf_sor_norm, 1, dsmtin(8))
plot(las_tile_ahbau_chm, col = height.colors(50))
```

![](14_lidR_ITDS_stemsha_L_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

# 2 Individual Tree Detection

To avoid replicated trees across adjacent tiles, the catalog engine
‘uniqueness’ function was applied to the local maxima algorithm. This
was fitted with your custom window function ‘wf’, which is also plotted
below with your adopted code chunks. Super interesting stuff. For
visualization purposes, the detected crown points were plotted as
‘sp\>sf’ object over chm raster.

``` r
# window function (Quan, 2022)
wf_Quan<-function(x){ #so far these parameters are best will try 6m floor. 
  a=0.179-0.1
  b=0.51+0.5 #Go big but not 2.49 big. maybe increase slope a hair.
  y<-a*x+b #y[x<15]=2.2
  return(y)}
heights <- seq(0,40,0.5)
window <- wf_Quan(heights)
plot(heights, window, type = "l",  ylim = c(0,12), xlab="point elevation (m)", ylab="window diameter (m)")

# find trees
las_tile_ahbau_ttops <- find_trees(las_tile_ahbau_csf_sor_norm, lmf(wf_Quan), uniqueness = "bitmerge")
las_tile_ahbau_ttops_sf = st_as_sf(las_tile_ahbau_ttops)
proj4string(las_tile_ahbau_ttops) <- defaultCRS
proj4string(las_tile_ahbau_chm) <- defaultCRS
st_crs(las_tile_ahbau_ttops_sf) = defaultCRS

mypalette<-brewer.pal(8,"Greens")
plot(las_tile_ahbau_chm, col = mypalette, alpha=0.6)
plot(st_geometry(las_tile_ahbau_ttops_sf["treeID"]), add=TRUE, cex = 0.001, pch=19, col = 'red', alpha=0.8)
```

![](Data/treeI.png)

![](Data/windowFunction_Quan2022.png)

# 3 Individual Tree Segmentation (treeID-proofed)

Not 100% sure, but from partial reading of the lidR bookdown, it looks
like Romaine is suggesting that because of tile buffering needed in the
engine processing, there is need to segment the crowns to make sure
trees are not double counted by multiple tiles. He notes this is because
the engine was designed for batch processing. (Romaine, 2022:
[14.12](https://r-lidar.github.io/lidRbook/engine.html#engine-its)).

FYI, Ive been using an older lidR package version (3.2.3) and some
deprecated function names here which need replacing for the new LidR
4.0.0.

``` r
algo = dalponte2016(las_tile_ahbau_chm, las_tile_ahbau_ttops)
las_tile_ahbau_segmented = segment_trees(las_tile_ahbau_csf_sor_norm, algo)
las_tile_ahbau_segmented_crowns = delineate_crowns(las_tile_ahbau_segmented, type = 'convex')
las_tile_ahbau_segmented_crowns_spplot = sp::plot(las_tile_ahbau_segmented_crowns, cex=0.000001, axes = TRUE, alpha=0.1)
```

![](Data/treeTops_crowns_spPolygons.png)

# 4 Derive ‘stemsha_L’ Predictor From Segmented Tree Count: ITDS Route

``` r
#sp::spTransform(las_tile_ahbau_segmented_crowns) = defaultCRS
las_tile_ahbau_segmented_crowns_sf = st_as_sf(las_tile_ahbau_segmented_crowns, coords = c("XTOP", "YTOP"), st_crs=defaultCRS)
psych::describe(las_tile_ahbau_segmented_crowns_sf$treeID)

raster_template = rast(ext(las_tile_ahbau_chm), resolution = 20, crs = st_crs(las_tile_ahbau_chm)$wkt)
las_tile_ahbau_ttops_rast = rasterize(vect(las_tile_ahbau_ttops_sf), raster_template, fun = sum, touches = TRUE)
stemsha_L_rast = las_tile_ahbau_ttops_rast*5
crs(stemsha_L_rast) = defaultCRS
plot(stemsha_L_rast)
stemsha_L = raster::raster(stemsha_L_rast, filename = "./Data/stemsha_L.tif")
stemsha_L = writeRaster(stemsha_L, filename = "./Data/stemsha_L.tif", overwrite=TRUE)
plot(stemsha_L)
```

![](Data/stemsha_L_raster.png)

# 5 Derive ‘stemsha_L’ Predictor from CHMs: Raster Resolutions Route

``` r
# Point-to-raster 2 resolutions
ttops_chm_p2r_05 <- locate_trees(chm_p2r_05, lmf(5))

las_tile_ahbau_chm = grid_canopy(las_tile_ahbau_csf_sor_norm, 1, dsmtin(8))
plot(lead_htop_raster, col = height.colors(50))

chm_p2r_05 <- rasterize_canopy(las, 0.5, p2r(subcircle = 0.2), pkg = "terra")
chm_p2r_1 <- rasterize_canopy(las, 1, p2r(subcircle = 0.2), pkg = "terra")

# Pitfree with and without subcircle tweak
chm_pitfree_05_1 <- rasterize_canopy(las, 0.5, pitfree(), pkg = "terra")
chm_pitfree_05_2 <- rasterize_canopy(las, 0.5, pitfree(subcircle = 0.2), pkg = "terra")

# Post-processing median filter
kernel <- matrix(1,3,3)
chm_p2r_05_smoothed <- terra::focal(chm_p2r_05, w = kernel, fun = median, na.rm = TRUE)
chm_p2r_1_smoothed <- terra::focal(chm_p2r_1, w = kernel, fun = median, na.rm = TRUE)
```

``` r
###########################
# chm to stem map pipeline
lead_htop = rast("./Data/Raster_Covariates/lead_htop_raster.tif")
elev = rast("./Data/Raster_Covariates/elev_raster.tif")
elev = mask(elev, vect(aoi_sf))
lead_htop = mask(lead_htop, vect(aoi_sf))
lead_htop[lead_htop < 1.3] <- NA
elev = mask(elev, lead_htop, inverse=FALSE)
plot(lead_htop, main = "canopy height (m)")
plot(elev, main = "elevation")
```

Using terra aggregate function with listed loops we derived a stem map
drawn from a cell grid of 1m resolution that was post-processed with
logistic median smoothing filter. Aggregation sampling used in this
terra function applies a simple pattern of arithmetic from top left to
right and so depends on the divisional fraction of total cell numbers
whether exact multiplication is returned or numbers are rounded down
with ommissions, as was seen in the Gaspard analysis below (ncells:
1080\>330). For comparison, a second aggreation was tested with centroid
based sampling using a polygon overlaid on top of the cells along
represented by block dots. From this neighbourhoods, centroid values
were extracted to a polygon and assessed using histograms before 95th
percentile reassigned as canopy height layer and stem detection input.

## 5.1 Variable Window Size Functions

``` r
# window function (Quan, 2022)
wf_Quan<-function(x){ 
  a=0.179-0.1
  b=0.51+0.5 
  y<-a*x+b 
  return(y)}
# window function (Plowright, 2017)  
# Plowright, A. (2015). Extracting trees in an urban environment using airborne LiDAR. 
# Doctoral dissertation, University of British Columbia.
wf_Plowright<-function(x){ # floor reccomended at 2 and then 1.5m
  a=0.05
  b=0.6 #Go big but not 2.49 big. maybe increase slope a hair.
  y<-a*x+b #y[x<15]=2.2
  return(y)}
# window function (Pitkänen, et al., 2004) - Spruce + Scots Pine (lsmedian) : 1.26 0.16
heights <- seq(0,40,0.5)
window_Quan <- wf_Quan(heights)
window_Plowright <- wf_Plowright(heights)
plot(heights, window_Quan, type = "l",  
     ylim = c(0,12), xlab="point elevation (m)", ylab="window diameter (m)", main='project allometry')
```

![](14_lidR_ITDS_stemsha_L_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
plot(heights, window_Plowright, type = "l",  
     ylim = c(0,12), xlab="point elevation (m)", ylab="window diameter (m)", main='plowright')
```

![](14_lidR_ITDS_stemsha_L_files/figure-gfm/unnamed-chunk-8-2.png)<!-- -->

## 5.2 Custom Function for 95th% Tree Metrics Extraction

``` r
# custom function writing
quant95 <- function(x, ...) quantile(x, c(.95), na.rm = TRUE)
custFuns <- list(quant95, max)
names(custFuns) <- c("95thQuantile", "Max")
```

### 5.2.1 Stem Mapping

``` r
#raster post-process smoothing
kernel <- matrix(1,3,3)
terra::focal(lead_htop_rast, w = kernel, fun = median, na.rm = TRUE)
lead_htop_raster = raster(lead_htop)
#foretTools pipeline
ttops_6m_Quan = vwf(CHM = lead_htop_raster, winFun = wf_Quan, minHeight = 6)
ttops_6m_Plowright = vwf(CHM = lead_htop_raster, winFun = wf_Plowright, minHeight = 6)
ttops_6m_wf5 = vwf(CHM = lead_htop_raster, winFun = 5, minHeight = 6)
ttops_2m_Quan = vwf(CHM = lead_htop_raster, winFun = wf_Quan, minHeight = 2)
ttops_2m_Plowright = vwf(CHM = lead_htop_raster, winFun = wf_Plowright, minHeight = 2)
# window size mining
height_crown_cor = lm(ttops_6m_Quan$height ~ ttops_6m_Quan$winRadius)
height_crown_cor 

# custom functions insert here
sp_summarise(ttops_6m_Quan, areas = vri_aoi_sf, variables = "height", statFuns = custFuns)
sp_summarise(ttops_6m_Quan, areas = vri_aoi_sf, statFuns = custFuns) #treecount

# rasterized results at 50m resolution
gridStats <- sp_summarise(ttops_6m_Quan = ttops, grid = 50, variables = "height")

#lidr pipeline
ttops_6m_Quan = lidR::find_trees(lead_htop_raster[lead_htop_raster>6], lmf(wf_Quan))
ttops_6m_Plowright = lidR::find_trees(lead_htop_raster[lead_htop_raster>6], lmf(wf_Plowright))
stemsha_L_sf = st_as_sf(ttops_6m_Quan)
stemsha_L_raster = raster::raster(stemsha_L_sf)
# visualize
mypalette<-brewer.pal(8,"Greens")
plot(lead_htop_raster, col = mypalette, alpha=0.6)
plot(st_geometry(stemsha_L["treeID"]), add=TRUE, cex = 0.001, pch=19, col = 'red', alpha=0.8)
writeRaster(stemsha_L_raster, filename = "./Data/Raster_Covariates/stemsha_L_raster.tif", overwrite=TRUE)
plot(stemsha_L_raster)
```

\`\`\`
