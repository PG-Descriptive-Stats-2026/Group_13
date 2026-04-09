# Lab 3 - Data Wrangling Report

## Group Information
- Group number: 13

- Members:
    - **Berkant Cora (212130)**
    - **Pierre-Antoine Andries (212129)**
    - **Arda Tutmaz (212134)**

- Course: Descriptive Statistics 2026

## Source
- Instructor DS repository: [Data_Analysis_Workshops](https://github.com/kflisikowsky/Data_Analysis_Workshops)
- Working notebook used: [Report1.ipynb](https://github.com/kflisikowsky/Descriptive_Statistics/blob/main/Report1.ipynb)

## Contribution Split
- Berkant Cora: Exercises 1 to 3
- Arda Tutmaz: Exercises 4 to 6

## Exercise 1
### Task
Select the columns `mpg` and `horsepower` from the cars DataFrame.

### Code
```python
cars[["mpg", "horsepower"]]
```

### Result (summary)
A two-column DataFrame with 398 rows is returned, containing only `mpg` and `horsepower`.

### Interpretation
This operation narrows the dataset to variables directly related to fuel efficiency and engine power.

## Exercise 2
### Task
Select the columns `mpg` and `horsepower` from the cars DataFrame using `drop`.

### Code
```python
cars.drop(columns=cars.columns.difference(["mpg", "horsepower"])).head()
```

### Result (summary)
The output displays only `mpg` and `horsepower` for the first rows.

### Interpretation
Using `drop` with inverted selection is an alternative way to keep specific variables and remove the rest.

## Exercise 3
### Task
Select all columns except `model_year` and `name` from the cars DataFrame.

### Code
```python
cars.drop(columns=["model_year", "name"]).head()
```

### Result (summary)
The DataFrame keeps all analytical numeric columns and `origin`, while excluding year and model name.

### Interpretation
This simplifies the table for many wrangling and summary operations by removing columns that may be unnecessary for a specific task.

## Short Conclusion
The three exercises demonstrate core data wrangling patterns in pandas: selecting variables directly, selecting through inverse dropping, and excluding specific columns for a cleaner analysis table.

---
## Part 2: Implementation of Additional Exercises (Contributed by Arda Tutmaz)

Following our group strategy, I have implemented three additional exercises from the DS repository to demonstrate various data wrangling techniques such as filtering, aggregating, and basic data exploration.

### Exercise 4: Chipotle Order Analysis
**Objective:** Exploring the frequency and quantity of restaurant orders.

```python
import pandas as pd

# Load dataset
url = 'https://raw.githubusercontent.com/justmarkham/DAT8/master/data/chipotle.tsv'
chipo = pd.read_csv(url, sep = '\t')

# Display the first 5 rows
print(chipo.head(5))

# Identify the most ordered item
most_ordered = chipo.groupby('item_name').quantity.sum().sort_values(ascending=False).head(1)
print(f"\nMost ordered item and quantity: \n{most_ordered}")
```

### Exercise 5: Euro 2012 Team Statistics
**Objective**: Filtering and sorting sports data based on performance metrics.

```python
import pandas as pd

# Load dataset
url = 'https://raw.githubusercontent.com/guipsamora/pandas_exercises/master/02_Filtering_%26_Sorting/Euro12/Euro_2012_stats_TEAM.csv'
euro12 = pd.read_csv(url)

# View teams that scored more than 6 goals
high_scoring_teams = euro12[euro12['Goals'] > 6]
print("Teams with more than 6 goals:")
print(high_scoring_teams[['Team', 'Goals']])
```
### Exercise 6: Global Drinks Consumption
**Objective**: Using grouping functions to find regional averages.

```python
import pandas as pd

# Load dataset
url = 'https://raw.githubusercontent.com/justmarkham/DAT8/master/data/drinks.csv'
drinks = pd.read_csv(url)

# Average beer consumption per continent
avg_beer = drinks.groupby('continent').beer_servings.mean()
print("Average beer servings by continent:")
print(avg_beer)
```

## Conclusion & Key Takeaways

In this Lab 3 assignment, our group successfully collaborated using Git and GitHub to perform comprehensive data wrangling tasks. By selecting and solving multiple exercises from the course repository, we demonstrated our proficiency in several core data science skills:

* **Data Exploration:** Understanding the structure, dimensions, and data types of various datasets (such as Chipotle, Euro12, and Drinks).
* **Data Manipulation with Pandas:** Utilizing essential functions such as `.groupby()`, `.filter()`, and `.sort_values()` to extract meaningful insights.
* **Data Cleaning:** Handling different file formats (like CSV and TSV) and managing specific delimiters.
* **Collaborative Workflow:** Effectively using a shared repository to divide tasks, integrate individual contributions, and document our findings in a unified Markdown report.

Overall, these exercises provided valuable hands-on experience in transforming raw data into structured, analyzable formats. This collaborative approach not only strengthened our technical Pandas skills but also improved our version control practices for future data engineering projects.
