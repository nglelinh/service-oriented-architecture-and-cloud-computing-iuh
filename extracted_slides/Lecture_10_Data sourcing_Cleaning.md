# Lecture_10_Data sourcing_Cleaning


## Slide 1

### Data Sourcing / Cleaning


## Slide 2

### Data Licensing


Can you use any data?
Check data sources for restrictions (commercial uses, foreign uses, etc)
MIT License
Very permissive, as long as you keep the license and copyright
GPLv2/v3
If you use it, you must distribute the source of anything built with it
Must include copyright, license, link to the original, and details of your changes
“Spreads virally”, anything that uses it must then become GPL


## Slide 3

### Data Sources


Data is everywhere! We can track things at scales that we never have before
Refined Datasets
These are typically academic or governmental datasets that are released to the public
For the most part all the cases of missing values and parsing is done for you (the data is in a table-like format)
Raw Data
This can be any data source:
Social media, sensor data, scientific data
You will need to clean the data in order to get it to a usable state
i.e. Missing values? Formatting? Non-normalized data?


## Slide 4

### Premade Dataset


There are a lot of good data sources already
If you can, use them instead of getting your own data
It has been cleaned; typically easier to download
There are other papers you can look to for examples of how they manipulated it
You may want to combine datasets or fill in null values


https://github.com/cazala/mnist


## Slide 5

### Raw Data


How would you process your newsfeed?


http://static.adweek.com/adweek.com-prod/wp-content/uploads/sites/2/2016/02/NewsFeedTeaser640.jpg


## Slide 6

### Raw Data - Web Scraping


Ideally you start with a headless browser and download a subset of the javascript objects/html from the page
Once you have the objects, save them in some format that makes sense. This can be a CSV, a Table, what have you.
You may need to do some HTML parsing in this case. Get yourself an HTML parser and write all the relevant files


## Slide 7

### 20 Second Example


from bs4 import BeautifulSoup soup = BeautifulSoup("""
<html><body><ul>
<li class="shoe-item">Air Jordans</li>
<li class="shoe-item">Light up Sketchers</li>
</ul><body></html> """, 'html.parser')
for item in soup.find_all(attrs='shoe-item'):
print(item.text)


## Slide 8

### Scraping Problems


You may get rate limited
You may get blocked
You may be violating the law
Whenever the HTML changes, your code immediately breaks


http://4.bp.blogspot.com/--POGMPcheZQ/U3AFerYzrKI/AAAAAAAABDo/ uNPZGsCpGAI/s1600/web+scraping+jime.jpg


## Slide 9

### Data Cleaning


## Slide 10

### Why do we need to clean the data?


Data could have:
Missing Values
Duplicate Values
Invalid Values
Useless Values
Etc
We need to make sure that the data that we give to the machine learning algorithm is as close to representative as possible


## Slide 11

### Why do we need to clean the data?


We usually want the data in some normalized format


https://www.red-gate.com/simple-talk/wp-content/uploads/imported/1519-image003.png


## Slide 12

### Missing Values


What can we do?
We can drop data
Careful: this may skew our dataset, especially if we have a lot of missing values.
We can make an educated guess of the values
Hard to do for categorical data
For numerical data:
Simple replacement with the mean
Sample a probability distribution to fill in the values (randomly)
Some algorithms don’t need all the values filled in
In that case, we leave as is because we want to put as little of our bias into the data


## Slide 13

### Missing Data Example


## Slide 14

### Duplicated Data


Simple answer:
Deduplicate the data
Can be challenging with very large datasets

Pitfalls:
Duplicate data can have meaning
How do you determine duplicate data?
Exact match? Close match?


## Slide 15

### Duplicates Example


## Slide 16

### Invalid and Useless Values


Problem domain species what is valid / useful
Context matters significantly:
i.e., age > 0
For “useless” values
Keep them around, they may be useful later


## Slide 17

### Data processing pipelines


## Slide 18

### Final Destination


Your data needs to be accessible to your processing framework
i.e. It needs to be placed in HDFS, S3, MySQL, etc.
For Hadoop / Spark:
HDFS has the “copyFromLocal” tool
For SQL Databases:
Many tools available for loading data
Write your own queries/scripts to load data from some other source
Data can be very large (> 1 PB):
Services like AWS Data Transfer (Snowball) exist to bulk transfer large amounts of data
