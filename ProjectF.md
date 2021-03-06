# Practical Machine Learning

### Pees Assessment Project

The goal of this project is, given usage data from 
accelerometers on the belt, 
forearm, arm, and dumbell 
of 6 participants, to predict the manner 
in which they did the exercise, which is quantified by
the $classe$ variable in the provided *.csv* file. For this purpose, 
we compute tree different models, choosing the
best one as measured by *accuracy*.



## Data Preprocessing.

The provided data is contained in the "*pml-training.csv* file. 
The first step is to load this data into a data frame object and
prepossess it by removing *unwanted* variables. First I remove 
the variables that contain $NA$ instances since, upon a visual inspection of the data,
these variables contain a minimal amount of information. Of the remaining variables
(60) I remove the variables $X$ and $user\_name$ given that I do not consider 
that they do not contain useful information for the predictors.

The final preprocessing step is to divide the data into two separated sets:
$training$ used for predictor training and $testing$ which will be used for 
cross-validation.
 


```r
# Loads the data frame.
df <- read.csv("pml-training.csv",  
        na.strings = c("NA", "", "#DIV/0!"))
        
# Removes data with NA's
clean <- function(x) {
    j = 1
    k = 1
    l <- c()
    for (i in x){
        if(sum(is.na(i)) <= 0){
            l[k] <- j
            k = k + 1
            }
        j <- j + 1
        }
    l
    }
df <- df[,clean(df)]

#Remove useless variables.
rVars <- names(df) %in% c("X","user_name") 
df <- df[!rVars]

#Save variables used.
vars <- names(df)


# Creates test and partition data.
inTrain <- createDataPartition(y = df$classe, 
            p = 0.80, list = FALSE)

training <- df[inTrain,]
testing <- df[-inTrain,]
```

The next step is to train the predictors using the $training$
data set obtained above. Since the 
target data is discrete, I consider that linear models
are not adequate for the task at hand. I chose to use
tree different models: $modFitT$ which is based on a simple tree prediction algorithm,
$modFitGBM$ which is a boosted tree prediction model and $modFit$, a combination
of the first two using a decision tree structure that weights both models in order to 
(potentially) control over-fitting.



```r
# Fits a tree model.
modFitT <- train (classe ~ ., 
    method = "rpart", data = training)

# Fits a boosting (with trees) model.
modFitGBM <- train (classe ~ ., verbose = NULL,
    method = "gbm", data = training)
```

```
## Loading required package: plyr
```

The following code computes a predictor ($modFit$) combining
$modFitT$ and $modFitGBM$.


```r
# Vectors of predictions for Combining predictors.
tPred <- predict(modFitT, newdata = training)
gbmPred <- predict(modFitGBM, newdata = training)

# New data frame.
dfComb <- data.frame(tPred, gbmPred, 
                classe = training$classe)

#Combined predictor using trees
modFit <- train (classe ~ ., method = "rpart", data = dfComb)
```

## Model Cross-Validation.

Next I use the  $testing$ data frame for the validation of the computed models,
choosing the more effective one as measured by accuracy, which is defined as the
quotient of successful predictions between the total number of measures.


```r
tPred <- predict(modFitT, newdata = testing)
gbmPred <- predict(modFitGBM, newdata = testing)
dfComb <- data.frame(tPred, gbmPred, 
                classe = testing$classe)
comPred <- predict(modFit, newdata = dfComb)

accuracy(testing$classe, tPred)
```

```
## [1] 0.5592659
```

```r
accuracy(testing$classe, gbmPred)
```

```
## [1] 0.9961764
```

```r
accuracy(testing$classe, comPred)
```

```
## [1] 0.6594443
```

Given that the unweighed $gbmPred$ variable has the best accuracy by a wide margin,
this is the model chosen going forward. The expected effectivity of this model is
high at 0.996, which yiels an **error rate** of
0.004 or 0.4%.


## Test file.

Finally, we apply the prediction algorithm 
to the provided test file (*pml-testing.csv*).


```r
test <- read.csv("pml-testing.csv",  
        na.strings = c("NA", "", "#DIV/0!"))
        
testPred <- predict(modFitGBM, newdata = test)

testPred
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

I report that the proposed machine learning model got 20/20 of correct answers, 
obtaining a 100% effectivity rate on this
small test file, which is in line with the expected 0.4% error
rate. 



