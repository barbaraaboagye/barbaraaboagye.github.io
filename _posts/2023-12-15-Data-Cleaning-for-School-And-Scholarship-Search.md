---
layout:     post
title:     HOW TO BUILD A SCHOOL AND SCHOLARSHIP SEARCH APP (3SA) USING PYTHON AND STREAMLIT (PART 1) : DATA CLEANING AND PRE-PROCESSING
subtitle: 
date:       2023-12-17
author:     "Barbara Aboagye"
header-img: 
---

# HOW TO BUILD A SCHOOL AND SCHOLARSHIP SEARCH APP (3SA) USING PYTHON AND STREAMLIT (PART 1): DATA CLEANING AND PREPROCESSING

How many of you have been shocked by University tuition cost? How many hours have you spent searching for suitable schools and scholarships that match your program of interest and still not found anything? 

To address these two major issues - the high cost of education and the difficulty of finding program-specific schools and scholarships, - I've built the School and Scholarship Search App (3SA) using Python and Streamlit. 

This app allows students to find schools that are tailored to their programs with accompanying scholarships, explore country-specific scholarships based on their program of interest, and find scholarships or schools based on their academic level of interest.

In this article, we'll explore the process of building the School and Scholarship Search App (3SA) in two parts: Part 1 focuses on cleaning and pre-processing the data, while Part 2 focuses on creating a user-friendly interface. Specifically, we'll cover cleaning, handling missing data, and restructuring the dataset to build 3SA.

## Data Cleaning and Preprocessing Process to build 3SA 

Let's break down the steps involved :

### 1. Data Collection 

The first step is collecting the dataset. The data used in this project was manually collected from university websites and social media platforms (Twitter and LinkedIn). You can find and download the raw dataset [here](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv). 

### 2. Importing libraries and loading dataset

Let's start by importing the necessary libraries and the [dataset](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv) :

```python
#Importing necessary python libraries

import numpy as np
import pandas as pd


# Load dataset
df = pd.read_csv('scholarshipdatabase.csv')

# Display the first few rows
df.head()
```

![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/a22929b7fca24b580f62be82c1afd8b539b3fb69/_posts/images/scholarship%20snapshot.png)

The libraries used are : 
- `numpy` for numerical operations.
- `pandas` for data manipulation and DataFrame operations.

The dataset has various headings with several missing data as shown above. The next step will be to explore the dataset further.

### 3. Data Exploration
Now, let's use the `info()` method to get information about the data types and the presence of any missing values in our dataset.

``` python
# Get information about the dataset
print(df.info())
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/dbabfe3bc4b66751f6acca4dbc48dfcce439eee6/_posts/images/info.png)

There are 704 entries with several null entries. The shape of the original dataset is (704,11)

### 4. Features selection

Out of the 11 columns/features present in the dataset, we are only interested in 4. We will therefore create a new DataFrame with only the relevant features. 

``` Python
# New dataframe with selected features
features = ['Name', 'Area of specialisation', 'Country', 'Level needed']
data = df[features]
data.head()
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/ca7a71275b7e6c24faf594320cf6ca13c87fb5b4/_posts/images/filtered%20dataset.png)

The new DataFrame `data` has only 4 features: Name, Area of specialisation, Country and Level needed. Let's check the shape 

``` Python
data.shape
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/552426fca6c3657fd9bf0c3ea08a2e467ea2b692/_posts/images/uncleaned%20dataset%20shape.png)

### 5. Splitting Columns into Separate Rows

The selected features: `Area of specialisation`, `Country` and `Level needed` contains multiple entries separated by commas. Split these entries into separate rows to make the data more accessible. 

``` Python
# Split certain columns into separate rows
data = data.assign(**{'Area of specialisation': data['Area of specialisation'].str.split(', ')})
data = data.explode('Area of specialisation')

data = data.assign(**{'Country': data['Country'].str.split(', ')})
data = data.explode('Country')

data = data.assign(**{'Level needed': data['Level needed'].str.split(", ")})
data = data.explode('Level needed')

# Reset the index for unique labels
data.reset_index(drop=True, inplace=True)

data.head()
```

![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/22edbc8a4187cf47be5361a44e2c59cac57b4712/_posts/images/cleaned%20dataset.png)

There are no more multiple entries for the columns. Let's check the shape. 

``` Python
data.shape
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/66397cb5881e6f8f9c6ee2417c668b88b7411b85/_posts/images/cleaned%20dataset%20shape.png)

The splitting has resulted in more rows in our DataFrame, changing the shape from (704,4) to (3829,4).

### 5. Handling Missing Data

Now that our data is organized, we will now identify and address the missing data.

``` Python
# Further identify missing data
data.isnull().sum()
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/de0698a2051bc062f4e6941823c1f0924282845d/_posts/images/shape%20filtered%20dataset.png)

``` Python
# Drop rows with missing 'Name' values
data.dropna(subset=['Name'], inplace=True)

# Fill remaining missing values with empty strings
data.fillna('', inplace=True)
data.isnull().sum()
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/d98f46baa50cd15daa61f70925e03e05137677ef/_posts/images/shape%20filtered%20dataset%20after%20filling.png)

### 6. Data Saving

Our dataset is now clean and ready to be used to build 3SA. It is saved in a new CSV file as "scholarship_df.csv"

``` Python
# Save the cleaned dataset
data.to_csv('scholarship_df.csv', index=False)
```
Now, your dataset is clean and ready to be used for building the School and Scholarship Search App (3SA). You can access the complete Python code [here](https://github.com/barbaraaboagye/My-MachineLearning-Journey/blob/31e326d6e5a6ac53e94cd4e9fc3145590a404c52/Projects/Scholarship%20recommender%20system/Data_processing.ipynb).
