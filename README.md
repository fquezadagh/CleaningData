Course Project for Getting and Cleaning Data - Coursera
```
##Step 0: Load initial data

##Now we can start with the data
##As David showed us at https://class.coursera.org/getdata-011/forum/thread?thread_id=181#comment-510, we have the
## "y_train.txt","y_test.txt": Values of activity
## "subject_train.txt","subject_test.txt": values of subject
## "X_train.txt","X_test.txt": Values of features 
## "features.txt": Names of features
## "activity_labels.txt": Levels of activities

##Test data
ActivityTest<-read.table("UCI HAR Dataset/test/y_test.txt",header=FALSE)
SubjectTest<-read.table("UCI HAR Dataset/test/subject_test.txt",header=FALSE)
FeaturesTest<-read.table("UCI HAR Dataset/test/X_test.txt",header=FALSE)

##Train data
ActivityTrain<-read.table("UCI HAR Dataset/train/y_train.txt",header=FALSE)
SubjectTrain<-read.table("UCI HAR Dataset/train/subject_train.txt",header=FALSE)
FeaturesTrain<-read.table("UCI HAR Dataset/train/X_train.txt",header=FALSE)


##Step 1: Merges the training and the test sets to create one data set (by rows = rbind)

##Merging data
Activity<-rbind(ActivityTrain,ActivityTest)
Subject<-rbind(SubjectTrain,SubjectTest)
Features<-rbind(FeaturesTrain,FeaturesTest)

##Get features names
FeaturesNames<-read.table("UCI HAR Dataset/features.txt",header=FALSE)

##Set names to variables
names(Activity)<-c("Activity")
names(Subject)<-c("Subject")
names(Features)<-FeaturesNames$V2

##Merge all data frames to get one consolidate data frame

##I use a temporal data frame to merge Subject and Activity by column (cbind)
temporal<-cbind(Subject,Activity)

##Now I merge with Features data frame to get the Data data frame
Data<-cbind(Features,temporal)


##Step 2: Extracts only the measurements on the mean and standard deviation for each measurement


##To get this we have to create a factor by subsetting from FeaturesNames with "mean()" or "std()" on its name using
##grep()
WantedNames<-FeaturesNames$V2[grep("mean\\(\\)|std\\(\\)",FeaturesNames$V2)]

##Now we create a character vector adding to the WantedNames the columns "Subject" and "Activity"
WantedColumns<-c(as.character(WantedNames),"Subject","Activity")

##Finally, subset Data by WantedColumns. I opted to keep Data on memory, but it could be removed if you don't have
##enough memory
tidyData<-subset(Data,select=WantedColumns)


##Step 3: Uses descriptive activity names to name the activities in the data set


##Getting activities labels from activity_labels.txt
ActivityLabels<-read.table("UCI HAR Dataset/activity_labels.txt",header=FALSE)

##Convert the Activity variable to factor
tidyData$Activity<-as.factor(tidyData$Activity)

##Assigning levels
levels(tidyData$Activity)<-ActivityLabels$V2


##Step 4: Appropriately labels the data set with descriptive variable names


##If you check the names of the variables on the tidyData you'll see a lot of abbreviatures like t for time,
##f for frequency and so on, so we need to change that names using the gsub() function based on the next criteria (refer to features_info.txt):
##prefix t is replaced by time
##prefix f is replaced by frequency
##Acc is replaced by Accelerometer
##Gyro is replaced by Gyroscope
##Mag is replaced by Magnitude
##BodyBody is replaced by Body (duplicated word.  Why? I don't know)
names(tidyData)<-gsub("^t", "time", names(tidyData))
names(tidyData)<-gsub("^f", "frequency", names(tidyData))
names(tidyData)<-gsub("Acc", "Accelerometer", names(tidyData))
names(tidyData)<-gsub("Gyro", "Gyroscope", names(tidyData))
names(tidyData)<-gsub("Mag", "Magnitude", names(tidyData))
names(tidyData)<-gsub("BodyBody", "Body", names(tidyData))


##Step 5: From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject


##We can use aggregate() to summarize the information and get the average of each variable
tidyData2<-aggregate(. ~Subject + Activity, tidyData, mean)

##If we want, we can order the tidyData2 data frame by subject and by activity
tidyData2<-tidyData2[order(tidyData2$Subject,tidyData2$Activity),]

##Finally, export the resulting data frame to a text file
write.table(tidyData2, file="result.txt", row.names=FALSE)
```
