Import and organize the data
----------------------------

``` r
#Install and load any necessary packages
#install.packages("fields")
#install.packages("geoR")
library(fields)
```

    ## Loading required package: spam

    ## Loading required package: dotCall64

    ## Loading required package: grid

    ## Spam version 2.2-0 (2018-06-19) is loaded.
    ## Type 'help( Spam)' or 'demo( spam)' for a short introduction 
    ## and overview of this package.
    ## Help for individual functions is also obtained by adding the
    ## suffix '.spam' to the function name, e.g. 'help( chol.spam)'.

    ## 
    ## Attaching package: 'spam'

    ## The following objects are masked from 'package:base':
    ## 
    ##     backsolve, forwardsolve

    ## Loading required package: maps

    ## See www.image.ucar.edu/~nychka/Fields for
    ##  a vignette and other supplements.

``` r
library(geoR)
```

    ## --------------------------------------------------------------
    ##  Analysis of Geostatistical Data
    ##  For an Introduction to geoR go to http://www.leg.ufpr.br/geoR
    ##  geoR version 1.7-5.2.1 (built on 2016-05-02) is now loaded
    ## --------------------------------------------------------------

``` r
#Load the data
ppra.2015 <- read.csv("2015_master.csv")
ppra.2014 <- read.csv("2014_master.csv")
```

Convert data to geodata Apply spatial information to the variables, as type "geodata" This is a bit tricky, and specific for these data sets, but they work

``` r
geo.list.2015 <- NULL
geo.list.2014a <- NULL
geo.list.2014b <- NULL
row.z <- 0
row.zz <- 0
row.zzz <- 0
z <- c(7:58) 
zz <- c(7:35)
zzz <- c(36:45)
for (i in z){
  row.z <- row.z + 1
  geo.list.2015[[row.z]] <- as.geodata(ppra.2015[1:96,], data.col = i)
}
```

    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data
    ## as.geodata: 8 points removed due to NA in the data

``` r
geo.list.2015[[53]] <- as.geodata(ppra.2015[97:144,], data.col = 59)
for (i in zz){
  row.zz <- row.zz + 1
  geo.list.2014a[[row.zz]] <- as.geodata(ppra.2014[1:96,], data.col = i)
}
for (i in zzz){
  row.zzz <- row.zzz + 1
  geo.list.2014b[[row.zzz]] <- as.geodata(ppra.2014[97:120,], data.col = i)
}
#Give the variables their appropriate names
names(geo.list.2015) <- colnames(ppra.2015[7:59])
names(geo.list.2014a) <- colnames(ppra.2014[7:35])
names(geo.list.2014b) <- colnames(ppra.2014[36:45])
```

Now, all variables can be read as geodata from geo.list$variable

Next, pick the colors that you want to plot with, and create a custom color palette

``` r
color.pal <- c("#006600","#FFFFFF")
color.func <- colorRampPalette(color.pal)
```

Spatial plots of raw data
-------------------------

I'm looking at pre-Fv, pre-SCN eggs, R5 disease index, and yield First, load the raw data

