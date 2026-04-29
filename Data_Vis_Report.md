# Airbnb Data Visualization and Descriptive Statistics Report

**Author:** Arda Tutmaz  
**Student ID:** 212134  

---

## 1. Project Introduction
This single Markdown file contains the data visualization case study for the Airbnb dataset. The objective is to practice data visualization using the `matplotlib` and `seaborn` libraries in Python to understand pricing, room types, and geographical distribution.

## 2. Loading the Dataset
To begin the analysis, we load the dataset directly from the raw GitHub link and inspect the first few rows.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Load the dataset
url = '[https://raw.githubusercontent.com/kflisikowsky/Descriptive_Statistics/refs/heads/main/data/airbnb.csv](https://raw.githubusercontent.com/kflisikowsky/Descriptive_Statistics/refs/heads/main/data/airbnb.csv)'
airbnb = pd.read_csv(url, index_col='Unnamed: 0')

# Display the first 5 rows
print(airbnb.head())
```

## 3. Descriptive Statistics
Before visualizing, we check the summary statistics of the dataset to understand the central tendencies and distributions of numerical features like price.

```python
# Summary statistics
print(airbnb.describe())
```
**Interpretation:** The descriptive statistics give us the mean, median, and max values, which indicate the presence of high-price outliers in our dataset.

## 4. Data Visualizations

### A. Price Distribution
We use a histogram to visualize how the prices of Airbnb listings are distributed.

```python
plt.figure(figsize=(10, 6))
sns.histplot(airbnb['price'], bins=50, kde=True, color='skyblue')
plt.title('Distribution of Airbnb Prices')
plt.xlabel('Price')
plt.ylabel('Frequency')
plt.show()
```
**Insight:** The distribution is highly right-skewed, meaning the vast majority of listings are clustered at lower price points, with a few luxury properties pulling the average up.

### B. Room Type Popularity
A count plot helps us understand the market supply by showing the frequency of each room type.

```python
plt.figure(figsize=(8, 5))
sns.countplot(data=airbnb, x='room_type', palette='viridis')
plt.title('Listing Counts by Room Type')
plt.xlabel('Room Type')
plt.ylabel('Count')
plt.show()
```
**Insight:** This visualization reveals whether "Entire homes/apt" or "Private rooms" are more common in this specific Airbnb market.

### C. Geographical Price Distribution
By plotting the latitude and longitude, we can map the listings. Adding price as the hue shows us the most expensive neighborhoods.

```python
plt.figure(figsize=(10, 6))
sns.scatterplot(data=airbnb, x='longitude', y='latitude', hue='price', palette='coolwarm', alpha=0.6)
plt.title('Geographical Distribution of Listings by Price')
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.show()
```
**Insight:** Prices are not evenly distributed; certain geographic clusters command much higher prices, likely indicating city centers or popular tourist areas.
