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
In this article, we will learn the process of cleaning, handling missing data and restructuring the dataset  for building 3SA.  The data used in this project was manually collected and the raw dataset can be found [here](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv). 

## Importing Library and Loading the Scholarship Dataset

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Load dataset
df = pd.read_csv('scholarshipdatabase.csv')

# Display the first few rows
df.head()
```
This project required 3 libraries :
- `numpy` for numerical operations.
- `pandas` for data manipulation and DataFrame operations.
- `matplotlib.pyplot` for data visualisation
  
The necessary libraries are imported and the [scholarship dataset](https://raw.githubusercontent.com/barbaraaboagye/My-MachineLearning-Journey/1e19a3a7caf86f8b0603ed100144ff94d536a769/Projects/Scholarship%20recommender%20system/scholarshipdatabase.csv) is loaded into a Pandas DataFrame using the `read_csv`. The first few rows of the dataset are then displayed as shown in the snapshot below :

![](https://github.com/barbaraaboagye/barbaraaboagye.github.io/blob/a22929b7fca24b580f62be82c1afd8b539b3fb69/_posts/images/scholarship%20snapshot.png)

## Exploring the Data
``` python
# Get information about the dataset
print(df.info())
```
The `info()` method provides a snapshot, including data types and the presence of any missing values.
