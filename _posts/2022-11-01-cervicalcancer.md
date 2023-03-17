---
title: "Cervical Cancer Risk Factors Analysis with Machine Learning Models
"
layout: post
date: 2020-12-11
tag:
- Cervical cancer risk factors dataset
- Artificial Neural Network
- Random Forest
- Logistic Regression
- Naive Bayes
- Ensembl models
- Machine learning
- CRISP-DM

projects: true
description: "Cervical Cancer Risk Factors Analysis with Machine Learning Models"
---

**Boston, MA. 2022**

---


# Libraries

```{r}
library(skimr)
library(dplyr)
library(tidyr)
library(klaR)
library(neuralnet)
library(Hmisc)
library(nnet)
library(e1071)
library(ROSE)
library(rpart.plot)
library("FactoMineR")
library(randomForest)
library(crosstable)
library(skimr)
library(klaR)
library(caret)
library(dplyr)
library(psych)
library(ipred)
```

# Data Aquisition

```{r}
url <- "https://archive.ics.uci.edu/ml/machine-learning-databases/00383/risk_factors_cervical_cancer.csv"
cervicalrisk.df <- read.csv(url)
head(cervicalrisk.df)
```

The data is loaded from the url and saved in a dataframe named as cervicalrisk.df. The dataset is a real life cervical cancer risk factors dataset which was collected from 'Hospital Universitario de Caracas' in Caracas, Venezuela. The dataset has various features like demographic information, lifestyle and historic medical records which could all point to being risks to cervical cancer. The dataset has 4 target features which are 4 different testing techniques of cervical cancer which are Hinselmann,  Schiller, Citology and Biopsy.

# Data Exploration

```{r}
str(cervicalrisk.df)
```

The structure of the dataset is determined using the str() function. We can see from the output above that the dataset 858 rows and 36 columns. The dataset has a lot of numeric columns which are of "chr" datatype which needs to be converted.


```{r, warning=FALSE}
cervicalrisk.df <- data.frame(lapply(cervicalrisk.df, as.numeric)) # Converting all columns to  numeric datatype
```

```{r
cervicalrisk.df <- cervicalrisk.df[ , -which(names(cervicalrisk.df) %in% c("Smokes..years.","Smokes..packs.year.", "Hormonal.Contraceptives..years.", "IUD..years.", "STDs.condylomatosis", "STDs.cervical.condylomatosis", "STDs.vaginal.condylomatosis", "STDs.vulvo.perineal.condylomatosis", "STDs.syphilis", "STDs.pelvic.inflammatory.disease", "STDs.genital.herpes", "STDs.molluscum.contagiosum", "STDs.AIDS", "STDs.HIV", "STDs.Hepatitis.B", "STDs.HPV", "STDs..Number.of.diagnosis", "Schiller", "Citology", "Biopsy"))] # Removing columns that are not relevant 
```

Looking at the features in the dataset, a lot of columns seemed redundant and I have removed them by keeping only one important column regarding each redundant feature. For example, Smokes..years , Smokes..packs.year, and Smokes. seemed to be very repetitive and I have kept only the Smokes. column because it says whether or not  the individual is smoking. Similarly, Contraceptive..years. and IUD..years. and so on. BAsed on my domain knowledge, I also removed the multipe STD columns because having any type of STD (which is implied by STDs column) itself is a risk factor for cervical cancer. Further, I have converted all the columns to numeric.


```{r}
summary(cervicalrisk.df)
```

Further, I summarized the dataset with the summary() function to get a glimse of the summary statistics of all the columns like min, max, mean, quartiles and NAs. We can  see that there are numerous NAs in most of the columns. Moving forward, I refine the dataset and summarise it again to get a better glimpse.

```{r}
boxplot(cervicalrisk.df, col = "blue", ylab ="Value distribution with respect to the interquantile ranges", xlab ="Features")
```

