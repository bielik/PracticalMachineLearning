# Practical Machine Learning - Exercise type prediction project
Martin Bielik  
October 22, 2017  



## Data cleaning - pre-processing

At first, I reduce the complexity of prediction problem by elliminating variables with missing or NA values.
The Participant names and exercise time stamps are removed as well




```r
variable.CullBlank <- as.logical(training[1,] == "")
variable.CullNA <- is.na(variable.CullBlank)
variable.cull <- !variable.CullNA & !variable.CullBlank

variable.cull[1:5] <- F

trainingProc <- training[, variable.cull]
testingProc <- testing[, variable.cull]
testingProc <- testingProc[,1:(ncol(testingProc)-1)]
```

The final training data set was redced from the original 160 to 54.

## Training

SInce the goal of this porject is prediction and not the ability to interpret and understand the link between sensor data and exercise quality, I adopted one of the most effective and universal machine learning algorithms - Random forest.

The out of the sample error was estimated by 5 fold cross-validation.

###Side note: 
The Random Fordest training was implemented as parallel computation.


```r
library(caret)
library(doParallel)
library(parallel)

cluster <- makeCluster(detectCores() - 1)
registerDoParallel(cluster)

fitControl <- trainControl(method = "cv",
                           number = 5,
                           allowParallel = TRUE)
modFit.RF <- train(classe ~ ., data = trainingProc, method = "rf", trControl = fitControl)

stopCluster(cluster)
registerDoSEQ()
```
## Final model

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 28
## 
##         OOB estimate of  error rate: 0.13%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 5578    1    0    0    1 0.0003584229
## B    6 3788    2    1    0 0.0023702923
## C    0    5 3417    0    0 0.0014611338
## D    0    0    7 3208    1 0.0024875622
## E    0    0    0    2 3605 0.0005544774
```

The estimated out of sample model accuracy is 99.7961472%

## Predicting on test set


```
##    ID.Question Prediction
## 1            1          B
## 2            2          A
## 3            3          B
## 4            4          A
## 5            5          A
## 6            6          E
## 7            7          D
## 8            8          B
## 9            9          A
## 10          10          A
## 11          11          B
## 12          12          C
## 13          13          B
## 14          14          A
## 15          15          E
## 16          16          E
## 17          17          A
## 18          18          B
## 19          19          B
## 20          20          B
```

After inserting the prediction into the final quiz, all 20 predictions were marked as corret resulting into the accuracy of 100%
