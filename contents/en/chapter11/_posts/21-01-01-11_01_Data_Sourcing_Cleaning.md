---
layout: post
title: 11-01 Data Sourcing, Cleaning, and Preparation
chapter: '11'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter11
lesson_type: required
---

Data preparation is a critical step in any data analytics or machine learning project. This lecture covers the essential techniques for sourcing data from various sources and cleaning it for analysis.

## Data Sourcing

### Data Licensing

> [!WARNING]
> **Can you use any data?**
> 
> Always check data sources for restrictions before use!

Common licenses:

#### MIT License
- **Very permissive**
- Can use commercially
- Must keep license and copyright notice

#### GPL (v2/v3)
- **Copyleft license**
- Must distribute source code of anything built with it
- Must include copyright, license, link to original, and change details
- "Spreads virally" - anything using it must become GPL

#### Other Considerations
- **Commercial use restrictions**
- **Foreign use limitations**
- **Attribution requirements**
- **Data privacy regulations** (GDPR, CCPA)

### Types of Data Sources

#### 1. Refined Datasets

**Characteristics**:
- Academic or governmental datasets
- Released to the public
- Pre-cleaned and formatted
- Table-like format (CSV, databases)

**Advantages**:
- ✓ Missing values handled
- ✓ Parsing done
- ✓ Easier to download
- ✓ Examples from other papers

**Examples**:
- **MNIST**: Handwritten digits dataset
- **ImageNet**: Image classification dataset
- **UCI Machine Learning Repository**
- **Kaggle Datasets**
- **Government open data portals**

#### 2. Raw Data

**Characteristics**:
- Unprocessed data from original sources
- May have missing values, formatting issues
- Requires significant cleaning

**Sources**:
- Social media (Twitter, Facebook)
- Sensor data (IoT devices)
- Scientific instruments
- Web scraping
- APIs

**Challenges**:
- ⚠ Missing values
- ⚠ Inconsistent formatting
- ⚠ Non-normalized data
- ⚠ Requires extensive cleaning

## Web Scraping

### Process

1. **Use headless browser** to download JavaScript objects/HTML
2. **Parse HTML** to extract relevant data
3. **Save in structured format** (CSV, JSON, database)

### Example with BeautifulSoup

```python
from bs4 import BeautifulSoup

html_content = """
<html><body><ul>
  <li class="shoe-item">Air Jordans</li>
  <li class="shoe-item">Light up Sketchers</li>
</ul></body></html>
"""

soup = BeautifulSoup(html_content, 'html.parser')

for item in soup.find_all(attrs={'class': 'shoe-item'}):
    print(item.text)

# Output:
# Air Jordans
# Light up Sketchers
```

### Web Scraping Challenges

⚠ **Rate limiting**: Servers may block excessive requests  
⚠ **IP blocking**: May get banned for aggressive scraping  
⚠ **Legal issues**: May violate terms of service  
⚠ **Fragile code**: Breaks when HTML structure changes  
⚠ **Dynamic content**: JavaScript-rendered content requires special handling  

> **Best Practice**: Use official APIs when available instead of scraping

## Data Cleaning

### Why Clean Data?

Data quality issues include:

- **Missing values**: Incomplete records
- **Duplicate values**: Repeated entries
- **Invalid values**: Data outside expected range
- **Useless values**: Irrelevant information
- **Inconsistent formats**: Different representations of same data

> **Goal**: Ensure data is as representative and accurate as possible for machine learning algorithms

### Data Normalization

Data should be in a normalized format:

```
Before:                    After:
Name: "John Smith"         first_name: "John"
Age: "25 years"           last_name: "Smith"
City: "NYC"               age: 25
                          city: "New York City"
```

## Handling Missing Values

### Strategies

#### 1. Drop Data

```python
import pandas as pd

# Drop rows with any missing values
df_clean = df.dropna()

# Drop rows where specific column is missing
df_clean = df.dropna(subset=['age'])

# Drop columns with too many missing values
df_clean = df.dropna(axis=1, thresh=len(df)*0.5)
```