The above plot is a boxplot for all the features in the dataset which shows the values of the features The ends of the boxes in the plot for each column shows the interquantile ranges for that column. The dots that appear above or below are the values that fall ouside this range or deviates too much from the mean and these are the outliers of those respective columns. 

```{r}
# Pairs panels plot 
pairs.panels(cervicalrisk.df[1:8])
pairs.panels(cervicalrisk.df[9:16])
```
I used the pairs panels function which shows a scatterplot of matrices. The diagonals show the histogram distribution of each feature and the scatterplots on the lower side of the diagonal. The numbers on the other side of the diagonal is the correlation values of all the columns. The detailed correlation plot and correlation analysis is done below.

```{r}
hist.data.frame(cervicalrisk.df)
```

I constructed the histograms for some of the features in the dataset. Let us overlay a normal curve to some of the plot and observe their distribution.

```{r}
histplot <- hist(cervicalrisk.df$Age, breaks = 30, density = 10, col = "lightgray", xlab = "Age") 
x_line <- seq(min(cervicalrisk.df$Age), max(cervicalrisk.df$Age), length = 40) 
y_line <- dnorm(x_line, mean = mean(cervicalrisk.df$Age), sd = sd(cervicalrisk.df$Age)) 
y_line <- y_line * diff(histplot$mids[1:2]) * length(cervicalrisk.df$Age) 

lines(x_line, y_line, col = "black", lwd = 2)
```

```{r}
h <- hist(cervicalrisk.df$First.sexual.intercourse, breaks = 30, density = 10, col = "lightgray", xlab = "First sexual intercourse") 
xfit <- seq(min(cervicalrisk.df$First.sexual.intercourse, na.rm = TRUE), max(cervicalrisk.df$First.sexual.intercourse,  na.rm = TRUE), length = 40) 
yfit <- dnorm(xfit, mean = mean(cervicalrisk.df$First.sexual.intercourse, na.rm =  TRUE), sd = sd(cervicalrisk.df$First.sexual.intercourse, na.rm = TRUE)) 
yfit <- yfit * diff(h$mids[1:2]) * length(cervicalrisk.df$First.sexual.intercourse) 

lines(xfit, yfit, col = "black", lwd = 2)
```

The First sexual intercourse distribution is just fairly normally distributed and looks slightly right skewed. The age plot is very right skewed. Skewed data occurs because of lower or upper bounds on the data that have a are skewed on two opposite ends. 

The correlation values for the entire dataset is displayed as a correlation matrix as well as a correlation plot above. Values of correlation ranges from -1 to 1. -1 indicates the maximum negative correlation and +1 indicates the maximum positive correlation. The more extreme the correlation coefficients closeness to -1 or 1, the stronger is the relationship. A correlation which is close to 0 implies that the two variables are independent and if one variable increases, it does not have any impact on the other. In the corplot above, we can see that some of the correlation  values are in the ranges of 0.8-0.9. For example, Dx.cancer and Dx.HPV have a correlation of 0.89. This shows that there is a high positive correlation between these two variables. On the other hand, Hormonal.contraceptives and STDs have -0.03 which is very close to 0 and implies that they are almost independent to each other. Having a high correlation, typically above 0.7 indicates collinearity in the dataset which can dd redundancy in the data because two predictors are indicating the same information about the response variables and could be unreliable in models.

# Data Cleaning & Shaping

```{r}
# Determining number of NAs
sapply(cervicalrisk.df, function(x) sum(is.na(x)))
sapply(cervicalrisk.df, function(x) sum(is.nan(x)))
```

Here I am determining the number of NA values in the dataset. The dataset has NA value occurrences in most of the columns and no NAN in  any columns. Let us impute it.

```{r}
# Finding percentage of NA in each column
(colMeans(is.na(cervicalrisk.df)))*100
```

Above, I determined the percentage of NAs in the columns. We can see that STDs..Time.since.first.diagnosis and STDs..Time.since.last.diagnosis have over 91% of NA values and hence this columns needs to be removed. The other NA values on the columns are imputed.