``` r
#Pre.ratio.2015 is currently in ng Fv DNA / ng total DNA
Pre.ratio.2015 <- matrix(ppra.2015$Pre.ratio[1:96], ncol = 6, nrow = 16)
#Convert Pre.ratio.2015 into fg Fv DNA / ng total DNA by * 1,000,000
Pre.ratio.2015 <- Pre.ratio.2015 * 1000000
Pre.SCN.eggs.2015 <- matrix(ppra.2015$PreSCN.eggs[1:96], ncol = 6, nrow = 16)
R5.DX.2015 <- matrix(ppra.2015$R5.DX[1:96], ncol = 6, nrow = 16)
Yield.2015 <- matrix(ppra.2015$Yield[97:144], ncol = 6, nrow = 8)

# This heatmap function transposes the data first, for formatting sake
# Then it plots the raw data in base R. 

# Note: Yield needs a different (very similar) function due to the # of data points
heatmap.t <- function(var){
  Pass <- 1:9
  Range <- 1:9
  Passlab <- c("15", "16", "17")
  Rangelab <- c("21", "22", "23", "24", "25", "26", "27", "28")
  par(mar=c(5.1, 4.1, 2.1, 5.5))
  plot(Pass,Range, xaxt = "n", yaxt="n", xaxs = "i", pch=NA, title(""),
       cex.lab = 1.5)
  axis(side=2, at=seq(1.25,9,1.075), labels = Rangelab)
  axis(side=1, at=seq(2.25,9,2.75), labels = Passlab)
  y <- c(1.45, 3.25, 4.10, 5.92, 6.77, 8.55)
  for (i in seq_along(y)){
    tvar <- t(var)
    tvarcol <- matrix(tvar[i,], nrow = 1, ncol = 16)
    add.image(y[i], 5, tvarcol, 
              zlim=c(min(tvar), max(tvar)), axes=FALSE,
              image.width = 1, image.height = 0.1, 
              col = color.func(length(tvar)))
#    text(y[i], y = (at = seq(0.95,9,0.535)), 
#          labels = format(round((tvar[i,]), 2), nsmall = 2))
  }
  abline(v=c((11/3), (19/3)))
  abline(a=1.75, b=0)
  abline(a=2.85, b=0)
  abline(a=3.91, b=0)
  abline(a=5.01, b=0)
  abline(a=6.10, b=0)
  abline(a=7.15, b=0)
  abline(a=8.25, b=0)
  image.plot(var, col = color.func(length(tvar)), 
             legend.only = TRUE, zlim=c(min(var), max(var)),
             axes=FALSE, smallplot = c(.85,.9,0.2,0.92))
}
yield_heatmap.t <- function(var){
  Pass <- 1:9
  Range <- 1:9
  Passlab <- c("15", "16", "17")
  Rangelab <- c("21", "22", "23", "24", "25", "26", "27", "28")
  par(mar=c(5.1, 4.1, 2.1, 5.5))
  plot(Pass,Range, xaxt = "n", yaxt="n", xaxs = "i", pch=NA, title(""),
       cex.lab = 1.5)
  axis(side=2, at=seq(1.25,9,1.075), labels = Rangelab)
  axis(side=1, at=seq(2.25,9,2.75), labels = Passlab)
  y <- c(1.65, 2.95, 4.35, 5.65, 7.05, 8.35)
  for (i in seq_along(y)){
    tvar <- t(var)
    tvarcol <- matrix(tvar[i,], nrow = 1, ncol = 8)
    add.image(y[i], 5, tvarcol, 
              zlim=c(min(tvar), max(tvar)), axes=FALSE,
              image.width = 1, image.height = 0.25, 
              col = color.func(length(tvar)))
#    text(y[i], y = (at = seq(0.95,9,0.535)), 
#          labels = format(round((tvar[i,]), 2), nsmall = 2))
  }
  abline(v=c((11/3), (19/3)))
  abline(a=1.75, b=0)
  abline(a=2.85, b=0)
  abline(a=3.91, b=0)
  abline(a=5.01, b=0)
  abline(a=6.10, b=0)
  abline(a=7.15, b=0)
  abline(a=8.25, b=0)
  image.plot(var, col = color.func(length(tvar)), 
             legend.only = TRUE, zlim=c(min(var), max(var)),
             axes=FALSE, smallplot = c(.85,.9,0.2,0.92))
}

# Note2: This is kind of a brute force way of plotting, so the formatting may be off if you try to show one map at a time
par(mfrow=c(2,4))
heatmap.t(Pre.ratio.2015)
title("2015 Pre-planting Fv quantities", cex.main = 1.5)
heatmap.t(Pre.SCN.eggs.2015)
title("2015 Pre-planting SCN eggs", cex.main = 1.5)
heatmap.t(R5.DX.2015)
title("2015 R5 In-field Disease Index", cex.main = 1.5)
yield_heatmap.t(Yield.2015)
title("2015 Yield (bu/acre)", cex.main = 1.5)

#Since we have spatial information though, I'll use simple kriging to interpolate the raw data
# This function separates the coordinates and raw data, then interpolates and plots the "heatmap"
fields.krig <- function(var){
  xy <- as.data.frame(var$coords)
  z <- as.data.frame(var$data)
  # Now Krig the data using Krig
  # Defaults to "stationary.cov" as covariance model 
  var.krig <- Krig(xy, z)
  # Now plot the Kriged data
  surface(var.krig, 
          type = "C", 
          extrap = T,
          col = color.func(length(var$data)),
          xlab = "", ylab = "",
          #cex.lab = 1.8,
          legend.width = 3, legend.lab = "")
}
fields.krig.yield <- function(var){
  xy <- as.data.frame(var$coords)
  z <- as.data.frame(var$data)
  # Now Krig the data using Krig
  # Defaults to "stationary.cov" as covariance model 
  var.krig <- Krig(xy, z)
  # Now plot the Kriged data
  surface(var.krig, 
          type = "C", 
          extrap = T,
          col = color.func(length(var$data)),
          xlab = "", ylab = "",
          #cex.lab = 1.5,
          legend.width = 3, legend.lab = "")
}
geo.list.2015$Pre.ratio$data <- geo.list.2015$Pre.ratio$data * 1000000
fields.krig(geo.list.2015$Pre.ratio)
title("2015 Pre-planting Fv quantities", cex.main = 1.5)
fields.krig(geo.list.2015$PreSCN.eggs)
title("2015 Pre-planting SCN eggs", cex.main = 1.5)
fields.krig(geo.list.2015$R5.DX)
```

    ## Warning: 
    ## Grid searches over lambda (nugget and sill variances) with  minima at the endpoints: 
    ##   (REML) Restricted maximum likelihood 
    ##    minimum at  right endpoint  lambda  =  0.04520064 (eff. df= 91.19999 )

