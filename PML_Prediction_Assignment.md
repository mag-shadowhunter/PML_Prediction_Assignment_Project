---
title: "PML_Prediction_Assignment"
author: "Meghana Anumolu"
date: "10/20/2020"
output:
  html_document:
    keep_md: yes
---


```r
knitr::opts_chunk$set
```

```
## function (...) 
## {
##     dots = resolve(...)
##     if (length(dots)) 
##         defaults <<- merge(dots)
##     invisible(NULL)
## }
## <bytecode: 0x00000000169c63e0>
## <environment: 0x00000000169d2860>
```

```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
library(rpart)
library(rpart.plot)
library(RColorBrewer)
library(tidyverse)
```

```
## -- Attaching packages ------------------------------------------- tidyverse 1.3.0 --
```

```
## v tibble  3.0.4     v dplyr   1.0.2
## v tidyr   1.1.2     v stringr 1.4.0
## v readr   1.4.0     v forcats 0.5.0
## v purrr   0.3.3
```

```
## -- Conflicts ---------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
## x purrr::lift()   masks caret::lift()
```

```r
library(rattle)
```

```
## Loading required package: bitops
```

```
## Rattle: A free graphical interface for data science with R.
## Version 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.
## Type 'rattle()' to shake, rattle, and roll your data.
```

```r
library(parallel)
library(doParallel)
```

```
## Loading required package: foreach
```

```
## 
## Attaching package: 'foreach'
```

```
## The following objects are masked from 'package:purrr':
## 
##     accumulate, when
```

```
## Loading required package: iterators
```

```r
library(e1071)
library(randomForest)
```

```
## randomForest 4.6-14
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:rattle':
## 
##     importance
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
sessionInfo()
```

```
## R version 3.6.2 (2019-12-12)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 17134)
## 
## Matrix products: default
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] parallel  stats     graphics  grDevices utils     datasets  methods  
## [8] base     
## 
## other attached packages:
##  [1] randomForest_4.6-14 e1071_1.7-3         doParallel_1.0.16  
##  [4] iterators_1.0.13    foreach_1.5.1       rattle_5.4.0       
##  [7] bitops_1.0-6        forcats_0.5.0       stringr_1.4.0      
## [10] dplyr_1.0.2         purrr_0.3.3         readr_1.4.0        
## [13] tidyr_1.1.2         tibble_3.0.4        tidyverse_1.3.0    
## [16] RColorBrewer_1.1-2  rpart.plot_3.0.9    rpart_4.1-15       
## [19] caret_6.0-86        ggplot2_3.2.1       lattice_0.20-41    
## 
## loaded via a namespace (and not attached):
##  [1] httr_1.4.1           jsonlite_1.6.1       splines_3.6.2       
##  [4] prodlim_2019.11.13   modelr_0.1.8         assertthat_0.2.1    
##  [7] stats4_3.6.2         blob_1.2.1           cellranger_1.1.0    
## [10] yaml_2.2.1           ipred_0.9-9          pillar_1.4.3        
## [13] backports_1.1.5      glue_1.4.2           pROC_1.16.2         
## [16] digest_0.6.23        rvest_0.3.6          colorspace_1.4-1    
## [19] recipes_0.1.14       htmltools_0.4.0      Matrix_1.2-18       
## [22] plyr_1.8.5           timeDate_3043.102    pkgconfig_2.0.3     
## [25] broom_0.7.2          haven_2.3.1          scales_1.1.0        
## [28] gower_0.2.2          lava_1.6.8           generics_0.0.2      
## [31] ellipsis_0.3.0       withr_2.1.2          nnet_7.3-12         
## [34] lazyeval_0.2.2       cli_2.0.1            survival_3.1-8      
## [37] magrittr_1.5         crayon_1.3.4         readxl_1.3.1        
## [40] evaluate_0.14        fansi_0.4.1          fs_1.5.0            
## [43] nlme_3.1-142         MASS_7.3-51.4        xml2_1.3.2          
## [46] class_7.3-15         tools_3.6.2          data.table_1.12.8   
## [49] hms_0.5.3            lifecycle_0.2.0      munsell_0.5.0       
## [52] reprex_0.3.0         compiler_3.6.2       rlang_0.4.8         
## [55] grid_3.6.2           rstudioapi_0.11      rmarkdown_2.4       
## [58] gtable_0.3.0         ModelMetrics_1.2.2.2 codetools_0.2-16    
## [61] DBI_1.1.0            reshape2_1.4.3       R6_2.4.1            
## [64] lubridate_1.7.9      knitr_1.28           stringi_1.4.4       
## [67] Rcpp_1.0.3           vctrs_0.3.4          dbplyr_1.4.4        
## [70] tidyselect_1.1.0     xfun_0.12
```