```{r}
# Removing columns with more than 90% NAs
cervicalrisk.df = subset(cervicalrisk.df, select = -(STDs..Time.since.first.diagnosis))
cervicalrisk.df = subset(cervicalrisk.df, select = -(STDs..Time.since.last.diagnosis))
```

```{r}
# Categorical columns
cervicalrisk.df.cat <- subset(cervicalrisk.df, select = c(5,6,7,8,10,11,12,13,14))

# Continuous columns
cervicalrisk.df.con <- subset(cervicalrisk.df, select = c(1,2,3,4,9))
```

To impute the NAs in categorical columns and continuous column accordingly, I seperate out all the categorical and continuous variables above into two sepearate dataframes.

```{r}
# Replacing NA by mode in categorical dataframe
mode_val <-  function(x){
    distinct_values <- unique(na.omit(x))
    distinct_tabulate <- tabulate(match(x, distinct_values))
    distinct_values[which.max(distinct_tabulate)]
}
cervicalrisk.df.cat <- cervicalrisk.df.cat %>% mutate(across(everything(), ~replace_na(.x, mode_val(.x))))
```

For the categorical columns, I imputed all the NAs by the mode of that particular column. To determine the mode, I have a function mode_val and have used the dplyr package to mutate all columns and impute NAs by the mode.

```{r}
sapply(cervicalrisk.df.cat, function(x) sum(is.na(x)))
```

Let us check for the NAs again. We can now see that all NAs from the categorical columns have been removed.

```{r}
# Replacing NA by median in continuous dataframe
cervicalrisk.df.con <- cervicalrisk.df.con %>% mutate(across(everything(), ~replace_na(., median(., na.rm=TRUE))))
```

Moving on to the continuous columns, I have replaced the NAs by the modian of that particular column.

```{r}
sapply(cervicalrisk.df.con, function(x) sum(is.na(x)))
```

Now, let us check again. There are 0 NAs everywhere.

```{r}
# Join the two dataframes
cervicalrisk.df <- cbind(cervicalrisk.df.con, cervicalrisk.df.cat)
head(cervicalrisk.df)
```

I am now joining the two dataframes back together. This dataframe now has no NAs.

```{r}
# Outlier analysis

# Function to get all outliers in the dataset 
outliers <- function(x) {
  stddev <- sd(x)
  avg <- mean(x)
  lower <- avg-(2*stddev)
  upper <- avg+(2*stddev)
  x > upper | x < lower
}

# subset outliers to see all outliers in dataset
subset_outliers <- function(df, cols = names(df)) {
  for (col in cols) {
    df <- df[!outliers(df[[col]]),]
  }
  df
}

cervical.without.outliers <- subset_outliers(cervicalrisk.df, c(1:14))
dim(cervical.without.outliers)
cervical.only.outliers <- anti_join(cervicalrisk.df, cervical.without.outliers)
dim(cervical.only.outliers)
```

Moving on, I have performed a detailed outlier analysis in the dataset. Above, I created a function "outliers" which calculates all the values that are 2 standard deviations above or below the mean. Further, I used the subset_outliers function to get a dataframe without the outliers. I obtained the dataframe without outliers and we can see that this datframe has 358 columns. The outliers and their corresponding rows of each column is in this dataframe.

```{r}
# Let us replace the outliers with median
replace <- function(x) {
  stddev <- sd(x)
  avg <- mean(x)
  lower <- avg-(2*stddev)
  upper <- avg+(2*stddev)
  x[x < lower | x > upper] <- median(x)
  x
}

cervicalrisk.df[1:5] <- lapply(cervicalrisk.df[1:5], replace)
```

The "replace" function replaces all the above outliers with  the median of that respective column. I applied it to all the continuous columns, and the categorical columns are replaced of outliers with the median of the column. 

```{r}
# Correlation analysis
cor(cervicalrisk.df)
corPlot(cervicalrisk.df)
```

