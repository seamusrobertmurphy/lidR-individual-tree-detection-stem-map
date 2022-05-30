# 14_lidR_ITDS_stemsha_L


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
