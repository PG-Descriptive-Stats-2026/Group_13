# Airbnb Data Visualization and Descriptive Statistics Report

**Authors:**
- Arda Tutmaz (212134) — Part 1
- Pierre-Antoine Andries (212129) — Part 2
- Berkant Cora (212130) — Part 3

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
url = 'https://raw.githubusercontent.com/kflisikowsky/Descriptive_Statistics/refs/heads/main/data/airbnb.csv'
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

---

## Part 2: Additional Analyses (Contributed by Pierre-Antoine Andries)

The Part 1 visualizations look at price overall, room-type frequency, and geography in isolation. Part 2 adds two analyses that address descriptive-statistics concepts not yet covered in the report: **how the price distribution varies *across* room types** (central tendency and dispersion within categories), and **how the numerical features relate to one another** (multivariate correlation).

### Data Preparation for Part 2

Two short cleanups are needed before the analyses below can run on the raw dataset:

```python
# Price is stored as a string with a "$" suffix (e.g. "45$") -> cast to float
airbnb['price_num'] = airbnb['price'].str.replace('$', '', regex=False).astype(float)

# Room type has trailing whitespace and case variants (e.g. "PRIVATE ROOM",
# "   Shared room") that split otherwise-identical categories. Normalize.
airbnb['room_type'] = airbnb['room_type'].str.strip().str.title()
```

**Note:** After normalization, two small residue categories remain (`Private` with 89 listings, `Home` with 66 listings) that look like truncated entries in the source data. They are kept in the analyses below as a transparent record of the underlying data quality, but the bulk of the ~9,950 listings sit in the three canonical categories `Entire Home/Apt`, `Private Room`, and `Shared Room`.

### D. Price Distribution by Room Type

The Part 1 histogram pools every listing into a single distribution, and the count plot shows how many listings exist per room type, but neither shows how *price itself* differs across room types. A box plot per room type makes the per-category median, IQR, whiskers, and outliers visible in a single figure.

```python
plt.figure(figsize=(10, 6))
sns.boxplot(data=airbnb, x='room_type', y='price_num', palette='viridis')
plt.title('Price Distribution by Room Type')
plt.xlabel('Room Type')
plt.ylabel('Price ($)')
plt.show()

summary = airbnb.groupby('room_type')['price_num'].agg(
    n='count', mean='mean', median='median', std='std',
    q1=lambda s: s.quantile(0.25), q3=lambda s: s.quantile(0.75)
)
summary['iqr'] = summary['q3'] - summary['q1']
print(summary.round(2))
```

**Insight:** `Entire Home/Apt` has by far the highest central tendency (median ≈ $163, mean ≈ $209) and the widest dispersion (IQR ≈ 110, std ≈ 251), confirming that this category is both the most expensive and the most variable. `Private Room` listings cluster much lower (median ≈ $70, IQR ≈ 44) and `Shared Room` listings are the cheapest (median ≈ $50). The mean exceeds the median in every category, which quantifies — per group — the right-skew that Part 1 observed on the pooled distribution.

### E. Correlation Matrix of Numerical Features

Part 1 is mostly univariate or bivariate-by-geography. A correlation heatmap brings all numerical features into one view to check whether `price` is related to review activity, availability, or rating, and to detect redundancy between features.

```python
num_cols = ['price_num', 'number_of_reviews', 'reviews_per_month',
            'availability_365', 'rating', 'number_of_stays', '5_stars']
corr = airbnb[num_cols].corr(method='pearson')

plt.figure(figsize=(8, 6))
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm',
            vmin=-1, vmax=1, center=0, square=True)
plt.title('Pearson Correlation — Numerical Features')
plt.show()
```

**Insight:** Three findings stand out.
1. `price_num` correlates only weakly with every other numerical feature (|r| < 0.1). This is a strong negative finding: price is **not** explained by review counts, availability, or rating, which means the price drivers shown in Part 1 — location and room type — are not just one signal among many but the dominant ones.
2. `number_of_reviews` and `number_of_stays` are perfectly correlated (r = 1.00). They carry the same signal and one of them is redundant for any further modeling. `reviews_per_month` correlates ~0.54 with both, which is consistent with it being a derived rate rather than an independent measurement.
3. `rating` and `5_stars` correlate moderately (r ≈ 0.36) but not strongly, suggesting the share of 5-star reviews is only a partial proxy for the overall rating.

