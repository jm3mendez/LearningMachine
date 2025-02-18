# Learning Machine classification Case using RandomForest
Jose Mendez  
Saturday, December 26, 2015  

#  RandomForest   Learning Machine Case

The idea here, is using Data that come from HAR devices and a Learning Machine scheme (RF) we could predict the type of activity that some guides are making categorized in 5 classe/categopries as the data expose. ( thank for the Data to:  http://groupware.les.inf.puc-rio.br/har )

# Preparaing environment and Data


```r
#  Libs
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(R.utils))
suppressPackageStartupMessages(library(ggplot2))
suppressPackageStartupMessages(library(caret))
suppressPackageStartupMessages(library(randomForest))
library(dplyr); library(caret); library(ggplot2); library(randomForest)

# Read data
setwd('/home/jmendez/learningMachine/data')
set.seed(12357)
#download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",destfile="pml-training.csv", method="curl")
#download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv",destfile="pml-testing.csv", method="curl")
pml=read.csv('pml-training.csv', stringsAsFactors = FALSE)
test=read.csv('pml-testing.csv', stringsAsFactors = FALSE)
str(pml,list.len=0)
```

```
## 'data.frame':	19622 obs. of  160 variables:
##   [list output truncated]
```

```r
#  'data.frame':	19622 obs. of  160 variables
```

# Modeling
 Our model is related to obtain classe of activity like:  sitting-down, starting-up, starting, walking,  sitting at the begining using closed to 19000 observations over 160 variables.
 We will use first the physical consideration in order to select the variables associated with this potencial model that have to involve accelarations, some positional results and time variables. The basic model propose should be:  
     classe ~ accelerator + positional + time variables + others   
 The others variables like positional are related with acceleration or delta velocity or delta positional or spacial components. 
 Then we will use acceleration components preferently, if not referred, positional and time to classify 5 classes excluidng some id variables that are related in general with overfitting. 

 Then we need to reselect the information we call that apml


```r
apml=select(pml, raw_timestamp_part_1, raw_timestamp_part_2, num_window,total_accel_belt, gyros_belt_x, gyros_belt_y, gyros_belt_z, accel_belt_x, accel_belt_y, accel_belt_z, magnet_belt_x, magnet_belt_y, magnet_belt_z,total_accel_arm, gyros_arm_x, gyros_arm_y, gyros_arm_z, accel_arm_x,accel_arm_y, accel_arm_z, magnet_arm_x, magnet_arm_y, magnet_arm_z, roll_dumbbell, pitch_dumbbell, yaw_dumbbell,classe)
# We check that approach. It was fine but slow then we reduce more the model as part of try an error process in order to search the adequate model.
# that is, in order to improve the model we extracted correlationed variables.
testcorr=cor(apml[-27])
names(apml)[findCorrelation(testcorr)]
```

```
## [1] "accel_belt_z" "accel_belt_y" "gyros_arm_x"
```

```r
###[1] "accel_belt_z" "accel_belt_y" "gyros_arm_x"
# Then we reduce from 190 to 24 variables to participate in the final model. thus new apml is:
apml=select(pml, raw_timestamp_part_1, raw_timestamp_part_2, num_window, total_accel_belt, gyros_belt_x, gyros_belt_y, gyros_belt_z,accel_belt_x, magnet_belt_x, magnet_belt_y, magnet_belt_z,
total_accel_arm, gyros_arm_y, gyros_arm_z, accel_arm_x,accel_arm_y, accel_arm_z, magnet_arm_x, magnet_arm_y, magnet_arm_z, roll_dumbbell,pitch_dumbbell,yaw_dumbbell,classe)
```
# The Ramdom Forest and Cross Validation Learning Machine scheme used


```r
# Then we defining the training and testing data
inTrain=createDataPartition(y=apml$classe, p=0.98, list=FALSE)
training=apml[inTrain,]
testing=apml[-inTrain,]
dim(training); dim(testing)
```

```
## [1] 19232    24
```

```
## [1] 390  24
```

# Learning Machine

 We Used random forest to create the model as classifier and cross validation with k-fold 5 too 
 The process time implied was high I used som facilities to improve that. resampling might give more better  result but i probed k=3 and now k=5 and the result was very closed, then augment the partitioning is not necessary.

ptm=proc.time();
modFit <- train(classe ~ ., method="rf", data=training, prox=TRUE, allowParallel=TRUE, trControl = trainControl(method="cv", number = 3))
proc.time() - ptm
```
    user   system  elapsed 
9003.021  182.622 9202.038 

modFit

19232 samples
   23 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (3 fold) 
Summary of sample sizes: 12821, 12822, 12821 
Resampling results across tuning parameters:

  mtry  Accuracy   Kappa      Accuracy SD   Kappa SD       '/n'
   2    0.9936044  0.9919107  0.0012478554  0.0015782508   '/n'
  12    0.9979201  0.9973692  0.0003604496  0.0004560641   '/n'
  23    0.9965681  0.9956587  0.0015834786  0.0020039552   '/n'

Accuracy was used to select the optimal model using  the largest value.
The final value used for the model was mtry = 12. 

 pred <- predict(modFit, newdata = training)
 confusionMatrix(pred, training$classe)

Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 5469    0    0    0    0
         B    0 3722    0    0    0
         C    0    0 3354    0    0
         D    0    0    0 3152    0
         E    0    0    0    0 3535

Overall Statistics
                                     
               Accuracy : 1          
                 95% CI : (0.9998, 1)
    No Information Rate : 0.2844     
    P-Value [Acc > NIR] : < 2.2e-16  
                                     
                  Kappa : 1          
 Mcnemar's Test P-Value : NA         

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
Prevalence             0.2844   0.1935   0.1744   0.1639   0.1838
Detection Rate         0.2844   0.1935   0.1744   0.1639   0.1838
Detection Prevalence   0.2844   0.1935   0.1744   0.1639   0.1838
Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000


 pred2=predict(modFit,newdata = testing)
 pred2

 confusionMatrix(pred2, testing$classe)
 Confusion Matrix and Statistics

          Reference
Prediction   A   B   C   D   E
         A 111   0   0   0   0
         B   0  75   0   0   0
         C   0   0  68   1   0
         D   0   0   0  63   0
         E   0   0   0   0  72

Overall Statistics
                                          
               Accuracy : 0.9974          
                 95% CI : (0.9858, 0.9999)
    No Information Rate : 0.2846          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9968          
 Mcnemar's Test P-Value : NA              

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            1.0000   1.0000   1.0000   0.9844   1.0000
Specificity            1.0000   1.0000   0.9969   1.0000   1.0000
Pos Pred Value         1.0000   1.0000   0.9855   1.0000   1.0000
Neg Pred Value         1.0000   1.0000   1.0000   0.9969   1.0000
Prevalence             0.2846   0.1923   0.1744   0.1641   0.1846
Detection Rate         0.2846   0.1923   0.1744   0.1615   0.1846
Detection Prevalence   0.2846   0.1923   0.1769   0.1615   0.1846
Balanced Accuracy      1.0000   1.0000   0.9984   0.9922   1.0000


 Then the 20th predictions asked using modFit are:
 
 pred3=predict(modFit,newdata=atest)
 pred3
 [1] B A B A A E D B A A B C B A E E A B B B
```

# Conclusion

It scheme worked well. Seeing the confusion matriz and the results of accuracy of 100% and the other statistics like The error rate 0, but it is not implies that we are completely free of other 
effects, at this time we tried to reduce the overfitting using Cross validation in a more 
evolutioned fitting method like random forest. We could obtained a underfittingig if we reduce 
the model with less representative variables. it was not the case.  The noise captured in 
overfitting is reduced with the random and cross validation scheme useed. Kappa value=1 imply 
very good agreement good prediction. This is an uncommon result, very nice or rare result 
but worked.

