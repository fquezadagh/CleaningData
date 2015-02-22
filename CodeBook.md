#Course Project Code Book

##Original (raw) data

The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% (7352 observations) of the volunteers was selected for generating the training data and 30% (2947 observations) the test data

The features selected for this database come from the accelerometer and gyroscope 3-axial raw signals tAcc-XYZ and tGyro-XYZ. These time domain signals (prefix 't' to denote time) were captured at a constant rate of 50 Hz. Then they were filtered using a median filter and a 3rd order low pass Butterworth filter with a corner frequency of 20 Hz to remove noise. Similarly, the acceleration signal was then separated into body and gravity acceleration signals (tBodyAcc-XYZ and tGravityAcc-XYZ) using another low pass Butterworth filter with a corner frequency of 0.3 Hz. 

Subsequently, the body linear acceleration and angular velocity were derived in time to obtain Jerk signals (tBodyAccJerk-XYZ and tBodyGyroJerk-XYZ). Also the magnitude of these three-dimensional signals were calculated using the Euclidean norm (tBodyAccMag, tGravityAccMag, tBodyAccJerkMag, tBodyGyroMag, tBodyGyroJerkMag). 

Finally a Fast Fourier Transform (FFT) was applied to some of these signals producing fBodyAcc-XYZ, fBodyAccJerk-XYZ, fBodyGyro-XYZ, fBodyAccJerkMag, fBodyGyroMag, fBodyGyroJerkMag. (Note the 'f' to indicate frequency domain signals). 

These signals were used to estimate variables of the feature vector for each pattern:  
'-XYZ' is used to denote 3-axial signals in the X, Y and Z directions.

* tBodyAcc-XYZ
* tGravityAcc-XYZ
* tBodyAccJerk-XYZ
* tBodyGyro-XYZ
* tBodyGyroJerk-XYZ
* tBodyAccMag
* tGravityAccMag
* tBodyAccJerkMag
* tBodyGyroMag
* tBodyGyroJerkMag
* fBodyAcc-XYZ
* fBodyAccJerk-XYZ
* fBodyGyro-XYZ
* fBodyAccMag
* fBodyAccJerkMag
* fBodyGyroMag
* fBodyGyroJerkMag

The set of variables that were estimated from these signals are: 

* mean(): Mean value
* std(): Standard deviation
* mad(): Median absolute deviation 
* max(): Largest value in array
* min(): Smallest value in array
* sma(): Signal magnitude area
* energy(): Energy measure. Sum of the squares divided by the number of values. 
* iqr(): Interquartile range 
* entropy(): Signal entropy
* arCoeff(): Autorregresion coefficients with Burg order equal to 4
* correlation(): correlation coefficient between two signals
* maxInds(): index of the frequency component with largest magnitude
* meanFreq(): Weighted average of the frequency components to obtain a mean frequency
* skewness(): skewness of the frequency domain signal 
* kurtosis(): kurtosis of the frequency domain signal 
* bandsEnergy(): Energy of a frequency interval within the 64 bins of the FFT of each window.
* angle(): Angle between to vectors.

Additional vectors obtained by averaging the signals in a signal window sample. These are used on the angle() variable:

* gravityMean
* tBodyAccMean
* tBodyAccJerkMean
* tBodyGyroMean
* tBodyGyroJerkMean

## Data processing and transformations

This process starts with the assumption that the Samsung data is available in the working directory in an unzipped UCI HAR Dataset folder

###Step 1: Load initial data

As David showed us at https://class.coursera.org/getdata-011/forum/thread?thread_id=181#comment-510, we have the following data hierarchy: 
* "y_train.txt","y_test.txt": Values of activity
* "subject_train.txt","subject_test.txt": values of subject
* "X_train.txt","X_test.txt": Values of features 
* "features.txt": Names of features
* "activity_labels.txt": Levels of activities