``` r
title("2015 R5 in-field Disease Index", cex.main = 1.5)
fields.krig.yield(geo.list.2015$Yield)
```

    ## Warning: 
    ## Grid searches over lambda (nugget and sill variances) with  minima at the endpoints: 
    ##   (REML) Restricted maximum likelihood 
    ##    minimum at  right endpoint  lambda  =  0.05117423 (eff. df= 45.60001 )

``` r
title("2015 Soybean Yield", cex.main = 1.5)
```

![](Heatmaps_and_correlations_files/figure-markdown_github/Spatial%20plots%20of%202015%20raw%20data-1.png)

Now, repeat with 2014 data

``` r
Pre.ratio.2014 <- matrix(ppra.2014$Pre.ratio[1:96], ncol = 6, nrow = 16)
Pre.ratio.2014 <- Pre.ratio.2014 * 1000000
Pre.SCN.eggs.2014 <- matrix(ppra.2014$PreSCN.eggs[1:96], ncol = 6, nrow = 16)
R5.DX.2014 <- matrix(ppra.2014$R5.DX[97:120], ncol = 3, nrow = 8)
Yield.2014 <- matrix(ppra.2014$Yield[97:120], ncol = 3, nrow = 8)

#Can use the same functions above, but also need this one for R5 DX
heatmap.t2 <- function(var){
  Pass <- 1:9
  Range <- 1:9
  Passlab <- c("15", "16", "17")
  Rangelab <- c("21", "22", "23", "24", "25", "26", "27", "28")
  par(mar=c(5.1, 4.1, 2.1, 5.5))
  plot(Pass,Range, xaxt = "n", yaxt="n", xaxs = "i", pch=NA, title(""),
       cex.lab = 1.5)
  axis(side=2, at=seq(1.25,9,1.075), labels = Rangelab)
  axis(side=1, at=seq(2.25,9,2.75), labels = Passlab)
  y <- c(2.3, 4.95, 7.6)
  for (i in seq_along(y)){
    tvar <- t(var)
    tvarcol <- matrix(tvar[i,], nrow = 1, ncol = 8)
    add.image(y[i], 5, tvarcol, 
              zlim=c(min(tvar), max(tvar)), axes=FALSE,
              image.width = 1, image.height = 0.45, 
              col = color.func(length(tvar)))
#    text(y[i], y = (at = seq(0.95,9,0.535)), 
#          labels = format(round((tvar[i,]), 2), nsmall = 2))
  }
  abline(v=c((11/3), (19/3)))
  abline(a=1.75, b=0)
  abline(a=2.85, b=0)
  abline(a=3.91, b=0)
  abline(a=5.01, b=0)
  abline(a=6.10, b=0)
  abline(a=7.15, b=0)
  abline(a=8.25, b=0)
  image.plot(var, col = color.func(length(tvar)), 
             legend.only = TRUE, zlim=c(min(var), max(var)),
             axes=FALSE, smallplot = c(.85,.9,0.2,0.92))
}
yield_heatmap.t2 <- function(var){
  Pass <- 1:9
  Range <- 1:9
  Passlab <- c("15", "16", "17")
  Rangelab <- c("21", "22", "23", "24", "25", "26", "27", "28")
  par(mar=c(5.1, 4.1, 2.1, 5.5))
  plot(Pass,Range, xaxt = "n", yaxt="n", xaxs = "i", pch=NA, title(""),
       cex.lab = 1.5)
  axis(side=2, at=seq(1.25,9,1.075), labels = Rangelab)
  axis(side=1, at=seq(2.25,9,2.75), labels = Passlab)
  y <- c(2.3, 4.95, 7.6)
  for (i in seq_along(y)){
    tvar <- t(var)
    tvarcol <- matrix(tvar[i,], nrow = 1, ncol = 8)
    add.image(y[i], 5, tvarcol, 
              zlim=c(min(tvar), max(tvar)), axes=FALSE,
              image.width = 1, image.height = 0.45, 
              col = color.func(length(tvar)))
#    text(y[i], y = (at = seq(0.95,9,0.535)), 
#          labels = format(round((tvar[i,]), 2), nsmall = 2))
  }
  abline(v=c((11/3), (19/3)))
  abline(a=1.75, b=0)
  abline(a=2.85, b=0)
  abline(a=3.91, b=0)
  abline(a=5.01, b=0)
  abline(a=6.10, b=0)
  abline(a=7.15, b=0)
  abline(a=8.25, b=0)
  image.plot(var, col = color.func(length(tvar)), 
             legend.only = TRUE, zlim=c(min(var), max(var)),
             axes=FALSE, smallplot = c(.85,.9,0.2,0.92))
}

par(mfrow=c(2,4))
heatmap.t(Pre.ratio.2014)
title("2014 Pre-planting Fv quantities", cex.main = 1.5)
heatmap.t(Pre.SCN.eggs.2014)
title("2014 Pre-planting SCN eggs", cex.main = 1.5)
heatmap.t2(R5.DX.2014)
title("2014 R5 In-field Disease Index", cex.main = 1.5)
yield_heatmap.t2(Yield.2014)
title("Yield (bu/acre)", cex.main = 1.5)

geo.list.2014a$Pre.ratio$data <- geo.list.2014a$Pre.ratio$data * 1000000
fields.krig(geo.list.2014a$Pre.ratio)
```

    ## Warning: 
    ## Grid searches over lambda (nugget and sill variances) with  minima at the endpoints: 
    ##   (REML) Restricted maximum likelihood 
    ##    minimum at  right endpoint  lambda  =  0.04520064 (eff. df= 91.19999 )