**Considerations**:
- ⚠ May skew dataset
- ⚠ Loss of information
- ✓ Simple and fast
- ✓ Works when few missing values

#### 2. Imputation - Fill with Mean/Median

```python
# Fill with mean
df['age'].fillna(df['age'].mean(), inplace=True)

# Fill with median (better for skewed data)
df['salary'].fillna(df['salary'].median(), inplace=True)

# Fill with mode (for categorical data)
df['category'].fillna(df['category'].mode()[0], inplace=True)
```

**Considerations**:
- ✓ Preserves dataset size
- ⚠ Introduces bias
- ⚠ Reduces variance

#### 3. Forward/Backward Fill

```python
# Forward fill (use previous value)
df['temperature'].fillna(method='ffill', inplace=True)

# Backward fill (use next value)
df['temperature'].fillna(method='bfill', inplace=True)
```

**Use case**: Time series data

#### 4. Interpolation

```python
# Linear interpolation
df['value'].interpolate(method='linear', inplace=True)

# Polynomial interpolation
df['value'].interpolate(method='polynomial', order=2, inplace=True)
```

#### 5. Probabilistic Sampling

```python
import numpy as np

# Sample from distribution
mean = df['age'].mean()
std = df['age'].std()
missing_count = df['age'].isnull().sum()

df.loc[df['age'].isnull(), 'age'] = np.random.normal(mean, std, missing_count)
```

#### 6. Leave As Is

Some algorithms handle missing values:
- Decision trees
- Random forests
- XGBoost

**Advantage**: Minimal bias introduction

## Handling Duplicates

### Identifying Duplicates

```python
# Find duplicate rows
duplicates = df.duplicated()

# Find duplicates based on specific columns
duplicates = df.duplicated(subset=['name', 'email'])

# View duplicate rows
df[df.duplicated(keep=False)]
```

### Removing Duplicates

```python
# Remove all duplicates
df_clean = df.drop_duplicates()

# Keep first occurrence
df_clean = df.drop_duplicates(keep='first')

# Keep last occurrence
df_clean = df.drop_duplicates(keep='last')

# Remove based on specific columns
df_clean = df.drop_duplicates(subset=['user_id'])
```

### Considerations

> [!IMPORTANT]
> **Duplicates can have meaning!**
> 
> - Multiple purchases by same customer
> - Repeated sensor readings
> - Time-series data

**Questions to ask**:
- What defines a duplicate? (Exact match? Close match?)
- Should we keep any duplicates?
- Is there temporal ordering to consider?

## Handling Invalid and Useless Values

### Invalid Values

**Problem domain specifies what is valid**:

```python
# Example: Age must be positive
df = df[df['age'] > 0]

# Example: Percentage must be 0-100
df = df[(df['percentage'] >= 0) & (df['percentage'] <= 100)]

# Example: Date must be in past
from datetime import datetime
df = df[df['date'] <= datetime.now()]
```

### Outliers

```python
# Z-score method
from scipy import stats
z_scores = np.abs(stats.zscore(df['value']))
df_clean = df[z_scores < 3]

# IQR method
Q1 = df['value'].quantile(0.25)
Q3 = df['value'].quantile(0.75)
IQR = Q3 - Q1
df_clean = df[(df['value'] >= Q1 - 1.5*IQR) & 
              (df['value'] <= Q3 + 1.5*IQR)]
```

### Useless Values

> **Tip**: Keep them around - they may be useful later!

- Columns with single unique value
- Highly correlated features
- Irrelevant to problem domain

## Data Processing Pipelines

### Pipeline Stages

```
┌─────────────┐
│ Data Source │
└──────┬──────┘
       │
┌──────▼──────┐
│  Extract    │ (Get data from source)
└──────┬──────┘
       │
┌──────▼──────┐
│ Transform   │ (Clean, normalize, enrich)
└──────┬──────┘
       │
┌──────▼──────┐
│   Load      │ (Store in destination)
└──────┬──────┘
       │
┌──────▼──────┐
│  Validate   │ (Check quality)
└─────────────┘
```

### Example Pipeline with Pandas