So, first we load the test data
```r
ActivityTest<-read.table("UCI HAR Dataset/test/y_test.txt",header=FALSE)
SubjectTest<-read.table("UCI HAR Dataset/test/subject_test.txt",header=FALSE)
FeaturesTest<-read.table("UCI HAR Dataset/test/X_test.txt",header=FALSE)
```
Now, the train data
```r
ActivityTrain<-read.table("UCI HAR Dataset/train/y_train.txt",header=FALSE)
SubjectTrain<-read.table("UCI HAR Dataset/train/subject_train.txt",header=FALSE)
FeaturesTrain<-read.table("UCI HAR Dataset/train/X_train.txt",header=FALSE)
```

###Step 2: Merges the training and the test sets to create one data set (by rows = rbind)

Merging data from Activity, Subject and Features in a single data frame for each variable
```r
Activity<-rbind(ActivityTrain,ActivityTest)
Subject<-rbind(SubjectTrain,SubjectTest)
Features<-rbind(FeaturesTrain,FeaturesTest)
```

Get features names from the "features.txt" file
```r
FeaturesNames<-read.table("UCI HAR Dataset/features.txt",header=FALSE)
```

Set names to variables
```r
names(Activity)<-c("Activity")
names(Subject)<-c("Subject")
names(Features)<-FeaturesNames$V2
```

Merge all data frames to get one consolidate data frame

I use a temporal data frame to merge Subject and Activity by column (cbind)
```r
temporal<-cbind(Subject,Activity)
```

Now I merge with Features data frame to get the Data data frame
```r
Data<-cbind(Features,temporal)
```

###Step 3: Extracts only the measurements on the mean and standard deviation for each measurement


To get this we have to create a factor by subsetting from FeaturesNames with "mean()" or "std()" on its name using grep()
```r
WantedNames<-FeaturesNames$V2[grep("mean\\(\\)|std\\(\\)",FeaturesNames$V2)]
```

Now we create a character vector adding to the WantedNames the columns "Subject" and "Activity" 
```r
WantedColumns<-c(as.character(WantedNames),"Subject","Activity")
```

Finally, subset Data by WantedColumns. I opted to keep Data on memory, but it could be removed if you don't have enough memory
```r
tidyData<-subset(Data,select=WantedColumns)
```


###Step 4: Uses descriptive activity names to name the activities in the data set

First, we get the activities labels from activity_labels.txt
```r
ActivityLabels<-read.table("UCI HAR Dataset/activity_labels.txt",header=FALSE)
```

Convert the Activity variable to factor
```r
tidyData$Activity<-as.factor(tidyData$Activity)
```

Assigning levels
```r
levels(tidyData$Activity)<-ActivityLabels$V2
```

###Step 5: Appropriately labels the data set with descriptive variable names


If you check the names of the variables on the tidyData you'll see a lot of abbreviatures like t for time, f for frequency and so on, so we need to change that names using the gsub() function based on the next criteria (refer to features_info.txt):
* prefix t is replaced by time
* prefix f is replaced by frequency
* Acc is replaced by Accelerometer
* Gyro is replaced by Gyroscope
* Mag is replaced by Magnitude
* BodyBody is replaced by Body (duplicated word.  Why? I don't know)
```r
names(tidyData)<-gsub("^t", "time", names(tidyData))
names(tidyData)<-gsub("^f", "frequency", names(tidyData))
names(tidyData)<-gsub("Acc", "Accelerometer", names(tidyData))
names(tidyData)<-gsub("Gyro", "Gyroscope", names(tidyData))
names(tidyData)<-gsub("Mag", "Magnitude", names(tidyData))
names(tidyData)<-gsub("BodyBody", "Body", names(tidyData))
```

###Step 6: From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject


We can use aggregate() to summarize the information and get the average of each variable
```r
tidyData2<-aggregate(. ~Subject + Activity, tidyData, mean)
```

If we want, we can order the tidyData2 data frame by subject and by activity
```r
tidyData2<-tidyData2[order(tidyData2$Subject,tidyData2$Activity),]
```

Finally, export the resulting data frame to a text file
```r
write.table(tidyData2, file="result.txt", row.names=FALSE)
```
