# coursera
In this repository, you will find the files generated for the assignment of week 4 of the coursera course "Getting and Cleaning Data". The parent directory is coursera and the data are stored in "./UCI HAR Dataset".
The goal of the assignment was to merge two dataset related to an experiment of "Human Activity Recognition using a Smartphone". The first dataset concerned the train group, and the second the test group. The final merged dataset should be clean and tidy and  concerns only mean and SD variables. The labels should be clear.
At the end, a new table "tidy_data.txt" was created and stored in the parent directory. This dataset calculated the mean for each variable.
The script is described below.


# read the file features (column names) ; limit to column number 2 to simplify the table
features <- read.table("./UCI HAR Dataset/features.txt")[,2]

# Extract only the names of the columns with mean or standard deviation in their name.
meanSD_features <- grepl("mean|std", features)

# 1° start with test
# Read the 2 test files: X_test and subject_test to assign the subjects.
X_test <- read.table("./UCI HAR Dataset/test/X_test.txt")
subject_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")

# Assign column names to X_test and to the only column in subject_test
names(X_test) = features
names(subject_test) = "subject"

# Keep only the the columns with mean or standard deviation in their name using meanSD_features earlier defined
X_test = X_test[,meanSD_features]

# Read the file activity labels ; limit to column 2 and read y_test (which reports the activities)
activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")[,2]
y_test <- read.table("./UCI HAR Dataset/test/y_test.txt")

#Combine y_test with activity_labels ; take only the column 2 and give a name to column 1
y_test[,2] = activity_labels[y_test[,1]]

# Give a name to the two columns of y_test
names(y_test) = c("Activity_ID", "Activity_Label")

# Bind data using cbind to the 3 objects: subject_test, y_test and X_test; use as.data.table to transform subject_test
test_data <- cbind(as.data.table(subject_test), y_test, X_test)

#2° train ; repeat the same process for train
# Read the 2 train files: X_train and subject_train to assign the subjects.
X_train <- read.table("./UCI HAR Dataset/train/X_train.txt")
subject_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")

# Assign column names to X_train and to the only column in subject_train
names(X_train) = features
names(subject_train) = "subject"

# Keep only the the columns with mean or standard deviation in their name using meanSD_features earlier defined
X_train = X_train[,meanSD_features]

# read y_test (which reports the activities); # the file activity labels has already been read earlier
y_train <- read.table("./UCI HAR Dataset/train/y_train.txt")

#Combine y_train with activity_labels ; take only the column 2 and give a name to column 1
y_train[,2] = activity_labels[y_train[,1]]

# Give a name to the two columns of y_train
names(y_train) = c("Activity_ID", "Activity_Label")

# Bind data using cbind to the 3 objects: subject_test, y_test and X_test; use as.data.table to transform subject_test
train_data <- cbind(as.data.table(subject_train), y_train, X_train)

# 3° Merge test and train data with rbind (add the rows)
mergedDT = rbind(test_data, train_data)

# Add the labels
id_labels = c("subject", "Activity_ID", "Activity_Label")
data_labels = setdiff(colnames(mergedDT), id_labels)
melt_data = melt(mergedDT, id = id_labels, measure.vars = data_labels)

# Apply mean function to melt_data using dcast function
tidy_data = dcast(melt_data, subject + Activity_Label ~ variable, mean)

# Write the new table after merging test and train
write.table(tidy_data, file = "./tidy_data.txt")