The correlation values for the entire dataset is displayed as a correlation matrix as well as a correlation plot above. Values of correlation ranges from -1 to 1. -1 indicates the maximum negative correlation and +1 indicates the maximum positive correlation. The more extreme the correlation coefficients closeness to -1 or 1, the stronger is the relationship. A correlation which is close to 0 implies that the two variables are independent and if one variable increases, it does not have any impact on the other. In the corplot above, we can see that some of the correlation  values are in the ranges of 0.8-0.9. For example, Dx.cancer and Dx.HPV have a correlation of 0.886. This shows that there is a high positive correlation between these two variables. On the other hand, Hormonal.contraceptives and STDs have -0.04 which is very close ro 0 and implies that they are almost independant to each other. Having a high correlation, typically above 0.7 indicates collinearity in the dataset which can dd redundancy in the data because two predictors are indicating the same information about the response variables and could be unreliable in models.

```{r}
# PCA on continuous variables
cervical.df.pca <- prcomp(cervicalrisk.df[1:5], scale = TRUE)
cervical.df.pca
```

The PCA analysis lets us see the overall shape of the dataset, and identifies the samples which are similar as well as very different from each other. This allows us to identify sample groups that are similar and work out which variables creates the difference between these groups. PCA is typically done to perform feature reduction and is particularly handy when working with wide datasets. The values in each principal component shows the deviation of these groups.

```{r}
set.seed(1000)

# Normalization of data
normalize <- function(x) {
  (x - min(x)) / (max(x) - min(x))
}

cervicalrisk.df.norm <- cervicalrisk.df
cervicalrisk.df.norm[1:5] <- as.data.frame(lapply(cervicalrisk.df[1:5], normalize))

split_norm <- sample(c(rep(0, 0.7 * nrow(cervicalrisk.df.norm)), rep(1, 0.3 * nrow(cervicalrisk.df.norm))))
cervical.df.norm.train <- cervicalrisk.df.norm[split_norm == 0,]
cervical.df.norm.test <- cervicalrisk.df.norm[split_norm == 1,]

# Checking the distribution of train and test
prop.table(table(cervical.df.norm.train$Hinselmann))
prop.table(table(cervical.df.norm.test$Hinselmann))

# Oversampling  the data
cervical.df.norm.train <- ovun.sample(Hinselmann~., data=cervical.df.norm.train, method = "over")$data
```

The dataset is then normalized by min-max normalization. I created a function for the min-max normalization and applied it to all the continuous variables in the dataset. This dataset is named as cervicalrisk.df.norm. I further split the normalized data into training and test sets. I will be using this normalized dataset for my ANN model. Further, I checked the distribution of the target variable in the train and test sets. We can see  from the output that the target variables are proportionally distributed in both the sets. It is also evident that the target data is very non-uniformly undersampled. I made use of the ovun.sample() function to re-sample the training data and oversample it to distribute it evenly such that the column is not predominantly filled with 0s.   

# Model Construction

## Naive Bayes
```{r}
set.seed(1000)

# Split the data into training and test sets
cervical.df.NB <- cervicalrisk.df

cervical.df.NB$Age <- as.numeric(cut(cervical.df.NB$Age ,c(0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95, 100)))

cervical.df.NB$Number.of.sexual.partners <- as.numeric(cut(cervical.df.NB$Number.of.sexual.partners ,c(0, 5, 10, 15, 20, 25, 30)))


cervical.df.NB$First.sexual.intercourse <- as.numeric(cut(cervical.df.NB$First.sexual.intercourse ,c(-2, 2, 6, 10, 14, 18, 22, 26, 30, 34)))

cervical.df.NB$Num.of.pregnancies <- as.numeric(cut(cervical.df.NB$Num.of.pregnancies ,c(-2, 0, 2, 4, 6, 8, 10, 12)))

cervical.df.NB$STDs..number. <- as.numeric(cut(cervical.df.NB$STDs..number. ,c(-1, 5, 10, 15, 20)))
```

