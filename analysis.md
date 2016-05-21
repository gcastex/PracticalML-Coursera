#Report for the final project in the Practical Machine Learning Course on Coursera

## Reading the Data
Read the training and testing files, and split the training data into training and validation sets.

    library(caret)
    data<-read.csv("pml-training.csv", stringsAsFactors=FALSE)
    
    inTrain <- createDataPartition(data$classe, p=0.70, list=FALSE)
    training <- data[inTrain,]
    validation <- data[-inTrain,]
    
    testing<-read.csv("pml-testing.csv")

