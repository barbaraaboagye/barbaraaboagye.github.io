---
layout:     post
title:      Data cleaning and school and scholarship search (3SA)
subtitle:   
date:       2023-11-15
author:     "Barbara Aboagye"
header-img: 
---

# 1 Building a School and Scholarship Search App (3SA): Data Pre-processing And Cleaning 


## Loading the Scholarship Dataset

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Load dataset
df = pd.read_csv('scholarshipdatabase.csv')

# Display the first few rows
df.head()