Here, the continuous columns are converted to categorical by binning (creating bin thresholds). 

```{r}
set.seed(1000)
# Splitting into training and test set
split <- sample(c(rep(0, 0.7 * nrow(cervical.df.NB)), rep(1, 0.3 * nrow(cervical.df.NB))))
cervical.df.NB.train <- cervical.df.NB[split == 0,]
cervical.df.NB.test <- cervical.df.NB[split == 1,]

# Over sampling the data
cervical.df.NB.train <- ovun.sample(Hinselmann~., data=cervical.df.NB.train, method = "over")$data
```

Now, I have sampled this dataset with binned columns into training and test sets. This dataset also has been oversampled using ovun.sample(). I will be using this dataset for Naive Bayes and logistic regression which performs well with categorical dataset.

```{r}
# Building the naive bayes model and predicting the test set 
set.seed(1000)
model.NB <- naiveBayes(Hinselmann~., data = cervical.df.NB.train)
pred.NB <- predict(model.NB, cervical.df.NB.test)
```

Here, I have built the naive bayes model using naiveBayes() function. The pred.NB has the prediction of the test set from the Naive Bayes. 

## Logistic regression
```{r}
# Building the logistic regression model and predicting the test set 
set.seed(1000)
model.LR <- glm(Hinselmann ~ ., data = cervical.df.NB.train, family = "binomial")
pred.LR <-  predict(model.LR, cervical.df.NB.test, type="response")
pred.LR <- ifelse(pred.LR < 0.5, 0, 1)
```

Similarly, the logistic regression model is built using glm() function. The family is set to binomial. The prediction is determined for the test set. The prediction from this model is not 0s and 1s. So,  I added a thresh old to convert all values below 0.5 to 0 and everything above to 1. 

## ANN
```{r}
# Building the ANN model and predicting the test set 
set.seed(1000)
model.ANN <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=5)
pred.ANN <- predict(model.ANN, cervical.df.norm.test)
pred.ANN <- ifelse(pred.ANN < 0.5, 0, 1)
```

The ANN model is build with the nnet() function and just like logistic regression, the threshold is set to convert values to 0s and 1s.

# Model evaluation

```{r}
# Metrics function

calculate.metrics <- function(tab) {
  TP <- tab$table[1][1]
  FP <- tab$table[2][1]
  FN <- tab$table[3][1]
  TN <- tab$table[4][1]
  
  precision <- TP / (TP + FP)
  recall <- TP / (TP + FN)
  f1 <- (2*precision*recall)/(precision + recall)
  metric.values <- paste("Precision = ", precision, "Recall = ", recall, "F1 score = ", f1, sep="\n")
  return(metric.values)
}  
```

I have created a function that calculates the metrics like precision, recall and F1 score for each confusion matrix. Precision, which is also known as the positive predictive value is the proportion of positive datapoints that are truly positive- which means that a model predicts the positive class correctly. This is determined by TP/(TP+FP). Recall is a parameter to measure of how complete the results are. This is calculated by TP/(TP+FN). The F1 score which is also known as F measure is the measure of performance of the model based on the combination of precision and recall into a single value.

## Naive Bayes
```{r}
# Confusion matrix
set.seed(1000)
tab.NB <- confusionMatrix(as.factor(pred.NB), as.factor(cervical.df.NB.test$Hinselmann))
```

The confusion matrix for the naive bayes model is determined above. We can see that the accuracy of this model is 93.4% for. 

```{r}
# Precision, recall and F1 score
metrics.NB <- calculate.metrics(tab.NB)
cat(metrics.NB)
```

We can see that the Naive Bayes model has a pretty good accuracy and recall. The precision seems to be less for the model and F1 score at a lower range too. Let us see how the other models performed.

## Logistic regression
```{r}
# Confusion matrix
set.seed(1000)
tab.LR <- confusionMatrix(as.factor(pred.LR), as.factor(cervical.df.NB.test$Hinselmann))
```