``` r
title("Pre-planting Fv quantities", cex.main = 1.5)
fields.krig(geo.list.2014a$PreSCN.eggs)
title("2014 Pre-planting SCN eggs", cex.main = 1.5)
fields.krig(geo.list.2014b$R5.DX)
```

    ## Warning: 
    ## Grid searches over lambda (nugget and sill variances) with  minima at the endpoints: 
    ##   (REML) Restricted maximum likelihood 
    ##    minimum at  right endpoint  lambda  =  0.06003311 (eff. df= 22.79999 )

``` r
title("R5 Foliar SDS Disease Index", cex.main = 1.5)
fields.krig.yield(geo.list.2014b$Yield)
```

    ## Warning: 
    ## Grid searches over lambda (nugget and sill variances) with  minima at the endpoints: 
    ##   (REML) Restricted maximum likelihood 
    ##    minimum at  right endpoint  lambda  =  0.06003311 (eff. df= 22.79999 )

``` r
title("Soybean Yield", cex.main = 1.5)
```

![](Heatmaps_and_correlations_files/figure-markdown_github/Spatial%20plots%20of%202014%20raw%20data-1.png)

LLoyd's Index of Patchiness
---------------------------

To determine if any of the variables collected are significantly aggregated, calculate an LIP value From [Nordmeyer et al 2009:](http://www.stat.purdue.edu/~zhanghao/ASS/Reference%20Papers/Spatial%20and%20temporal%20dynamics%20of%20seeding%20population.pdf) Calculated as \[mean + ((var/mean) - 1)\] / mean If LIP is less than 1, it is not patchy but random If LIP is greater than one, there is significant aggregation

``` r
LIP.calc <- function(x){
  y <- as.vector(x)
  LIP <- (mean(y) + ((var(y) / mean(y))-1)) / mean(y)
  #LIP2 <- 1 + ((var(y) - mean(y))/mean(y)^2)
  cat("LIP is ",as.character(LIP))
}

LIP.calc(Pre.ratio.2014)
```

    ## LIP is  0.0797809252310824

``` r
LIP.calc(Pre.ratio.2015)
```

    ## LIP is  0.902438903413806

``` r
#Since LIP is meant to be used on count data, not density (proportional) data, try it with raw qPCR values
LIP.calc(ppra.2014$Pre.qPCR[1:96]) # Greater than 1
```

    ## LIP is  1.07773487911001

``` r
LIP.calc(ppra.2015$Pre.qPCR[1:96]) # Greater than 1
```

    ## LIP is  1.10411563206122

``` r
LIP.calc(ppra.2014$V3.DW[1:96])
```

    ## LIP is  -0.505552561882913

``` r
LIP.calc(ppra.2015$V3.DW[1:96])
```

    ## LIP is  -0.595634356505912

``` r
LIP.calc(ppra.2014$R5.DW[1:96])
```

    ## LIP is  0.847459500761406

``` r
LIP.calc(ppra.2015$R5.DW[1:96])
```

    ## LIP is  0.886836036733507

``` r
LIP.calc(ppra.2014$V3.foliar[1:96])
```

    ## LIP is  1.15823216293451

``` r
LIP.calc(ppra.2015$V3.foliar[1:96])
```

    ## LIP is  9.99573657162302

``` r
LIP.calc(ppra.2014$V3.root[1:96])
```

    ## LIP is  1.24120460194861

``` r
LIP.calc(ppra.2015$V3.root[1:96])
```

    ## LIP is  1.21456200508314

``` r
LIP.calc(ppra.2014$R5.foliar[1:96])
```

    ## LIP is  0.894848891240704

``` r
LIP.calc(ppra.2015$R5.foliar[1:96])
```

    ## LIP is  1.11246491812622

``` r
LIP.calc(ppra.2014$R5.root[1:96])
```

    ## LIP is  1.17901450792282

``` r
LIP.calc(ppra.2015$R5.root[1:96])
```

    ## LIP is  1.68967961881111

``` r
LIP.calc(ppra.2014$PreSCN.cysts[1:96])
```

    ## LIP is  1.3100390625937

``` r
LIP.calc(ppra.2014$PreSCN.eggs[1:96])
```

    ## LIP is  1.36432349434713

``` r
LIP.calc(ppra.2014$PreSCN.juvs[1:96])
```

    ## LIP is  1.57933275363862

``` r
LIP.calc(ppra.2015$PreSCN.cysts[1:96])
```

    ## LIP is  1.25800644468314

``` r
LIP.calc(ppra.2015$PreSCN.eggs[1:96])
```

    ## LIP is  1.44280536909704

``` r
LIP.calc(ppra.2015$PreSCN.juvs[1:96])
```

    ## LIP is  1.45476358337924

``` r
LIP.calc(ppra.2014$Pre.spiral[1:96])
```

    ## LIP is  1.74008336682328

``` r
LIP.calc(ppra.2014$Pre.lesion[1:96])
```

    ## LIP is  NaN

``` r
LIP.calc(ppra.2014$Pre.dagger[1:96])
```

    ## LIP is  NaN

``` r
LIP.calc(ppra.2015$Pre.spiral[1:96])
```

    ## LIP is  1.73186619924728

``` r
LIP.calc(ppra.2015$Pre.lesion[1:96])
```

    ## LIP is  4.78297873228996

``` r
LIP.calc(ppra.2015$Pre.dagger[1:96])
```

    ## LIP is  30.3684210526316

``` r
LIP.calc(ppra.2014$PostSCN.cysts[1:96])
```

    ## LIP is  1.24420331155248

``` r
LIP.calc(ppra.2014$PostSCN.eggs[1:96])
```

    ## LIP is  1.26394264047836

``` r
LIP.calc(ppra.2014$PostSCN.juvs[1:96])
```

    ## LIP is  1.41112273752746

``` r
LIP.calc(ppra.2015$PostSCN.cysts[1:96])
```

    ## LIP is  1.66581008198032

``` r
LIP.calc(ppra.2015$PostSCN.eggs[1:96])
```

    ## LIP is  1.79498673397302

``` r
LIP.calc(ppra.2015$PostSCN.juvs[1:96])
```

    ## LIP is  2.49203707006219

``` r
LIP.calc(ppra.2014$Post.spiral[1:96])
```

    ## LIP is  1.79275148886573

``` r
LIP.calc(ppra.2014$Post.lesion[1:96])
```

    ## LIP is  16.6789473684211

``` r
LIP.calc(ppra.2015$Post.spiral[1:96])
```

    ## LIP is  2.40502347280945

``` r
LIP.calc(ppra.2015$Post.lesion[1:96])
```

    ## LIP is  7.96250543714659

``` r
LIP.calc(ppra.2014$Pre.qPCR[1:96])
```

    ## LIP is  1.07773487911001

``` r
LIP.calc(ppra.2015$Pre.qPCR[1:96])
```

    ## LIP is  1.10411563206122

``` r
LIP.calc(ppra.2014$V3.qPCR[1:96])
```

    ## LIP is  0.866382535638941

``` r
LIP.calc(ppra.2015$V3.qPCR[1:96])
```

    ## LIP is  1.0981822791779

``` r
LIP.calc(ppra.2014$R5.qPCR[1:96])
```

    ## LIP is  0.860484231330568

``` r
LIP.calc(ppra.2015$R5.qPCR[1:96])
```

    ## LIP is  0.863709866589326

``` r
LIP.calc(ppra.2014$Post.qPCR[1:96])
```

    ## LIP is  1.9218782765723

``` r
LIP.calc(ppra.2015$Post.qPCR[1:96])
```

    ## LIP is  1.12974343830696

``` r
LIP.calc((ppra.2014$Post.ratio[1:96])*1000000)
```

    ## LIP is  1.52156242644103

``` r
LIP.calc((ppra.2015$Post.ratio[1:96])*1000000)
```

    ## LIP is  0.893328392122597

``` r
LIP.calc(ppra.2014$R4.DX[97:120])
```

    ## LIP is  1.56715787469571

``` r
LIP.calc(ppra.2015$R4.DX[1:96])
```

    ## LIP is  3.62638038229036

``` r
LIP.calc(ppra.2014$R5.DX[97:120])
```

    ## LIP is  1.51897236562385

``` r
LIP.calc(ppra.2015$R5.DX[1:96])
```

    ## LIP is  1.99920985471441

``` r
LIP.calc(ppra.2014$R6.DX[97:120])
```

    ## LIP is  1.2134264917173

``` r
LIP.calc(ppra.2015$R6.DX[1:96])
```

    ## LIP is  1.5648468027577

``` r
LIP.calc(ppra.2014$Yield[97:120])
```

    ## LIP is  1.02472603430343

``` r
LIP.calc(ppra.2015$Yield[97:144])
```

    ## LIP is  1.01825903319186

``` r
LIP.calc(ppra.2014$Pre.DNA[1:96])
```

    ## LIP is  1.10329116630433

``` r
LIP.calc(ppra.2015$Pre.DNA[1:96])
```

    ## LIP is  0.99519056761082

Correlation analyses
--------------------

There are a few categorical variables in the master data file that I don't need for this analysis. Drop these. Since I have 96 data points for most variables, I want to correlate the 96 points to 96 points But, since Yield is on a different scale from everything, I'll need to do some data averaging prior to correlations with yield

``` r
library(corrplot)
```

    ## corrplot 0.84 loaded

``` r
ppra.df <- ppra.2015[,-(1:6)]
#Yield is on a different scale from everything, so drop it from the ppra.2015 data frame for now
ppra.df <- ppra.df[,-53]
#Since yield is now gone from the data frame, I can delete columns 97:144
ppra.df <- ppra.df[-(97:144),]

#In order to obtain correlations with yield, I need to average my data points from 96 to 48
# So, average samples A1 and A2, A3 and A4, etc.
row.list <- 0
row.2015 <- 0
for (i in 1:52) {
  for (j in seq(1,96,2)) {
    row.list <- row.list + 1
    row.2015[[row.list]] <- (ppra.df[j, i] + ppra.df[j+1, i]) / 2
  }
}
avg.2015.mat <- matrix(row.2015, ncol = 52, nrow = 48)
colnames(avg.2015.mat) <- names(ppra.df)

#Add yield back in as a new column to the data frame
yield <- cbind(ppra.2015$Yield[97:144])
colnames(yield) <- c("Yield")
all.2015 <- cbind.data.frame(yield, avg.2015.mat)


#To make a corrplot that correlates the 96 points to 96 points, use "ppra.df" and ignore the yield boxes in the plot!!!
#To make a corrplot that correlates the 48 points to 48 poitns including yield, use "all.2015" and ignore all boxes except the yield boxes!!!
#Then, export the figures and adjust them as necessary in Powerpoint.
#Also, comment out the variables that were not duplicated in both years
ordered.2015 <- cbind(ppra.df$Pre.ratio, 
                      ppra.df$PreSCN.cysts, 
                      ppra.df$PreSCN.eggs,
                      ppra.df$PreSCN.juvs, 
                      ppra.df$Pre.spiral,
                      ppra.df$Pre.lesion, 
                      ppra.df$Pre.dagger,
                      #ppra.df$V3.qPCR,
                      #ppra.df$V3.DW, 
                      #ppra.df$V3.root,
                      #ppra.df$V3.foliar, 
                      #ppra.df$R5.qPCR,
                      #ppra.df$R5.DW,
                      #ppra.df$R5.root,
                      #ppra.df$R5.foliar, 
                      #ppra.df$R4.DX, 
                      ppra.df$R5.DX,
                      #ppra.df$R6.DX, 
                      #ppra.df$Post.ratio,
                      #ppra.df$PostSCN.cysts,
                      #ppra.df$PostSCN.eggs, 
                      #ppra.df$PostSCN.juvs,
                      #ppra.df$Post.spiral,
                      #ppra.df$Post.lesion,
                      all.2015$Yield)

colnames(ordered.2015) <- c("Pre.Fv.qPCR", 
                            "Pre.SCN.cysts", 
                            "Pre.SCN.eggs",
                            "Pre.SCN.juvs", 
                            "Pre.spiral", 
                            "Pre.lesion", 
                            "Pre.dagger",
                            #"V3.Fv.qPCR",
                            #"V3.root.DW", 
                            #"V3.root.rot",
                            #"V3.foliar.SDS", 
                            #"R5.Fv.qPCR",
                            #"R5.root.DW",
                            #"R5.root.rot",
                            #"R5.foliar.SDS" ,
                            #"R4.SDS.DX",
                            "R5.SDS.DX",
                            #"R6.SDS.DX", 
                            #"Post.Fv.qPCR",
                            #"Post.SCN.cysts", 
                            #"Post.SCN.eggs", 
                            #"Post.SCN.juvs", 
                            #"Post.spiral", 
                            #"Post.lesion",
                            "Yield")

#Perform the pairwise pearson correlations
all <- cor(ordered.2015, 
                  use = "pairwise.complete.obs",
                  method = "pearson")

#Determine if the correlations are significant
#This function obtains p-values for my matrix of correlations
p.matrix <- function(mat) {
    mat <- as.matrix(mat)
    n <- ncol(mat)
    p.mat<- matrix(NA, n, n)
    diag(p.mat) <- 0
    for (i in 1:(n - 1)) {
        for (j in (i + 1):n) {
            tmp <- cor.test(mat[, i], mat[, j])
            p.mat[i, j] <- p.mat[j, i] <- tmp$p.value
        }
    }
  colnames(p.mat) <- rownames(p.mat) <- colnames(mat)
  p.mat
}
all.p <- p.matrix(ordered.2015)

#Plot the corrplot
par(mfrow=c(1,1))
corrplot(all, 
         method = "color",
         type = "lower", 
         outline = TRUE,
         p.mat = all.p, 
         sig.level = 0.05,
         diag = FALSE,
         addCoefasPercent = FALSE,
         number.cex = 1.1,
         order = "original",
         insig = "blank",
         tl.col="black", tl.pos="lt", tl.cex = 0.8, cl.cex = 1,
         cl.pos = "n",
         addCoef.col = "black")
```

![](Heatmaps_and_correlations_files/figure-markdown_github/Pearson%20correlations%202015-1.png)

Now, repeat for 2014 Again, I need to do some averaging to get to the same level as yield (and R5 DX) Get all variables with 96 data points to an average with 24 data points

``` r
# First, average A1 and A2, A3 and A4, etc.
row.list <- 0
row.2014 <- 0
row.2014.df <- cbind(ppra.2014[,(7:33)])
#Need to scale the 2014 data to be on same scale as DX ratings
#This code takes the first 25 variables and averages the first two
#For example, averages A1 and A2, A3 and A4, etc...
for (i in 1:27) {
  for (j in seq(1,96,2)) {
    row.list <- row.list + 1
    row.2014[[row.list]] <- (row.2014.df[j, i] + row.2014.df[j+1, i]) / 2
  }
}
row.2014.mat <- matrix(row.2014, ncol = 27, nrow = 48)

#Now average A1+2 with B1+2, A3+4 with B3+4, etc...
avg.list <- 0
avg.2014 <- 0
avg.2014.df <- data.frame(row.2014.mat)
for (i in 1:27) {
  for (j in seq(1,8)) {
    avg.list <- avg.list + 1
    avg.2014[[avg.list]] <- (avg.2014.df[j, i] + avg.2014.df[j+8, i]) / 2
  }
}
avg.2014.mat1 <- matrix(avg.2014, ncol = 27, nrow = 8)
#avg.2014.mat1
#Now average C1+2 with D1+2, C3+4 with D3+4, etc...
avg.list <- 0
avg.2014 <- 0
avg.2014.df <- data.frame(row.2014.mat)
for (i in 1:27) {
  for (j in seq(17,24)) {
    avg.list <- avg.list + 1
    avg.2014[[avg.list]] <- (avg.2014.df[j, i] + avg.2014.df[j+8, i]) / 2
  }
}
avg.2014.mat2 <- matrix(avg.2014, ncol = 27, nrow = 8)
#avg.2014.mat2
#Now average E1+2 with F1+2, E3+4 with F3+4, etc...
avg.list <- 0
avg.2014 <- 0
avg.2014.df <- data.frame(row.2014.mat)
for (i in 1:27) {
  for (j in seq(33,40)) {
    avg.list <- avg.list + 1
    avg.2014[[avg.list]] <- (avg.2014.df[j, i] + avg.2014.df[j+8, i]) / 2
  }
}
avg.2014.mat3 <- matrix(avg.2014, ncol = 27, nrow = 8)
#avg.2014.mat3

#Now, rbind the three matricies into one dataframe with 24 rows
avg.2014.mat <- data.frame(rbind(avg.2014.mat1, 
                                 avg.2014.mat2,
                                 avg.2014.mat3))
colnames(avg.2014.mat) <- names(ppra.2014[7:33])


#Finally, make corrplot with data (now to scale with DX ratings)
dx <- cbind(ppra.2014$R4.DX[97:120], 
            ppra.2014$R5.DX[97:120],
            ppra.2014$R6.DX[97:120],
            ppra.2014$Yield[97:120])
colnames(dx) <- c("R4.DX", "R5.DX", "R6.DX", "Yield")
all.2014 <- cbind.data.frame(dx, avg.2014.mat)

ordered.2014 <- cbind(all.2014$Pre.ratio,
                      all.2014$PreSCN.cysts,
                      all.2014$PreSCN.eggs,
                      all.2014$PreSCN.juvs,
                      all.2014$Pre.spiral,
                      all.2014$Pre.lesion,
                      all.2014$Pre.dagger,
                      #all.2014$V3.qPCR,
                      #all.2014$V3.DW,
                      #all.2014$V3.root,
                      #ll.2014$V3.foliar,
                      #all.2014$R5.qPCR,
                      #all.2014$R5.DW,
                      #all.2014$R5.root,
                      #all.2014$R5.foliar,
                      #all.2014$R4.DX,
                      all.2014$R5.DX,
                      #all.2014$R6.DX,
                      #all.2014$Post.qPCR,
                      #all.2014$PostSCN.cysts,
                      #all.2014$PostSCN.eggs,
                      #all.2014$PostSCN.juvs,
                      #all.2014$Post.spiral,
                      #all.2014$Post.lesion,
                      all.2014$Yield
                      )

colnames(ordered.2014) <- c("Pre.Fv.qPCR",
                            "Pre.SCN.cysts", 
                            "Pre.SCN.eggs", 
                            "Pre.SCN.juvs",
                            "Pre.spiral", 
                            "Pre.lesion",
                            "Pre.dagger",
                            #"V3.Fv.qPCR",
                            #"V3.root.DW", 
                            #V3.root.rot",
                            #"V3.foliar.SDS", 
                            #R5.Fv.qPCR",
                            #R5.root.DW",
                            #"R5.root.rot",
                            #"R5.foliar.SDS",
                            #"R4.SDS.DX", 
                            "R5.SDS.DX",
                            #"R6.SDS.DX", 
                            #"Post.Fv.qPCR",
                            #"Post.SCN.cysts",
                            #"Post.SCN.eggs", 
                            #"Post.SCN.juvs",
                            #"Post.spiral",
                            #"Post.lesion",
                            "Yield"
                            )

#Perform the pairwise pearson correlations
ordered.correlation <- cor(ordered.2014, use = "pairwise.complete.obs",
                       method = "pearson")
```

    ## Warning in cor(ordered.2014, use = "pairwise.complete.obs", method =
    ## "pearson"): the standard deviation is zero

``` r
#Determine the significance of the correlations
ordered.p <- p.matrix(ordered.2014)
```

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

    ## Warning in cor(x, y): the standard deviation is zero

``` r
#Plot the corrplot
par(mfrow=c(1,1))
corrplot(ordered.correlation, 
         method = "color",
         type = "upper", 
         outline = TRUE,
         p.mat = ordered.p, 
         sig.level = 0.05,
         diag = FALSE,
         addCoefasPercent = FALSE,
         number.cex = 1.5,
         order = "original",
         insig = "blank",
         tl.col="black", tl.pos="lt", tl.cex = 0.8, cl.cex = 1,
         cl.pos = "n",
         addCoef.col = "black")
```

![](Heatmaps_and_correlations_files/figure-markdown_github/Pearson%20correlations%202014-1.png)

``` r
#Note: to get the image used in the manuscript, I manually merged the 2014 and 2015 corrplots in powerpoint
```
