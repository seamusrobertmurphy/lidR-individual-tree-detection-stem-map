Script Draft: ‘stems_grids_2020125’
================
LQ-SRM
22/02/2022

-   [Action](#action)
-   [1 Import LAScatalog and LAS tile](#import-lascatalog-and-las-tile)

## Action

This following includes R Markdown documentation of steps to derive
stems/ha predictor from LiDAR point cloud for the TCC-EFI predictive
ecosystem model. This pipeline was adopted from the original script
shared by LQuan and reformatted to use inputs from the normalized,
cleaned and classified point cloud produced in previous deliverable
[‘13_lidR_PointCloud_Processing’](https://github.com/seamusrobertmurphy/13_lidR_PointCloud_Processing.git).All
package lists, markdown outputs, and virtual environment settings were
stored in the github repository titled
[‘14_lidR_stemha_L’](https://github.com/seamusrobertmurphy/14_lidR_ITDS_stemsha_L.git).

# 1 Import LAScatalog and LAS tile

Since, the output from previous deliverable ‘13_lidR_Processing’ that is
needed here is in point-form, there are challenges with how to share the
full collection remotely due to file size and internet connection here.
Instead, I’ve saved all system files from that pipeline environment in a
.RData file in the U:drive folder ‘EFI Project - SRM 2021-2022’ titled
‘lidar-processing-pipeline.RData’. File paths to ‘AOI’ in the original
script were replaced with the normalized LAScatolog
‘las_ctg_ahbau_csf_sor_norm’ and normalized tile
’las_tile_ahbau_csf_sor_norm. Thgough for sake of speed, I’ve tested
this pipeline with the normalized tile only. The tile chunk can also be
processed, classified, and normalized quickly enough on your end with
processing steps below which were copied and pasted from the previous
output. Make sure to unnormalize if using the RData files. For
rendering, see required packages above (i.e. rgl, webshot…)

``` r
AOI<-"Meldrum"
AOIpath<-paste0("H:\\",AOI,"\\")
GRIDS<-paste0(AOIpath,"GRIDS\\")
TREES<-paste0(AOIpath,"TREES\\")
defaultCRS<-CRS("+init=EPSG:3153")
```

…replaced with:

``` r
las_tile_ahbau = readLAS("./Data/Ahbau/Las_v12_ASPRS/093g030122ne.las", select = 'xyzri')
las_tile_ahbau = filter_duplicates(las_tile_ahbau)
las_check(las_tile_ahbau)
las_tile_ahbau_csf = classify_ground(las_tile_ahbau, csf(sloop_smooth=TRUE, 0.5, 1))
las_tile_ahbau_csf_so = classify_noise(las_tile_ahbau_csf, sor(k=10, m=3))
las_tile_ahbau_csf_sor = filter_poi(las_tile_ahbau_csf_so, Classification != LASNOISE)
las_tile_ahbau_csf_sor_norm = normalize_height(las_tile_ahbau_csf_sor, knnidw())

plot(las_tile_ahbau, bg = "white")
plot(las_tile_ahbau_csf, color = "Classification", bg = "white") 
plot(las_tile_ahbau_csf_so, color = "Classification", bg = "white") 
# plot normalization using derived chm
las_tile_ahbau_chm = grid_canopy(las_tile_ahbau_csf_sor_norm, 1, dsmtin(8))
plot(las_tile_ahbau_chm, col = height.colors(50))
```