The accuracy for the logistic regression model is 72.37%.

```{r}
# Precision, recall and F1 score
metrics.LR <- calculate.metrics(tab.LR)
cat(metrics.LR)
```

The precision, recall as well as F1 score all seems to be high for this model unlike the naive bayes model.

## ANN
```{r}
# Confusion matrix
set.seed(1000)
tab.ANN <- confusionMatrix(as.factor(pred.ANN), as.factor(cervical.df.NB.test$Hinselmann))
```

The artificial neural network has an accuracy of 83.66%.

```{r}
# Precision, recall and F1 score
metrics.ANN <- calculate.metrics(tab.ANN)
cat(metrics.ANN)
```

The precision, recall and F1 score is also very good for this model. 

When all the three metrics are compared, we can see that just the accuracy is very high for the naive bayes model while the F1 score and precision is very poor. Logistic regression and ANN has a good accuracy percentage and with very good precision, recall and F1 scores. The ANN model seems to be doing better than the logistic regression model in this case with an accuracy of 83.66%. 

# Changing parameters of the model to improve it - Hyperparameter tuning

# Naive Bayes

```{r}
set.seed(1000)
model.NB.new <- naiveBayes(Hinselmann~., data = cervical.df.NB.train, laplace = 1)
pred.NB.new <- predict(model.NB, cervical.df.NB.test)
confusionMatrix(as.factor(pred.NB.new), as.factor(cervical.df.NB.test$Hinselmann))
```

To improve the model, I added a laplace parameter to the model. Surprisingly, this seemed to make no difference in the models accuracy. No other values for laplace also changed the models accuracy.

# Logistic regression

```{r}
# Determine the most important features
set.seed(1000)
varImp(model.LR) %>% arrange(desc(Overall))

# Rebuild the model
model.LR.new <- glm(Hinselmann ~ STDs+IUD+Smokes+Age+Hormonal.Contraceptives+Dx.Cancer+Dx, data = cervical.df.NB.train, family = "binomial")
pred.LR.new <-  predict(model.LR.new, cervical.df.NB.test, type="response")
pred.LR.new <- ifelse(pred.LR.new < 0.5, 0, 1)  
confusionMatrix(as.factor(pred.LR.new), as.factor(cervical.df.NB.test$Hinselmann))
```

Moving on, for the logistic regression model, I used the varImp() function to the logistic regression model to determine the importance given to each feature. Arranging it in descending order lets me see the most significant feature as STDs and so on. I have used the top 6 best features to re-train the model. The model seemed to have only slightly improved to 73.15%.

## ANN

```{r}
set.seed(1000)
model.ANN.new1 <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=1)
pred.ANN.new1 <- predict(model.ANN.new1, cervical.df.norm.test)
pred.ANN.new1 <- ifelse(pred.ANN.new1 < 0.5, 0, 1)
confusionMatrix(as.factor(pred.ANN.new1), as.factor(cervical.df.NB.test$Hinselmann))

model.ANN.new2 <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=2)
pred.ANN.new2 <- predict(model.ANN.new2, cervical.df.norm.test)
pred.ANN.new2 <- ifelse(pred.ANN.new2 < 0.5, 0, 1)
confusionMatrix(as.factor(pred.ANN.new2), as.factor(cervical.df.NB.test$Hinselmann))

model.ANN.new3 <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=3)
pred.ANN.new3 <- predict(model.ANN.new3, cervical.df.norm.test)
pred.ANN.new3 <- ifelse(pred.ANN.new3 < 0.5, 0, 1)
confusionMatrix(as.factor(pred.ANN.new3), as.factor(cervical.df.NB.test$Hinselmann))

model.ANN.new4 <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=4)
pred.ANN.new4 <- predict(model.ANN.new4, cervical.df.norm.test)
pred.ANN.new4 <- ifelse(pred.ANN.new4 < 0.5, 0, 1)
confusionMatrix(as.factor(pred.ANN.new4), as.factor(cervical.df.NB.test$Hinselmann))

model.ANN.new5 <- nnet(Hinselmann ~ .,  data = cervical.df.norm.train, size=6)
pred.ANN.new5 <- predict(model.ANN.new5, cervical.df.norm.test)
pred.ANN.new5 <- ifelse(pred.ANN.new5 < 0.5, 0, 1)
confusionMatrix(as.factor(pred.ANN.new5), as.factor(cervical.df.NB.test$Hinselmann))
```

