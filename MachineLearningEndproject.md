    download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", destfile = "pml-training.csv")

    download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", destfile = "pml-testing.csv")

    training_orig <-read.csv("pml-training.csv",na.strings = c("NA","")) 
    # str(training)

    useless <- vector()
    count <- 1

    for (i in 1:ncol(training_orig)){  # deleting every variable with more than 90% NAs
         
            if(mean(is.na(training_orig[,i]))>0.9){
                    useless[count] <- i
                    count <- count +1
            }   
            
    }

    training <- training_orig[,-useless]


    testing_orig <-read.csv("pml-testing.csv",na.strings = c("NA","")) 
    testing <- testing_orig[,-useless]

After loading the data str() was used to analyse the dataset. A lot of
variables were mostly NA. Those were the percentage of NA exceeded 90%
were deleted from the dataset for both the training and the testing
dataset. In the next step variables that were not measurements from the
devices were deleted (e.g. name of subject, timestamps). While those
might be useful to predict within the sample (e.g. one task classe might
be performed continuous for a certain period of time, so times might
correlate with task classe), it would not be useful for predicting
outcomes not taken dircetly from this experiment:

    #delete timestamps, names etc. 
    training <- training[,8:60]     # wenn aktiv outcome = col:53, sonst col:60
    testing <- testing[, 8:60]      # wenn aktiv outcome = col:53, sonst col:60

Before training the model the data was split into a training (70%) and a
test set (30%) for cross validation.

Two models were tested: random forest and gradient boosting machine with
all remaining varibales included. These model types were chosen since
they usuallly perform well in prediction, since in this task we are not
interested in interpretability, their disadvantage is not relevant in
this context.

In sample testing showed an accuracy of 1 for the random forest model
and 97.5% for the gradient boosting machine model.

Crossvalidation to get out of sample error was performed on the earlier
generated testing data (pre\_test). Out of sample error for the random
forest model was 0.06% and 3.5% for the gradient boosting machine model.
For both in sample and out of sample error the random forest model is
superior to the gradient boosting machine model. To sum up both models
show an adequate prediction accuracy to solve the task of predicting 20
out of sample samples (considering the 80% passing condition of the
quiz).

    set.seed(2710)

    suppressMessages(library(caret))

    ## Warning: package 'caret' was built under R version 3.5.3

    ## Warning: package 'ggplot2' was built under R version 3.5.1

    suppressMessages(library(randomForest))

    ## Warning: package 'randomForest' was built under R version 3.5.3

    inTrain <- createDataPartition(training$classe, p = 0.7, list = FALSE)
    train_1 <- training[inTrain,]
    pre_test <- training[-inTrain,]


    rand_forest  <- randomForest(classe ~., data=train_1)
    gbm <- train(x = train_1[,-53], y = train_1[,53], method = "gbm", verbose = FALSE) 

    # in sample prediction:
    pred_test_rf <- predict(rand_forest, train_1[,-53])
    pred_test_gbm <- predict(gbm, train_1[,-53])

    mean(pred_test_rf == train_1[,53])

    ## [1] 1

    mean(pred_test_gbm == train_1[,53])

    ## [1] 0.9759773

    # cross validation:
    pred_test_rf <- predict(rand_forest, pre_test[,-53])
    pred_test_gbm <- predict(gbm, pre_test[,-53])

    mean(pred_test_rf == pre_test[,53])

    ## [1] 0.9967715

    mean(pred_test_gbm == pre_test[,53])

    ## [1] 0.9682243

In this code junk the quiz sample is calculated:

    # some variables were read in as integer instead of double type in the testing set; here it is changed to double
    x <- sapply(training, typeof)
    y <- sapply(testing, typeof)
    testing[,x != y] <- lapply(testing[,x != y], as.numeric) 

    prediction <- predict(gbm, testing[,-53])

    prediction2 <- predict(rand_forest, testing[,-53])
    prediction

    ##  [1] B A B A A E D B A A B C B A E E A B B B
    ## Levels: A B C D E

    prediction2

    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    ##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    ## Levels: A B C D E

Both models agree in their prediction, which enhaces the probability of
the predictions being correct. A disagreement in the predictions would
have hinted at the necessity to further improve the models.
