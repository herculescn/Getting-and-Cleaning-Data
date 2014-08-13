Code book for run_analysis.R
=========================

The code in run_analysis.R does the following. 
* Get feature name of the measurements on the mean and standard deviation for each measurement. 
* Extracts only the measurements on the mean and standard deviation for each measurement and merges the training and the test sets to create one data set.
* Uses descriptive activity names to name the activities in the data set
* Appropriately labels the data set with descriptive variable names. 
* Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 

Here are the data for the project: 
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

### Get feature name of the measurements on the mean and standard deviation for each measurement. 
```
featureInfo <- read.table("UCI HAR Dataset/features.txt",stringsAsFactors =FALSE)
count<- 1
colNames <- ""
colNums <- 1
# get all feature name include "mean" or "std"
for (i in 1: length(featureInfo[,1])){
  colName <- featureInfo[,2][i]
  nameLen <- nchar(colName)
  subName <- substring(colName, nameLen-7,nameLen -4)
  if(subName == "mean" | subName == "-std"){
    # put name of feature in colNames
    colNames[count] <- colName
    # put number of feature in colNums
    colNums[count] <- i 
    count <- count + 1    
  }  
}
colNames[count] <- "ActivityName"
```

### Extracts only the measurements on the mean and standard deviation for each measurement and merges the training and the test sets to create one data set.
```
testData <- read.table("UCI HAR Dataset/test/X_test.txt")
trainData <- read.table("UCI HAR Dataset/train/X_train.txt")
mergedData <- rbind(testData[,colNums],trainData[,colNums])
```

### Uses descriptive activity names to name the activities in the data set
```
# read acciveity names
activityName <- read.table("UCI HAR Dataset/activity_labels.txt", stringsAsFactors =FALSE)
# get active label for each item of merged data
testAndTraningLabels <- 
  rbind(read.table("UCI HAR Dataset/test/y_test.txt"),
        read.table("UCI HAR Dataset/train/y_train.txt"))

# get activity name for each item
testAndTraningActivityName<- activityName[,2][testAndTraningLabels[,1]]

# name the activities in the data set
dataSet <- cbind(mergedData,testAndTraningActivityName)
```
### Appropriately labels the data set with descriptive variable names. 
```
names(dataSet)<- colNames
```

### Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 
```
# get test and train subject labels
subjectLabels <- 
  rbind(read.table("UCI HAR Dataset/test/subject_test.txt"),
        read.table("UCI HAR Dataset/train/subject_train.txt"))
# paste activity name and subject for average
activityAndSubjec <- paste( testAndTraningActivityName, subjectLabels[,1])

# get the average of each variable for each activity and each subject. 
avgData <- lapply(split(dataSet, factor(activityAndSubjec)),
       function(x) colMeans(x[, colNames[1:count - 1]]))
# write the result to file
write.table(avgData, "avgData.txt", row.name=FALSE)
```