I re-trained the ANN model with 5 different cases for size - 1,2,3,4 and 6 apart from the original where size was 5. The model with size = 5 seems to perform the best among all these cases with the highest accuracy.

# Hyperparameter tuning

# Model Tuning & Performance Improvement

# Ensemble model 

```{r}
# Bagging
set.seed(1000)
model.bag <- bagging(Hinselmann ~ ., data = cervical.df.NB.train, nbagg = 30)
pred.bag <- predict(model.bag, cervical.df.NB.test)
pred.bag <- ifelse(pred.bag < 0.5, 0, 1)
confusionMatrix(as.factor(pred.bag), as.factor(cervical.df.NB.test$Hinselmann))
```

I created an ensemble bagging model to see if there is any improvement compared to the above 3 models. The bagging model has an accuracy of 62.65% which is not better than any of our previous models.

# Randomforest
```{r}
set.seed(1000)
model.rf <- randomForest(Hinselmann ~ ., data = cervical.df.NB.train)
pred.rf <- predict(model.rf, cervical.df.NB.test)
pred.rf <- ifelse(pred.rf < 0.5, 0, 1)
confusionMatrix(as.factor(pred.rf), as.factor(cervical.df.NB.test$Hinselmann))
```

The random forest model seems to  be doinf much  better than the bagging model at an accuracy of 79.38%.

# Ensemble function combing all 3 models
```{r}
# Ensembl function based on majority voting system

set.seed(1000)
ensembl <- function(input){
  
  pred.NB <- predict(model.NB, input)
  pred.LR <-  predict(model.LR, input)
  pred.ANN <- predict(model.ANN, input)
  vote <- ifelse(pred.NB == 0, 0, 0)
  # create a majority voting system
  return (ifelse(vote == 0 & pred.LR == 0,0,
            ifelse(vote == 0 & pred.ANN ==0,0,
            ifelse(pred.LR == 0 & pred.ANN == 0,0,1))))
}

pred.ensembl <- ensembl(cervical.df.NB.test)
tab.ensembl <- confusionMatrix(as.factor(pred.ensembl), as.factor(cervical.df.NB.test$Hinselmann))


# Metrics
metrics.ensembl <- calculate.metrics(tab.ensembl)
cat(metrics.ensembl)
```

Here, I have built an ensemble function which combines all 3 models and gives a decision based on a combined majority voting system. This model improved the accuracy significantly to 93.4%. This is a significant improvement in model accuracy which shows that the error rate of the model has reduced considerably from almost 28% to around 4%.

To conclude, after observing trends and completing the analysis of the model comparison, the ANN model did the best when compared to the Naive Bayes and Logistic regression models. Comparison of just the accuracy % is not a reliable criteria since we saw that although the accuracy of the Naive bayes model was the highest, the other metrics like precision, recall and F1 score was very poor for the model. The ANN model did well in all the metrics as well as accuracy. Hence, for this data and this approach, the ANN model seemed to be best fit model to predict cervical cancer based on the risk factors. The ensemble model further enhanced the prediction accuracy to 96.11% as well as the metrics to a really good level. Undersampling of data has an extremely negating effect and upsampling the data significantly helps in improving model performance. Accuracy alone is not entirely reliable to gauge the model performance. A lot of factors play a role in a models performance and allows us the compare different metrics to determine the best performing model with all factors considered. 








