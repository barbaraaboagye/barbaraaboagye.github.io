---
layout:     post
title:     HOW TO BUILD A SCHOOL AND SCHOLARSHIP SEARCH APP (3SA) USING PYTHON AND STREAMLIT (PART 1): DATA CLEANING AND PREPROCESSING
subtitle:   
date:       2023-12-17
author:     "Barbara Aboagye"
header-img: 
---

# HOW TO BUILD A SCHOOL AND SCHOLARSHIP SEARCH APP (3SA) USING PYTHON AND STREAMLIT (PART 1) : DATA CLEANING AND PREPROCESSING

When it comes to education, academics are often faced with two main challenges : the high cost of education and the overwhelming process of finding suitable schools and scholarships for their specific program. I’ve addressed these issues with the School and Scholarship Search App (3SA) built using Python and Streamlit. 

This article is divided into two parts: Part 1 covers data cleaning and pre-processing, while Part 2  will look into creating and designing an intuitive user interface.
In this article, we will learn the process of cleaning, handling missing data and restructuring the dataset  for building 3SA. 

## Data Preprocessing and cleaning to build 3SA : Process To Follow

Below is the process we can follow : 
1. Gathering, importing necessary libraries and loading the scholarship data
    - Import necessary libraried
    - Load the dataset

2. Data Exploration
   - Displays the first few rows of the dataset.
   - Retrieves information about the dataset.

3. Data Cleaning - Handling Missing Data:
   - Identifies missing data in the dataset.
   - Creates a new DataFrame with selected features.
   - Splits certain columns into separate rows.
   - Resets the index for unique labels.

4. Data Cleaning - Handling Missing Data (Continued):
   - Further identifies missing data in the cleaned dataset.
   - Drops rows with missing 'Name' values.
   - Fills remaining missing values with empty strings.

5. Data Saving:
   - Saves the cleaned dataset as "scholarship_df.csv."

So the process starts with collecting the dataset to be used. The data used in this project was manually collected and the raw dataset can be found and downloaded [here](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv). 

### Importing libraries and loading dataset
Let's start by importing the necessary libraries and the [dataset](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv) :

```python
#Importing necessary python libraries

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Load dataset
df = pd.read_csv('scholarshipdatabase.csv')

# Display the first few rows
df.head()
```

![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/a22929b7fca24b580f62be82c1afd8b539b3fb69/_posts/images/scholarship%20snapshot.png)
The libraries used are : 
- `numpy` for numerical operations.
- `pandas` for data manipulation and DataFrame operations.
- `matplotlib.pyplot` for data visualization

The dataset has various headings with a number of missing datas as shown above. The next step will be to explore the dataset further

## Exploring the Data
Now, let's use the `info()` method to get information about the DataFrame  including data types and the presence of any missing values.

``` python
# Get information about the dataset
print(df.info())
```
![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/dbabfe3bc4b66751f6acca4dbc48dfcce439eee6/_posts/images/info.png)

There are 704 entries with

### Selecting relevant features

Out of the 11 columns/features present in the DataFrame, we are only interested in 4. We will therefore create a new DataFrame with only the relevant features. 

``` Python
# New dataframe with selected features
features = ['Name', 'Area of specialisation', 'Country', 'Level needed']
data = df[features]
```

## Splitting Columns into Separate Rows
Certain columns, like 'Area of specialisation,' 'Country,' and 'Level needed,' contain multiple entries separated by commas. We split these entries into separate rows to make the data more accessible. 
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

This is how the dataset looks like now : 
