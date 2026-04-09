# Lab 3 - Data Wrangling Report

## Group Information
- Group number: 13

- Members:
    - Berkant Cora (212130)
    - Pierre-Antoine Andries (212129)
    - Arda Tutmaz (212134)

- Course: Descriptive Statistics 2026

## Source
- Instructor DS repository: [Data_Analysis_Workshops](https://github.com/kflisikowsky/Data_Analysis_Workshops)
- Working notebook used: [Report1.ipynb](../Report1.ipynb)

## Exercise 1
### Task
Select the columns `mpg` and `horsepower` from the cars DataFrame.

### Code
```python
cars >> select(X.mpg, X.horsepower)
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
cars >> drop(~X.mpg, ~X.horsepower) >> head()
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
cars >> select(~X.model_year, ~X.name) >> head()
```

### Result (summary)
The DataFrame keeps all analytical numeric columns and `origin`, while excluding year and model name.

### Interpretation
This simplifies the table for many wrangling and summary operations by removing columns that may be unnecessary for a specific task.

## Short Conclusion
The three exercises demonstrate core data wrangling patterns in `dfply`: selecting variables directly, selecting through inverse dropping, and excluding specific columns for a cleaner analysis table.
