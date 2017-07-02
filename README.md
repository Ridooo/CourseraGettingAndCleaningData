# Getting and Cleaning Data Course Project
Riad Darawish  
16 June 2017  



## Introduction

This document explains the steps of producing a tidy dataset from the original version of [Human Activity Recognition dataset](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones). The final dataset is obtained after performing a number of cleaning operations that includes merging training and test datasets, giving appropriate names for the variables, extracting certain variables and then averaging those variables for each activity and for each subject. The last step is to reshape the summary dataset to produce a tidy the dataset. 

This work deliverers the requirements of the course project assignment of Getting and Cleaning Data Course.

The rest of the document is organized as follow:

Section 1: Loading packages and Reading dataset files.
Section 2: Combing and merging datasets.
Section 3: Extracting measurements on the mean and standard deviation for each measurement.
Section 4: Using descriptive activity names.
Section 5: Labelling the dataset with descriptive variable names.
Section 6: Averaging the variables by activity and subject.
Section 7: Reshape the summary dataset.

## Section 1: Loading packages and Reading dataset files

In this section, *dplyr* and *tidyr* packages are installed and loaded. The former is used to group and summarize a set of variables while the latter is to convert data set to molten format. In the following code snippet, [Human Activity Recognition dataset](https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip "Human Activity Recognition dataset") is downloaded and uncompressed in the workspace directory. 


```r
if (!require(dplyr)) {
    install.packages("dplyr")
}
if (!require(tidyr)) {
    install.packages("tidyr")
  }
  library(dplyr)
  library(tidyr)
  
  download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip","dataset.zip")
  unzip("dataset.zip", overwrite = TRUE)
  
  dataset_dir <- paste(getwd(),"/UCI HAR Dataset", sep = "")
```

We are interested in loading the files of training and test datasets that contain records of 561-feature vector, activity subject and the participant who performed each activity. The files names are described as follow:


1.  'features.txt': List of all features.
2.  'activity_labels.txt': Links the class labels with their activity name.
3.  'train/X_train.txt': Training set.
4.  'train/y_train.txt': Training labels.
5.  'test/X_test.txt': Test set.
6.  'test/y_test.txt': Test labels.





## Section2: Combing and merging datasets

In this section, training and test datasets are merged. We do that by first by giving meaningful names to the columns of all loaded datasets. Second, *TRAIN* dataset is produced by combining the columns of x_train, y_train and subject_train datasets. The result dataset consist of 561 features columns + activity_label column + subject_code column. The same work is applied on test datasets to produce *TEST* dataset.
Finally, *TRAIN* and *TEST* datasets are merged together to obtain *DAT* dataset.


### Labelling columns headers with appropriate names for all loaded datasets

*activity_labels* dataset contains two variables. The first column which stores the code of each activity and is given a header activity_code. The second column is named as activity_label because it holds the descriptive name of each activity. 

*subject* datasets have one column which represents identifier codes of each participants who carried out the activity. This column is named as *subject_code*.

*y_\** datasets have one column which represents the code of the activity. This column is named as *activity_code*.

*x_\** datasets have 561 columns which are the variables of feature vector. These columns are named using the list of 561 features stored in the second column of *features* dataset.


```r
colnames(activity_labels) <-  c("activity_code", "activity_label") 

colnames(subject_train) <- c("subject_code")
colnames(subject_test) <- c("subject_code")

colnames(y_train) <- c("activity_code") 
colnames(y_test) <- c("activity_code") 

colnames(x_train) <- features[,2]
colnames(x_test) <- features[,2]
```


### Combining and merging

To ease performing aggregation operations on the datasets, we bind the columns of training datasets (x_train, y_train and subject_train) to produce one dataset called *TRAIN*. For testing dataset, the same binding is performed to generate *TEST* dataset. *TRAIN* and *TEST* datasets have the same structure of 561 columns features + activity_code + subject_code. 

*TRAIN* and *TEST* datasets are merged to obtain *DAT* dataset.

```r
TRAIN <- bind_cols(x_train, y_train,subject_train)
TEST <- bind_cols(x_test, data.frame(y_test),subject_test)
DAT <- rbind(TRAIN,TEST)
```


## Section 3: Extracting measurements on the mean and standard deviation for each measurement
According to features_info.txt file, there are 561 variables estimated from 33 features. We are only extracting the variables that represent the mean and standard deviation for each feature. As we have 33 features, our new dataset should have 66 variables plus two columns of activity_code and subject_code.

We use *grep()* function to extract that columns that their names contain either mean or std. The result is stored in a new data frame called *DAT2*.


