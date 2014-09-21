Human Activity Recognition
========================================================
    ### Introduction
    ### Read and Prepare Data
    
    ```r
    #Knitr prep
    library(caret);library(randomForest)
    ```
    
    ```
    ## Warning: package 'caret' was built under R version 3.1.1
    ```
    
    ```
    ## Loading required package: lattice
    ## Loading required package: ggplot2
    ## randomForest 4.6-7
    ## Type rfNews() to see new features/changes/bug fixes.
    ```
    
    ```r
    training<- read.csv("pml-training.csv",header=T)
    training[training=="#DIV/0!"]<-NA
    columnNAsBlanks<-colSums(training==""|is.na(training),na.rm=T)/dim(training)[1]*100
    variablesExclude<-names(columnNAsBlanks[columnNAsBlanks>60])
    training<-training[,!names(training)%in%variablesExclude]
    variablesInclude<-names(training)[grepl("_belt|_forearm|_arm|_dumbell|classe",names(training))]
    training<-training[,names(training)%in%variablesInclude]
    
    
    testing<- read.csv("pml-testing.csv",header=T)
    testing[testing=="#DIV/0!"]<-NA
    testing<-testing[,names(testing) %in% names(training)]
    
    set.seed(4444)
    sample<-sample(nrow(training),10000)
    training<-training[sample,]
    ```
Variables exhibiting high levels of sparsity, as defined by having over 60 percent of observations consisting of NA or 0, are removed from the analysis.  Thereafter, only variables pertaining to belt, forearm, arm and dumbbell sensors, in addition to the predictor (classe), are included.  Lastly, a random sample of 10,000 is drawn to ease computation constrain.


### Cross Validation   

```r
foldsTrain<-createFolds(y=training$classe,k=3,list=T,returnTrain=T)
foldsTest<-createFolds(y=training$classe,k=3,list=T,returnTrain=F)

sapply(foldsTrain,length)
```

```
## Fold1 Fold2 Fold3 
##  6667  6668  6665
```

```r
sapply(foldsTest,length)
```

```
## Fold1 Fold2 Fold3 
##  3334  3332  3334
```

```r
training1<-training[foldsTrain[[1]],]
training2<-training[foldsTrain[[2]],]
training3<-training[foldsTrain[[3]],]

testing1<-training[foldsTest[[1]],]
testing2<-training[foldsTest[[2]],]
testing3<-training[foldsTest[[3]],]
```
3-fold cross validation is implemented and 3 training and testing data subsets are generated.


### Model Build and Test: Classification Trees

```r
modFit_tree_1<-train(classe~.,method="rpart",data=training1)
```

```
## Loading required package: rpart
```

```r
modFit_tree_2<-train(classe~.,method="rpart",data=training2)
modFit_tree_3<-train(classe~.,method="rpart",data=training3)
```

```r
accuracy_tree_1<-confusionMatrix(testing1$classe,predict(modFit_tree_1,newdata=testing1))$overall["Accuracy"]
accuracy_tree_2<-confusionMatrix(testing2$classe,predict(modFit_tree_2,newdata=testing2))$overall["Accuracy"]
accuracy_tree_3<-confusionMatrix(testing3$classe,predict(modFit_tree_3,newdata=testing3))$overall["Accuracy"]

ExpOutSampleAccuracy_tree<-mean(accuracy_tree_1,accuracy_tree_2,accuracy_tree_3)

ExpOutSampleError_tree<-1-ExpOutSampleAccuracy_tree
```
Classification tree models are built for each of the training sets, which in turn are tested versus the test subsets.   The average accuracy for all three classification models is 43.46% and the expected error for is 56.54% (1-accuracy).



### Model Build and Test: Random Forest

```r
modFit_rf_1<-randomForest(classe~.,prox=T, data=training1,ntree=50)
modFit_rf_2<-randomForest(classe~.,prox=T, data=training2,ntree=50)
modFit_rf_3<-randomForest(classe~.,prox=T, data=training3,ntree=50)
```



```r
accuracy_rf_1<-confusionMatrix(testing1$classe,predict(modFit_rf_1,newdata=testing1))$overall["Accuracy"]
accuracy_rf_2<-confusionMatrix(testing2$classe,predict(modFit_rf_2,newdata=testing2))$overall["Accuracy"]
accuracy_rf_3<-confusionMatrix(testing3$classe,predict(modFit_rf_3,newdata=testing3))$overall["Accuracy"]

ExpOutSampleAccuracy_rf<-mean(accuracy_rf_1,accuracy_rf_2,accuracy_rf_3)

ExpOutSampleError_rf<-1-ExpOutSampleAccuracy_rf
```

Similarly to the proccess for classification tree models, three random forest models are generated, limiting the number of trees to grow to 50 to ease computational strain.  The average accuracy for all three random tree models is 99.34% and the expected error for is 0.66% (1-accuracy).


### Conclusion: Model Selection and Prediction

```r
modFit_rf_final<-randomForest(classe~.,prox=T, data=training,ntree=50)

answers<-predict(modFit_rf_final,newdata=testing)
```
The random forest model seems to be the better approach as judged by accuracy and expected error.  Thus a final model is generated from all 10,000 observations.  The final random forest model is put to the testing data and predications are generated: B, A, B, A, A, E, D, B, A, A, B, C, B, A, E, E, A, B, B, B. 