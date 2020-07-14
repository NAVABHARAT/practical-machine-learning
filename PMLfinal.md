---
title: "Practical Machine Learning Final Project"
author: "Navabharat Reddy"
date: "July 14th 2020"
output: html_document
---
#1.Overview
The goal of this project is to create a prediction model for how well a person performs barbell lifts, based on data from accelerometers on the persons belt, forearm, arm and dumbell.  

Data for the model was collected from 6 participants, that were asked to perform barbell lifts correctly and incorrectly in 5 different ways. Data comes from this source: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har
```{r, echo=FALSE, message=FALSE}
library(caret)
```

#2.Data preprocessing

First, data is downloaded and cleaned. Some records have missing values of the form  "#DIV/0!" or simply no value "", these values are replaced by "NA". The first seven columns of each file don't include predictors, so they are discarded. 

Each column that has at least one NA is discarded as a predictor. 

A validation set is created with 30% of the records of training set. 
```{r, echo=TRUE}
trainingset_url <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testingset_url <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
train_name<-"pmltrain.csv"
test_name<-"pmltest.csv"
#Only download files if they aren't already in the working directory
if (!file.exists(train_name)){
  download.file(trainingset_url,destfile=train_name)
}
if (!file.exists(test_name)){
  download.file(testingset_url,destfile=test_name)
}
#An exploration of the training and testing set shows missing values represented as this strings
na_strings<-c("#DIV/0!","","NA")
#Create dataframes with NAs as "NA"
trainingset<-read.csv(train_name, na.strings=na_strings)
testingset<-read.csv(test_name,na.strings=na_strings)
#First seven are not predictors
trainingset<-trainingset[,8:dim(trainingset)[2]]
testingset<-testingset[,8:dim(testingset)[2]]
#Only keep columns without missing values
trainingset<-trainingset[,-(as.vector(which(apply(trainingset,2,function(x) any(is.na(x))))))]
#Create validation set
trainreg<-createDataPartition(trainingset$classe,p=0.70, list=FALSE)
validateset<-trainingset[-trainreg,]
trainingset<-trainingset[trainreg,]
```

#3. Model training
A Random forest will be trained, with 5-fold cross validation. Cross validation assesses how the results will generalize to an independent test set. Random Forest method has a very good performance againts the best supervised learning algorithms. 

```{r, echo=TRUE}
model<-train(classe~.,data=trainingset, method="rf", trControl=trainControl(method="cv",5),ntree=50)
model
```

#4.Performance evaluation

Using the validation set, we get a confusion Matrix, that show the Accuracy of the model predictions. 
```{r, echo=TRUE}
prediction<-predict(model,validateset)
confusionMatrix(validateset$classe,prediction)
```
The accuracy obtained from the model is 99% 

#5.Predictions with the test set
The model is used to predict classes with testing set.
```{r echo=TRUE}
classes<-predict(model,testingset)
classes
```
Class name interpretation is as follows:
A-Exactly according to specification, this is the correct way to do the exercise.
B-Throwing elbows to the front
C-Lifting the dumbell only halfway
D-Lowering the dumbell only halfway
E-Throwing the hips to the front
