# Code Book
Riad Darawish  
2 July 2017  



## Code book
This code book explains how the summary dataset *molten* is generated from original version of [Human Activity Recognition dataset](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones). *molten* dataset is obtained after being exposed to the following operations:

1. Merging the training and testing datasets files of the orginal dataset.
2. Combining the variables of the subject code and activity code with merged dataset.
3. Replacing the activity codes with descriptive activity names.
4. Extracting the variables that represents the mean and standard deviation for each measurement
5. Modifying the variables names by replacing - with _ and capitalising the first letter of mean and std.
6. Averaging the variables by activity and subject to produce summary dataset.
7. Tidying summary dataset by melting it.


The *molten* dataset has the following four variables:


activity_label: Six activity labels. WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING and LAYING

subject_code: An identifier of the subject who carried out the experiment. Its range is from 1 to 30.

feature: set of estimated variables that represents the mean and standard deviation for each measurement

average: The average of each estimated variable for each activity and subject.
