# Airbnb Data Cleansing Report

## Group Information
- Group number: 13
- Members:
    - **Pierre-Antoine Andries (212129)**
    - **Arda Tutmaz (212134)**
    - **Berkant Cora (212130)**
- Course: Descriptive Statistics 2026

## Source
- Tutorial followed: [Data Wrangling and Cleansing](https://kflisikowsky.github.io/Descriptive_Statistics/wrangling/)
- Dataset: [airbnb.csv](https://github.com/kflisikowsky/Descriptive_Statistics/blob/main/data/airbnb.csv)

## Contribution Split
- **Pierre-Antoine Andries** — Stages 1–2 (initial diagnosis, type conversion)
- **Arda Tutmaz** — Stages 3–4 (splitting compound columns, categorical normalization)
- **Berkant Cora** — Stages 5–6 (duplicate removal, missing-value handling)

## Objective
Apply the cleansing techniques from the tutorial — missing-data handling, type conversion / `pd.to_datetime`, error correction, and string clean-up — to the raw `airbnb.csv` dataset and produce a single tidy DataFrame ready for descriptive analysis. Each stage below operates on the same `airbnb` DataFrame so that the final state at the bottom of the report is the cumulative result of every preceding stage.

## Stage 0 — Loading the Raw Dataset

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

url = 'https://raw.githubusercontent.com/kflisikowsky/Descriptive_Statistics/refs/heads/main/data/airbnb.csv'
airbnb = pd.read_csv(url, index_col='Unnamed: 0')
print(airbnb.shape)   # (10019, 16)
```

---

## Stage 1 — Initial Diagnosis (Pierre-Antoine Andries)

Before any cleansing decision, we inspect the dataset's structure, missingness pattern, duplication, and the categorical columns that look suspect.

```python
print(airbnb.shape)
print(airbnb.dtypes)
print(airbnb.isna().sum())
print(airbnb.duplicated().sum(), airbnb['listing_id'].duplicated().sum())
print(airbnb['room_type'].value_counts())
```

**Findings**

| Check | Result | Implication |
|---|---|---|
| Shape | 10,019 rows × 16 cols | Baseline |
| `price` dtype | `object` (e.g. `"45$"`) | Cast to numeric in Stage 2 |
| `last_review`, `listing_added` dtype | `object` | Convert with `pd.to_datetime` in Stage 2 |
| Missing values | 238 in `price`, 2,075 in each review-related field, 5 in `name` | Handled in Stage 6 |
| Duplicates | 13 fully-duplicated rows; 20 duplicated `listing_id` | Dedup in Stage 5 |
| `room_type` unique values | 7 categories instead of ~3 (`"PRIVATE ROOM"`, `"   Shared room"`, `"home"`, `"Private"`, …) | Normalize in Stage 4 |
| `coordinates` | Single string `"(40.63, -73.93)"` | Split in Stage 3 |
| `neighbourhood_full` | Single string `"Brooklyn, Flatlands"` | Split in Stage 3 |

## Stage 2 — Type Conversion (Pierre-Antoine Andries)

Two type conversions are needed: numeric for `price`, datetime for the two date columns. `errors='coerce'` turns malformed timestamps into `NaT`, which is the tutorial's recommended way to absorb parse failures without dropping rows.

```python
# Price: strip the trailing "$" and cast to float
airbnb['price'] = airbnb['price'].str.replace('$', '', regex=False).astype(float)

# Dates: parse to datetime; unparseable values become NaT
airbnb['last_review']   = pd.to_datetime(airbnb['last_review'],   errors='coerce')
airbnb['listing_added'] = pd.to_datetime(airbnb['listing_added'], errors='coerce')

print(airbnb[['price', 'last_review', 'listing_added']].dtypes)
```

**Result:** `price` is now `float64`, both date columns are `datetime64[ns]`. Numerical statistics (mean, median, std) and time-based filtering are now possible directly on these columns.

---

## Stage 3 — Splitting Compound Columns (Arda Tutmaz)

`coordinates` and `neighbourhood_full` each pack two pieces of information into one string. We extract them into separate columns and drop the originals to keep the schema tidy.

```python
# coordinates "(40.63222, -73.93398)" -> latitude / longitude
coords = airbnb['coordinates'].str.extract(r'\(([^,]+),\s*([^)]+)\)')
airbnb['latitude']  = coords[0].astype(float)
airbnb['longitude'] = coords[1].astype(float)

# neighbourhood_full "Brooklyn, Flatlands" -> borough / neighbourhood
airbnb[['borough', 'neighbourhood']] = airbnb['neighbourhood_full'].str.split(',', n=1, expand=True)
airbnb['neighbourhood'] = airbnb['neighbourhood'].str.strip()

airbnb = airbnb.drop(columns=['coordinates', 'neighbourhood_full'])
```

**Result:** four new analysis-ready columns (`latitude`, `longitude`, `borough`, `neighbourhood`), all properly typed. The geographic scatter plot from the visualization report can now run as written, and grouping by `borough` or `neighbourhood` becomes a one-liner.

## Stage 4 — Categorical Normalization (Arda Tutmaz)

Stage 1 revealed that `room_type` has 7 distinct values where there should be ~3. After stripping whitespace and title-casing, two small residue categories remain — `Private` (89 listings) and `Home` (66 listings) — which look like truncated entries of `Private Room` and `Entire Home/Apt`. We map them to the canonical labels.

```python
airbnb['room_type'] = airbnb['room_type'].str.strip().str.title()
airbnb['room_type'] = airbnb['room_type'].replace({
    'Private': 'Private Room',
    'Home':    'Entire Home/Apt',
})
print(airbnb['room_type'].value_counts())
```

**Result:** the column collapses to three canonical categories — `Entire Home/Apt` (5,186), `Private Room` (4,607), `Shared Room` (226). All earlier groupings, plots, and summary tables produced on `room_type` will now operate on consistent labels.

---

## Stage 5 — Duplicate Removal (Berkant Cora)

Two duplication checks were run in Stage 1: 13 fully-duplicated rows and 20 duplicate `listing_id` values. The second number is more conservative: each `listing_id` is supposed to identify a unique listing, so any row whose id is already present is suspect even if other fields differ slightly. We dedup on `listing_id`, which subsumes the fully-duplicated rows.

```python
print(f'Before dedup: {len(airbnb)}')
airbnb = airbnb.drop_duplicates(subset=['listing_id'], keep='first')
print(f'After dedup:  {len(airbnb)}')
```

**Result:** 20 rows dropped (10,019 → 9,999). Using `subset=['listing_id']` is the right choice here because the entity of interest is the *listing*, not the row — even imperfect duplicates of the same listing would inflate counts and bias means.

## Stage 6 — Missing-Value Handling (Berkant Cora)

The tutorial proposes three strategies: drop, constant-fill, and statistical imputation. The right choice is column-by-column.

```python
# Visualize the missingness pattern (tutorial: sns.heatmap on isna())
plt.figure(figsize=(10, 4))
sns.heatmap(airbnb.isna(), cbar=False)
plt.title('Missing-value pattern')
plt.show()
```

**Strategy per column:**

| Column | NaN count | Strategy | Rationale |
|---|---|---|---|
| `price` | 238 | **Drop the row** (`dropna(subset=['price'])`) | Price is the primary analysis variable; imputing it would invent the very signal we want to study. |
| `reviews_per_month`, `number_of_stays`, `5_stars` | ~2,024 each | **Fill with 0** | The NaN pattern is co-located: these fields are missing exactly when `last_review` is missing, i.e. listings that have never been reviewed. Zero is the *true* value, not an imputation. |
| `last_review` | ~2,024 | **Leave as `NaT`** | "Never reviewed" is meaningful — replacing it with a fake date would be misleading. |
| `rating` | ~2,024 | **Cap at 5.0, then median-impute** | The raw column contains values up to 5.18, which is invalid (rating scale tops at 5). We first clip the invalid range, then impute the remaining NaNs with the median (tutorial: statistical imputation). |

```python
# Drop rows missing the analysis-critical price
airbnb = airbnb.dropna(subset=['price'])

# Fill review counts with 0 (NaN here means "no reviews yet")
zero_fill = ['reviews_per_month', 'number_of_stays', '5_stars']
airbnb[zero_fill] = airbnb[zero_fill].fillna(0)

# Cap invalid rating values, then median-impute the remaining NaN
airbnb.loc[airbnb['rating'] > 5, 'rating'] = 5.0
airbnb['rating'] = airbnb['rating'].fillna(airbnb['rating'].median())
```

**Result:** all critical columns are now NaN-free. `last_review` retains its `NaT`s on purpose. The 5 missing `name` and 2 missing `host_name` values are kept untouched — they are display fields, not analysis fields.

---

## Final Cleaned Dataset

```python
print(airbnb.shape)
print(airbnb.dtypes)
print(airbnb.isna().sum())
```

| Metric | Before | After |
|---|---|---|
| Rows | 10,019 | **9,761** (−258: 20 duplicates + 238 missing price) |
| Columns | 16 | **18** (+2: `latitude`, `longitude`, `borough`, `neighbourhood` added; `coordinates`, `neighbourhood_full` dropped) |
| `price` dtype | `object` | `float64` |
| Date dtypes | `object` | `datetime64[ns]` |
| `room_type` categories | 7 | **3** |
| Critical-column NaNs | 238 (price), 2,075×4 (reviews) | **0** |
| Invalid `rating` (> 5.0) | present | clipped to 5.0 |

## Conclusion

The pipeline applies six cleansing techniques drawn directly from the tutorial — diagnosis with `isna()` / `info()`, `pd.to_datetime`, string operations (`str.extract`, `str.split`, `str.strip`, `str.title`), `drop_duplicates`, `dropna(subset=...)`, constant-`fillna`, error correction by clipping, and median imputation. The result is a tidy 9,761-row DataFrame with eighteen properly-typed columns that downstream descriptive-statistics work (the visualization report, in particular) can consume without any further preprocessing.
