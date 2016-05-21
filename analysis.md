#Report for the final project in the Practical Machine Learning Course on Coursera

## Reading the Data
Read the training and testing files, and split the training data into training and validation sets.

    library(caret)
    data<-read.csv("pml-training.csv", stringsAsFactors=FALSE)
    inTrain <- createDataPartition(data$classe, p=0.70, list=FALSE)
    training <- data[inTrain,]
    validation <- data[-inTrain,]
    
    testing<-read.csv("pml-testing.csv")

## Formatting data
Basic exploration of the data shows that most columns contains mainly NA values.
The first columns contain data and time information, which should be irrelevant to the analysis. We remove the time columns, and the columns containing NA values. 53 columns variables remain in the dataset.

    ## EXTRACT CLASSE INFORMATION
    data<-subset(training,  select = -c(classe))

    ## REMOVE TIME INFORMATION
    data<-data[,-c(1:5)]

    ## SAVE COLUMN NAMES
    datnames<-names(data)
  
    ## CONVERT REMAINING VALUES TO NUMERIC
    temp<-lapply(data, as.numeric)
    data<-data.frame(matrix(unlist(temp), ncol=ncol(data)))

    ## REMOVE COLUMNS CONTAINING NA VALUES
    colindices<-which(colSums(is.na(data))==0)
    data<-data[,colindices]
    names(data)<-datnames[colindices]
    data<-cbind(classe=training$classe, data)

## Model fitting

The model used is Random Forest with cross-validation. Parallelisation is also used to speed up the computation time. Cross-validation and parallelisation options ae defined using the trainControl command.

    ## PARALLEL AND CROSS-VALIDATION OPTIONS
    library(parallel)
    library(doParallel)
    cluster <- makeCluster(detectCores() - 1) # convention to leave 1 core for OS
    registerDoParallel(cluster)
    fitControl <- trainControl(method = "cv",
                               number = 10,
                               allowParallel = TRUE)

    ## TRAINING  
    modelFit<-train(classe~., method="rf", data=data, trControl=fitControl)

    ## SAVE THE MODEL
    save(modelFit,file="rfFit.RData")


## Test the model on the validation dataset

The validation dataset is used to predict the out-of-sample accuracy of the model. As such, we preprocess the validation dataset in the same way as the training dataset, and we apply the model to it.

    temp<-lapply(subset(validation, select=-c(classe)), as.numeric)
    valclasse<-validation$classe
    valnames<-names(temp)
    validation<-data.frame(matrix(unlist(temp), ncol=ncol(validation)-1))
    names(validation)<-valnames
    validation<-cbind(classe=valclasse, validation)

    ## LOAD THE MODEL (IF NOT ALREADY IN MEMORY)
    load(file = "rfFit.RData")

    ## PREDICTIONS FOR VALIDATION SET 
    valpreds<-predict(modelFit, newdata=validation)

We use the command confusionMatrix to get detailed information on the model accuracy on the validation set.

    confusionMatrix(valpreds, validation$classe)

This gives us an estimate of the out-of sample accuracy of: 0.9993
Confusion matrix below.

|            | Reference |    A |    B |    C |    D |    E |
|------------|-----------|-----:|-----:|-----:|-----:|-----:|
| Prediction |           |      |      |      |      |      |
| A          |           | 1674 |    1 |    0 |    0 |    0 |
| B          |           |    0 | 1138 |    1 |    0 |    0 |
| C          |           |    0 |    0 | 1025 |    1 |    0 |
| D          |           |    0 |    0 |    0 |  963 |    1 |
| E          |           |    0 |    0 |    0 |    0 | 1081 |
