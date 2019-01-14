# Project-2.1-rahul-sharma
Project-2.1-rahul-sharma

1. Use R to do the preprocessing and training. Your final submission should have all the R
code and results coming from Training Model. Show all the accuracy measures and ROC
curve for different probability cut-off. Find the optimal Threshold.

2. Use R for giving all the results and statistics of the Trained Model performance on the Test
Dataset. Your final report should contain all these results.

3. Use Tableau for Visualization Reporting.

a. Perform the training and threshold optimization in tableau. Prepare required
worksheet to perform the same. You should be able to change the probability
threshold using a slider. When changing the threshold, you must show the
accuracy measures accordingly.

b. Once fixed on a particular threshold, you should use R_script to run the testing on
the test data from Tableau.

c. One you get the test result. Show the accuracy measure as well.

d. How the odds ratio on the viz, i.e. show the effect of change of input variables to
probability/chances of churn. You should provide option such that, we can choose
any input variable and change the value to see the effect. Change on numerical
variable would be offset change (some unit change), and categorical variable
would be change from one category to other (Example: If binary then change from 0 to 1.

Ans 1 - >

library(caret)
 
set.seed(442)
training <- twoClassSim(n = 1000, intercept = -16)
testing <- twoClassSim(n = 1000, intercept = -16)
(All the code used here was formatted by Pretty R at inside-R.org.)

In the training set the class frequency breakdown looks like this:

> table(training$Class)
Class1 Class2 
   899    101 

 set.seed(949)
> mod0 <- train(Class ~ ., data = training,
+               method = "rf",
+               metric = "ROC",
+               tuneGrid = data.frame(mtry = 3),
+               trControl = trainControl(method = "cv",
+                                        classProbs = TRUE,
+                                        summaryFunction = twoClassSummary))
> getTrainPerf(mod0)
   TrainROC TrainSens TrainSpec method
1 0.9288911      0.99 0.4736364     rf

The area under the ROC curve is very high, indicating that the model has very good predictive power for these data. Here's a test set ROC curve for this model:


					rog.png
The plot shows the default probability cut off value of 50%. The sensitivity and specificity values associated with this point indicate that performance is not that good when an actual call needs to be made on a sample. 


One of the most common ways to deal with this is to determine an alternate probability cut off using the ROC curve. But to do this well, another set of data (not the test set) is needed to set the cut off and the test set is used to validate it. We don't have a lot of data this is difficult since we will be spending some of our data just to get a single cut off value.


Alternatively the model can be tuned, using resampling, to determine any model tuning parameters as well as an appropriate cut off for the probabilities. 


Our code will use the distance between the current model's performance and the best possible performance and then have train minimize this distance when choosing it's parameters. Here is the code that we use to calculate this:


fourStats <- function (data, lev = levels(data$obs), model = NULL) {
  ## This code will get use the area under the ROC curve and the
  ## sensitivity and specificity values using the current candidate
  ## value of the probability threshold.
  out <- c(twoClassSummary(data, lev = levels(data$obs), model = NULL))
 
  ## The best possible model has sensitivity of 1 and specifity of 1. 
  ## How far are we from that value?
  coords <- matrix(c(1, 1, out["Spec"], out["Sens"]), 
                   ncol = 2, 
                   byrow = TRUE)
  colnames(coords) <- c("Spec", "Sens")
  rownames(coords) <- c("Best", "Current")
  c(out, Dist = dist(coords)[1])
}

Now let's run our random forest model and see what it comes up with for the best possible threshold:


set.seed(949)
mod1 <- train(Class ~ ., data = training,
              ## 'modelInfo' is a list object found in the linked
              ## source code
              method = modelInfo,
              ## Minimize the distance to the perfect model
              metric = "Dist",
              maximize = FALSE,
              tuneLength = 20,
              trControl = trainControl(method = "cv",
                                       classProbs = TRUE,
                                       summaryFunction = fourStats))

The resulting model output notes that:


Tuning parameter 'mtry' was held constant at a value of 3
Dist was used to select the optimal model using  the smallest value.
The final values used for the model were mtry = 3 and threshold = 0.887.

Using ggplot(mod1) will show the performance profile. Instead here is a plot of the sensitivity, specificity, and distance to the perfect model:

