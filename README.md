# tidydata

### load appropriate packages

library(dplyr)

library(tidyverse)

### import data from directory for all needed files, rename subject ID data in subtrain/subtest

xtest <- read.table('~/Downloads/UCI HAR Dataset/test/X_test.txt')

ytest <- read.table('~/Downloads/UCI HAR Dataset/test/y_test.txt') 

xtrain <- read.table('~/Downloads/UCI HAR Dataset/train/X_train.txt')

ytrain <- read.table('~/Downloads/UCI HAR Dataset/train/y_train.txt')

features <- read.table('~/Downloads/UCI HAR Dataset/features.txt')

activity <- read.table('~/Downloads/UCI HAR Dataset/activity_labels.txt')

subtrain <- read.table('~/Downloads/UCI HAR Dataset/train/subject_train.txt')

subtrain <- subtrain %>%
                rename(subjectID=V1) 

subtest <- read.table('~/Downloads/UCI HAR Dataset/test/subject_test.txt')

subtest <- subtest %>%
                rename(subjectID=V1)

### Rename columns in xtrain and xtest

features <- features[,2]

featrasp <- t(features)

colnames(xtrain) <- featrasp

colnames(xtest) <- featrasp

colnames(activity) <- c('id', 'actions')

### Bind rows together and then create data frame 

combineX <- rbind(xtrain, xtest)

combineY <- rbind(ytrain, ytest)

combineSubj <- rbind(subtrain, subtest)

XYdf <- cbind(combineX, combineY, combineSubj)

df <- merge(XYdf, activity, by.x = 'V1', by.y = 'id')

colNames <- colnames(df)

### create second df with means and standard deviations, tidy this up

df2 <- df %>%
        select(actions, subjectID, grep("\\bmean\\b|\\bstd\\b", colNames))

df2$actions <- as.factor(df2$actions)

colnames(df2)<-gsub("^t", "time", colnames(df2))

colnames(df2)<-gsub("^f", "frequency", colnames(df2))

colnames(df2)<-gsub("Acc", "Accelerometer", colnames(df2))

colnames(df2)<-gsub("Gyro", "Gyroscope", colnames(df2))

colnames(df2)<-gsub("Mag", "Magnitude", colnames(df2))

colnames(df2)<-gsub("BodyBody", "Body", colnames(df2))

### create 3rd df with aggregated data and write to .txt document for upload

df3 <- aggregate(. ~subjectID + actions, df2, mean)

write.table(df3, file = "tidydata.txt", row.names = FALSE)

### create codebook

library(dataMaid)
makeCodebook(df3)