```python
import pandas as pd

def data_pipeline(input_file, output_file):
    # 1. Extract
    df = pd.read_csv(input_file)
    
    # 2. Transform
    # Handle missing values
    df['age'].fillna(df['age'].median(), inplace=True)
    df['category'].fillna('Unknown', inplace=True)
    
    # Remove duplicates
    df = df.drop_duplicates(subset=['user_id'])
    
    # Handle invalid values
    df = df[df['age'] > 0]
    df = df[df['age'] < 120]
    
    # Normalize formats
    df['email'] = df['email'].str.lower().str.strip()
    df['phone'] = df['phone'].str.replace(r'\D', '', regex=True)
    
    # 3. Load
    df.to_csv(output_file, index=False)
    
    # 4. Validate
    print(f"Original rows: {len(pd.read_csv(input_file))}")
    print(f"Cleaned rows: {len(df)}")
    print(f"Missing values:\n{df.isnull().sum()}")
    
    return df

# Run pipeline
clean_data = data_pipeline('raw_data.csv', 'clean_data.csv')
```

## Loading Data into Processing Frameworks

### Hadoop / Spark

```bash
# Copy to HDFS
hdfs dfs -copyFromLocal local_file.csv /data/input/

# Or use Spark
spark-submit --master yarn load_data.py
```

### SQL Databases

```python
import pandas as pd
from sqlalchemy import create_engine

# Create connection
engine = create_engine('postgresql://user:pass@localhost/db')

# Load data
df.to_sql('table_name', engine, if_exists='replace', index=False)
```

### Cloud Storage

```python
# AWS S3
import boto3
s3 = boto3.client('s3')
s3.upload_file('local_file.csv', 'bucket-name', 'data/file.csv')

# Google Cloud Storage
from google.cloud import storage
client = storage.Client()
bucket = client.bucket('bucket-name')
blob = bucket.blob('data/file.csv')
blob.upload_from_filename('local_file.csv')
```

### Large Data Transfer

For very large datasets (> 1 PB):

- **AWS Snowball**: Physical device for bulk data transfer
- **Azure Data Box**: Microsoft's physical transfer solution
- **Google Transfer Appliance**: Google's hardware solution

## Best Practices

### 1. Document Everything

```python
# Keep track of transformations
cleaning_log = {
    'missing_values_filled': ['age', 'salary'],
    'duplicates_removed': 150,
    'outliers_removed': 23,
    'invalid_values_removed': 5
}
```

### 2. Validate at Each Step

```python
def validate_data(df, stage):
    print(f"\n=== Validation at {stage} ===")
    print(f"Shape: {df.shape}")
    print(f"Missing values:\n{df.isnull().sum()}")
    print(f"Duplicates: {df.duplicated().sum()}")
    print(f"Data types:\n{df.dtypes}")
```

### 3. Keep Original Data

```python
# Never modify original
df_original = pd.read_csv('data.csv')
df_working = df_original.copy()

# Work on copy
df_working = clean_data(df_working)
```

### 4. Automate Repetitive Tasks

```python
def clean_column(series, strategy='mean'):
    """Reusable cleaning function"""
    if strategy == 'mean':
        return series.fillna(series.mean())
    elif strategy == 'median':
        return series.fillna(series.median())
    elif strategy == 'mode':
        return series.fillna(series.mode()[0])
    return series
```

## Summary

Data sourcing and cleaning are critical steps in the data pipeline:

- **Sourcing**: Obtain data from refined datasets, APIs, or web scraping
- **Licensing**: Always check data usage rights
- **Missing values**: Drop, impute, or leave based on context
- **Duplicates**: Remove carefully, considering domain meaning
- **Invalid values**: Filter based on business rules
- **Pipelines**: Automate ETL processes
- **Validation**: Check quality at each step

**Remember**: Data scientists spend 60-80% of their time on data preparation - it's worth doing well!

## Further Reading

- [Pandas Documentation](https://pandas.pydata.org/docs/)
- [BeautifulSoup Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [Data Cleaning with Python](https://realpython.com/python-data-cleaning-numpy-pandas/)