```r
set.seed(1)
```

## Data for the analysis

URL for training data 
<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv>

URL for test data
<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv>



## Extracting Data


```r
rm(list=ls())

url_1 <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
url_2 <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

# download.file(url_1, destfile = "pml-training.csv")
# download.file(url_2, destfile = "pml-testing.csv")

loc <- paste(getwd(),"/", sep="")
tr_file <- file.path(loc, "machine-train-data.csv")
tst_file <- file.path(loc, "machine-test-data.csv")
```


## Reading files


```r
if (!file.exists(tr_file)) 
        download.file(url_1, destfile=tr_file)

if (!file.exists(tst_file)) 
        download.file(url_2, destfile=tst_file)

tr_data <- read.csv(tr_file, na.strings=c("NA","#DIV/0!",""))
tst_data <- read.csv(tst_file, na.strings=c("NA","#DIV/0!",""))
```

##Obtaining tidy Data


```r
tr_data <- tr_data[,7:length(colnames(tr_data))]
tst_data <- tst_data[,7:length(colnames(tst_data))]

tr_data <- tr_data[, colSums(is.na(tr_data)) == 0] 
tst_data <- tst_data[, colSums(is.na(tst_data)) == 0] 

nzv <- nearZeroVar(tr_data,saveMetrics=TRUE)
zv <- sum(nzv$nzv)
if ((zv>0)) 
        tr_data <- tr_data[,nzv$nzv==FALSE]
```

## Split the data in 70 and 30 ratio


```r
set.seed(100)
tr <- createDataPartition(tr_data$classe, p=0.7, list=F)
train <- tr_data[tr, ]
test <- tr_data[-tr, ]
```

## Train a model using random forest algorithm


```r
cl <- makeCluster(detectCores() - 1) 

# Register multi-core
registerDoParallel(cl)

#default resampling
set.seed(1001)


c_para <- trainControl(method="cv", 5)

my_model_1 <- randomForest(classe ~ ., data = train,trControl=c_para,
                        importance = TRUE, ntrees = 10)
my_model_1
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = train, trControl = c_para,      importance = TRUE, ntrees = 10) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.25%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 3904    1    0    0    1 0.0005120328
## B    6 2649    3    0    0 0.0033860045
## C    0    7 2389    0    0 0.0029215359
## D    0    0   12 2239    1 0.0057726465
## E    0    0    0    4 2521 0.0015841584
```

```r
saveRDS(my_model_1, "my_model_1.rds")

##load rf model

my_model_1 <- readRDS("my_model_1.rds")


#Deregister multi-core
stopCluster(cl)

registerDoSEQ()

      con <- file(getwd())
      on.exit(close(con))
```

## Estimatation


```r
pred_bt <- predict(my_model_1, test)

confusionMatrix(pred_bt, test$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    0    0    0    0
##          B    0 1139    3    0    0
##          C    0    0 1023   10    0
##          D    0    0    0  954    1
##          E    0    0    0    0 1081
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9976         
##                  95% CI : (0.996, 0.9987)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.997          
##                                          
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   0.9971   0.9896   0.9991
## Specificity            1.0000   0.9994   0.9979   0.9998   1.0000
## Pos Pred Value         1.0000   0.9974   0.9903   0.9990   1.0000
## Neg Pred Value         1.0000   1.0000   0.9994   0.9980   0.9998
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2845   0.1935   0.1738   0.1621   0.1837
## Detection Prevalence   0.2845   0.1941   0.1755   0.1623   0.1837
## Balanced Accuracy      1.0000   0.9997   0.9975   0.9947   0.9995
```

#Course Project Prediction Quiz

### Apply the test data to the model


```r
test_result <- predict(my_model_1, 
                   tst_data[, -length(names(tst_data))])
test_result
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```
