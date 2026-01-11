# Lecture_4.3_Spark_MLIB


## Slide 1

### Spark MLlib


## Slide 2

### Machine Learning Basics


What comes first?


## Slide 3

### Machine Learning Basics


What comes first?
Data, sparse and labeled


## Slide 4

### Machine Learning Basics


What comes first?
Data, sparse and labeled
How is the data represented?


## Slide 5

### Machine Learning Basics


What comes first?
Data, sparse and labeled
How is the data represented?
Continuous or Discrete? Supervised or Unsupervised?


## Slide 6

### Machine Learning Techniques


We will be covering three broad types of  techniques:
Regression
Tries to predict an output given data  (continuous)
Classifiers
Takes data and try to assign it a label (discrete)
Clustering
Don’t know labels or numbers.
Groups similar data points into a group (or  ‘cluster’).


## Slide 7

### Regression


Fits a function to your data.
For example, linear regression finds a line of best fit


## Slide 8

### Classifiers


Takes data and assigns them a label based on what it is ‘closest’ to.
Supervised


## Slide 9

### Clustering


Unsupervised; used when there are no labels
The algorithm determines the clusters


## Slide 10

### How Do I Know If My Model Is Any Good?


Check your data and clean it up!
Good models only come from good data
Don’t Overfit!!
Metrics
Precision, accuracy, area under ROC, true positive rate, root mean  squared error, etc…


## Slide 11

### Performance Metrics


Confusion Matrix
Useful for Classification

RMSE - Root Mean Square Error
Useful for Regression


## Slide 12

### Overfitting


When your model is too good
Happens when your model ‘learns’ random  noise in your training data.


## Slide 13

### Improve Models with Data


Get More Data
Invent, Simulate, Resample…
Transform Data
Reshape the distribution, Rescale the data...
Feature Engineering
Create and add new features
Clean Data
Missing data handling, Reduce Noise…


## Slide 14

### Improving Models


Feature Selection
Selecting features to improve the prediction model
Use when there are a lot of features (noise) and not enough data points
Sometimes adding more feature can also improve the model as it  decrease bias.
To
Reduce Overfitting
Improve Accuracy
Reduce overall Training


## Slide 15

### Distributed Machine  Learning


## Slide 16

### The Options


Apache Singa


## Slide 17

### Machine Learning on Spark (MLlib)


MLlib allows for distributed machine learning on very large datasets.
Built on top of Spark so you can use it easily within Spark
Designed to be similar in use to NumPy
Can interoperate with NumPy and SciPy


## Slide 18

### Machine Learning on Spark (MLlib)


Can use RDDs or DataFrames
Unfortunately, they have slightly different feature sets…

RDD API:
pyspark.mllib.*
Original API, now in “Maintenance Mode”

DataFrame API:
pyspark.ml.*
Primary API for MLlib for Spark 2.0+
Support for ML “pipelines”
Less “glue” code necessary


## Slide 19

### When to use MLlib?


When your data is LARGE
To work with the Spark Ecosystem
Real-Time Machine Learning (with Spark Streaming)