```r
DAT2<- DAT[,grep("mean\\(|std\\(",colnames(DAT))]
DAT2 <- cbind(DAT2,DAT[,c("activity_code","subject_code")])
dim(DAT2)
```

```
## [1] 10299    68
```



## Section 4: Using descriptive activity names

We use *factor()* function to convert the variable activity_code to factor and labelling its levels with the values of activity_label.


```r
table(DAT2$activity_code)
```

```
## 
##    1    2    3    4    5    6 
## 1722 1544 1406 1777 1906 1944
```

```r
DAT2$activity_code <- factor(DAT2$activity ,labels =  activity_labels$activity_label)
#colnames(DAT2$activity_code) == "activity_label"

table(DAT2$activity_code)
```

```
## 
##            WALKING   WALKING_UPSTAIRS WALKING_DOWNSTAIRS 
##               1722               1544               1406 
##            SITTING           STANDING             LAYING 
##               1777               1906               1944
```

## Section 5: labelling the dataset with descriptive variable names

Most of labelling work has been already performed in section 1 after. Here we perform a little of cleaning on variables names. By using *gsub()* function, we remove the parenthesis and capitalise the first letter of mean and std. We also replace - with _.




```r
colnames(DAT2) <- gsub("mean\\(\\)","Mean",colnames(DAT2)) 
colnames(DAT2) <- gsub("std\\(\\)","Std",colnames(DAT2)) 
colnames(DAT2) <- gsub("-","_",colnames(DAT2)) 
colnames(DAT2)[colnames(DAT2) == "activity_code"] <- "activity_label"
```


## Section6: Averaging the variables by activity and subject


```r
summary_DAT2 <-  DAT2 %>% group_by_(.dots = c("activity_label","subject_code")) %>%
summarise_each_(funs(mean), colnames(DAT2[,-c(67,68)]))


head(data.frame(summary_DAT2)[1:4,1:6])
```

```
##   activity_label subject_code tBodyAcc_Mean_X tBodyAcc_Mean_Y
## 1        WALKING            1       0.2773308     -0.01738382
## 2        WALKING            2       0.2764266     -0.01859492
## 3        WALKING            3       0.2755675     -0.01717678
## 4        WALKING            4       0.2785820     -0.01483995
##   tBodyAcc_Mean_Z tBodyAcc_Std_X
## 1      -0.1111481     -0.2837403
## 2      -0.1055004     -0.4236428
## 3      -0.1126749     -0.3603567
## 4      -0.1114031     -0.4408300
```

## Section7: Tidying summary dataset
Based on Hadley Wickham's definition of tidy data where each variable is a column, each observation is a row, and each type of observational unit is a table. It can be seen that the first precept of the framework is violated in *summary_DAT2*. The variable "feature" is stored in more than one column. The values of this variable are the column headers.

In fact, *summary_DAT2* in tidy view should have four variables, *activity_label*, *subject_code*, *feature* and *average*. To tidy it, we need to melt, or stack it. We do that by parameterizing columns *activity_label*, *subject_code* that are already variables. The other columns are converted into two variables. The first variable called *feature* that contains repeated column headings. The second variable called *average* that contains concatenated values from previously separated columns. This data restructuring is implemented in the following code chunk using *gather()* function from *tidyr* package that produces  molten dataset.



```r
molten <- summary_DAT2 %>% gather(feature, average, - which(colnames(summary_DAT2) %in% c("activity_label","subject_code")))
head(x = molten,n = 4)
```

```
## Source: local data frame [4 x 4]
## Groups: activity_label
## 
##   activity_label subject_code         feature   average
## 1        WALKING            1 tBodyAcc_Mean_X 0.2773308
## 2        WALKING            2 tBodyAcc_Mean_X 0.2764266
## 3        WALKING            3 tBodyAcc_Mean_X 0.2755675
## 4        WALKING            4 tBodyAcc_Mean_X 0.2785820
```

```r
tail(x = molten,n = 4)
```

```
## Source: local data frame [4 x 4]
## Groups: activity_label
## 
##   activity_label subject_code                  feature    average
## 1         LAYING           27 fBodyBodyGyroJerkMag_Std -0.9935523
## 2         LAYING           28 fBodyBodyGyroJerkMag_Std -0.9693198
## 3         LAYING           29 fBodyBodyGyroJerkMag_Std -0.9975852
## 4         LAYING           30 fBodyBodyGyroJerkMag_Std -0.9754815
```

```r
write.csv(file = "molten.csv", x = molten)
```

In *molten* dataset, each row of *average* variable is the average of the values of an esteimated varaible for activity for subject.

