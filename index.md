# Prediction Assignment
C. Smith  
March 31, 2016  




## Introduction 

The objective of this project is to take accelerometer data that monitors dumbbell motions and classify to determine if the exercise is being performed correctly.  The data is from http://groupware.les.inf.puc-rio.br/har. The data is from accelerometers on the belt, forearm, arm and dumbell of 6 participants. From their website:

Six young health participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).

I will attempt to predict the manner in which the participants did the exercise.

## Data

I downloaded the data and read it into a training and test set.



```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Rattle: A free graphical interface for data mining with R.
## Version 4.1.0 Copyright (c) 2006-2015 Togaware Pty Ltd.
## Type 'rattle()' to shake, rattle, and roll your data.
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## [1] 19622   160
```

```
## [1]  20 160
```

In the training set, there are 19,622 observations and 160 variables. The test set consists of 20 observations of 160 variables. The first seven variables don't look like they will be useful for prediction, e.g, X, user_name, etc., so I will remove those from both training and testing.
Of the remaining variables, many are NA, so I will remove those that have more than 80% the values as NA. 80% is arbitrary. That still leaves us with 53 variables. 

```r
unusedColumnNames <- colnames(trainingSet)[1:7]
unusedColumnNames
```

```
## [1] "X"                    "user_name"            "raw_timestamp_part_1"
## [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
## [7] "num_window"
```

```r
training <- trainingSet [,-which(names(trainingSet) %in% unusedColumnNames)]
testing <- testingSet [,-which(names(testingSet) %in% unusedColumnNames)]

tooMany <- 19622 *.8
training <- training[,colSums(is.na(training)) < tooMany]
tooMany <- 20 *.8
testing <- testing[,colSums(is.na(testing)) < tooMany]
dim(training)
```

```
## [1] 19622    53
```

```r
dim(testing)
```

```
## [1] 20 53
```

## Divide Training Data 
To estimate the prediction accuracy, I will divide the training set into two subsets, 70% into the first new training subset and 30% in the last training subset. These will be used as training and testing, instead of the test set.


```r
inTrain <- createDataPartition(y = training$classe, p = .7, list=F)
subTrain <- training[inTrain,]
subTest <-training[-inTrain,]
dim(subTrain)
```

```
## [1] 13737    53
```

```r
dim(subTest)
```

```
## [1] 5885   53
```
## Build Model

The model I'll use is the random forest model.  I chose this model because the random forest algorithm works well with a large number of predictors (53) and calculates the strongest predictor at the top split and then the next and so on. The random forest algorithm will also give us the important variables. Random decision forests correct for decision trees' habit of overfitting to their training set, so this seems like a reasonable choice.

## Cross Validation

I'll use the 70% subset of the training  with Cross validation (cv) as the train control method and iterate 5 times to attempt to improve the prediction rate. Five is an arbitrary number.

```r
set.seed(123)
modCV <- train(subTrain$classe ~ .,  trControl=trainControl(method = "cv", number = 5), data = subTrain, method="rf")
prediction <- predict(modCV, newdata=subTest)
confusionMatrix(prediction, subTest$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1672    7    0    0    0
##          B    2 1130    8    0    0
##          C    0    2 1015   17    2
##          D    0    0    3  946    1
##          E    0    0    0    1 1079
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9927          
##                  95% CI : (0.9902, 0.9947)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9908          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9988   0.9921   0.9893   0.9813   0.9972
## Specificity            0.9983   0.9979   0.9957   0.9992   0.9998
## Pos Pred Value         0.9958   0.9912   0.9797   0.9958   0.9991
## Neg Pred Value         0.9995   0.9981   0.9977   0.9964   0.9994
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2841   0.1920   0.1725   0.1607   0.1833
## Detection Prevalence   0.2853   0.1937   0.1760   0.1614   0.1835
## Balanced Accuracy      0.9986   0.9950   0.9925   0.9903   0.9985
```

## Expected Out of Sample Error

The out of sample error is the rate at which the testing data is misclassified.  From the Confusion Matrix we can see that the misclassifications are:

A - 7
B - 10
C - 21
D - 4
E - 1

Which is a total of 43 misclassification. The test set has 5885 observations, so that gives an error rate of .0073.  This is also given by 1 - Accuracy, .0073

## Test Set Data

To predict the outcome with the original test set, we will feed the new data into the model.


```r
predict(modCV, newdata=testing)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

## Conclusion

The random forest gave an excellent prediction with an accuracy of 99.27%. It also calculated the important variables, this is a plot of the top ten most important variables, as determined by the random forest model.  Five iterations may be excessive, however, as the cross validation model took over 10 minutes to build on my machine.  

![](index_files/figure-html/unnamed-chunk-6-1.png)


## Citation

Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013. 

Wikipedia https://en.wikipedia.org/wiki/Random_forest