### Conclusion of Part 2

Part 1 establishes the marginal distributions of price, room type, and geography. Part 2 sharpens the picture by showing that price distributions differ substantially **between** room types (D), and that — among the available numerical features — none of them adds meaningful explanatory signal for price beyond what room type and location already provide (E). Together, the two parts give a coherent first answer to the report's question: in this Airbnb market, price is primarily a categorical and spatial phenomenon, not a function of review activity or availability.

---

## Part 3: Additional Analyses (Contributed by Berkant Cora)

Part 3 extends the report with location-level and robustness-oriented checks. The goal is to answer two practical questions:

- Which boroughs and neighbourhoods are the most expensive on average?
- Do the main price patterns still hold after reducing the effect of extreme outliers?

### F. Borough and Neighbourhood Price Ranking

To make location analysis easier, we split the combined location column into borough and neighbourhood, then compare median prices.

```python
# Split neighbourhood_full into borough and neighbourhood
loc_parts = airbnb['neighbourhood_full'].str.split(',', n=1, expand=True)
airbnb['borough'] = loc_parts[0].str.strip()
airbnb['neighbourhood'] = loc_parts[1].str.strip()

# Borough-level median price
borough_price = (
    airbnb.groupby('borough')['price_num']
    .median()
    .sort_values(ascending=False)
)
print('Median price by borough:')
print(borough_price)

# Top 10 neighbourhoods by median price (with a minimum sample filter)
neigh_stats = (
    airbnb.groupby('neighbourhood')['price_num']
    .agg(median='median', n='count')
    .query('n >= 20')
    .sort_values('median', ascending=False)
    .head(10)
)
print('\nTop 10 neighbourhoods by median price (n >= 20):')
print(neigh_stats)
```

```python
plt.figure(figsize=(9, 5))
sns.barplot(x=borough_price.values, y=borough_price.index, palette='mako')
plt.title('Median Airbnb Price by Borough')
plt.xlabel('Median Price ($)')
plt.ylabel('Borough')
plt.show()
```

Insight: Price differences are strongly location-dependent. Borough medians and neighbourhood-level ranking confirm that location is a primary pricing driver, consistent with the earlier geographical visualization.

### G. Availability and Price Relationship

This check tests whether listings that are available for more days per year tend to be cheaper or more expensive.

```python
plt.figure(figsize=(8, 5))
sns.regplot(
    data=airbnb,
    x='availability_365',
    y='price_num',
    scatter_kws={'alpha': 0.15, 's': 20},
    line_kws={'color': 'red'}
)
plt.title('Availability vs Price')
plt.xlabel('Availability (days per year)')
plt.ylabel('Price ($)')
plt.ylim(0, airbnb['price_num'].quantile(0.99))
plt.show()

print('Correlation (availability_365, price_num):',
      airbnb[['availability_365', 'price_num']].corr().iloc[0, 1])
```

Insight: The relationship is weak in magnitude, meaning availability alone is not a strong predictor of price in this dataset.

### H. Outlier-Robust Price Comparison by Room Type

High-end listings can inflate averages. To test robustness, we compare room-type medians before and after trimming the top 1% of prices.

```python
# Baseline medians
baseline = airbnb.groupby('room_type')['price_num'].median().rename('baseline_median')

# Trim upper 1% outliers
cap = airbnb['price_num'].quantile(0.99)
trimmed = airbnb[airbnb['price_num'] <= cap]
trimmed_median = trimmed.groupby('room_type')['price_num'].median().rename('trimmed_median')

robust_check = pd.concat([baseline, trimmed_median], axis=1)
robust_check['difference'] = robust_check['baseline_median'] - robust_check['trimmed_median']
print(robust_check.sort_values('baseline_median', ascending=False))
```

Insight: Room-type ordering remains stable after outlier trimming. This supports the reliability of the main conclusion that entire homes/apartments are priced highest, followed by private rooms, then shared rooms.

### Conclusion of Part 3

Part 3 confirms that location is a major determinant of pricing and that the core room-type price hierarchy is robust even when extreme values are reduced. Together with Parts 1 and 2, this strengthens the overall interpretation that Airbnb pricing in this dataset is primarily structured by place and listing type rather than review activity metrics.
