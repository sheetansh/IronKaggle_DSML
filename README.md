# IronKaggle — House Price Prediction (King County)

## Overview

This project builds and compares regression models to predict house prices in King County, WA. The analysis covers feature engineering, correlation analysis, feature importance, and model evaluation across three algorithm families.

**Dataset:** King County Houses (`king_country_houses_aa.csv`)
- 21,613 rows × 21 original columns
- Target variable: `price`
- No missing values, no duplicates

---

## Dataset Features

| Column | Description |
|---|---|
| `price` | Sale price (target) |
| `bedrooms` / `bathrooms` | Room counts |
| `sqft_living` / `sqft_lot` | Living area and lot size (sq ft) |
| `floors` | Number of floors |
| `waterfront` | Waterfront property (0/1) |
| `view` | View quality score |
| `condition` / `grade` | Condition and construction quality |
| `sqft_above` / `sqft_basement` | Above-ground and basement sq ft |
| `yr_built` / `yr_renovated` | Year built and last renovated |
| `zipcode` / `lat` / `long` | Location data |
| `sqft_living15` / `sqft_lot15` | Neighbour avg living/lot sq ft |

---

## Feature Engineering

New features derived from the original data:

| New Feature | Logic |
|---|---|
| `Age_at_Sale` | `year_sold − yr_built` |
| `Year_since_Renovation` | `year_sold − yr_renovated` (falls back to `Age_at_Sale` if never renovated) |
| `was_renovated` | Boolean — True if `yr_renovated > 0` and `yr_renovated > yr_built` |
| `day_sold` | Day of month extracted from `date` |

**Dropped columns:** `date`, `id`, `yr_built`, `year_sold`, `month_sold`

**Final dataset shape:** 21,613 rows × 22 columns

---

## Train / Test Split

| Set | Samples | Proportion |
|---|---|---|
| Train | 17,290 | 80% |
| Test | 4,323 | 20% |

`random_state = 13`

---

## Feature Importance Analysis

### Lasso (L1) — Top Features by Absolute Coefficient

| Feature | Lasso Coefficient |
|---|---|
| `sqft_living` | 260,667 |
| `yr_renovated` | 115,540 |
| `grade` | 113,464 |
| `was_renovated` | 105,056 |
| `lat` | 83,899 |
| `sqft_above` | 81,996 |
| `Age_at_Sale` | 70,494 |

> Lasso aggressively penalises multicollinearity — `sqft_living` dominates here.

### Ridge (L2) — Key Differences from Lasso

| Feature | Ridge Coefficient |
|---|---|
| `yr_renovated` | 567,540 |
| `was_renovated` | 553,981 |
| `grade` | 113,284 |
| `sqft_living` | 83,031 |

> Ridge retains correlated features and redistributes weight — renovation features balloon significantly compared to Lasso.

### Random Forest — Feature Importance Scores

| Feature | Importance |
|---|---|
| `grade` | 0.3135 |
| `sqft_living` | 0.2746 |
| `lat` | 0.1594 |
| `long` | 0.0701 |
| `sqft_living15` | 0.0306 |
| `waterfront` | 0.0254 |

### XGBoost — Feature Importance Scores

| Feature | Importance |
|---|---|
| `grade` | 0.3821 |
| `sqft_living` | 0.1693 |
| `waterfront` | 0.1444 |
| `lat` | 0.0815 |
| `long` | 0.0424 |
| `view` | 0.0341 |

> Both tree-based models consistently rank `grade` and `sqft_living` as the top two predictors.

---

## Model Results

All metrics computed on train and test splits. Models are compared by R², Adjusted R², MAE, MAPE, and RMSE.

| Model | Split | R² | Adj. R² | MAE | MAPE | RMSE |
|---|---|---|---|---|---|---|
| Linear Regression | Train | 0.6977 | 0.6973 | 125,915 | 25.6% | 202,891 |
| Linear Regression | Test | 0.7165 | 0.7151 | 125,994 | 25.9% | 191,440 |
| Random Forest (n=110) | Train | 0.9820 | 0.9820 | 25,980 | 4.9% | 49,539 |
| Random Forest (n=110) | Test | 0.8956 | 0.8950 | 67,607 | 12.8% | 116,191 |
| Random Forest (default) | Train | 0.9819 | 0.9819 | 26,031 | 4.9% | 49,599 |
| Random Forest (default) | Test | 0.8946 | 0.8941 | 67,782 | 12.8% | 116,702 |
| XGBoost (n=100) | Train | 0.9761 | 0.9761 | 40,792 | 9.0% | 57,067 |
| **XGBoost (n=100)** | **Test** | **0.9038** | **0.9033** | **66,774** | **12.7%** | **111,528** |

---

## Key Findings

1. **Best model on test data: XGBoost** — R² of **0.9038**, MAE ~$66,774, RMSE ~$111,528. It generalises better than Random Forest despite lower training R².

2. **Random Forest overfits slightly** — train R² of 0.982 vs test R² of 0.896 shows a gap of ~0.086, indicating moderate overfitting.

3. **Linear Regression underperforms** — R² of ~0.70 suggests the relationship between features and price is non-linear. This is a useful baseline, but insufficient on its own.

4. **Top price drivers (consensus across models):**
   - `grade` — construction and design quality is the strongest single predictor
   - `sqft_living` — living area size
   - `lat` — geographic latitude (neighbourhood effect)
   - `waterfront` — premium for waterfront properties

5. **Renovation features (`was_renovated`, `yr_renovated`)** are heavily weighted by Lasso/Ridge but rank lower in tree models, suggesting they contribute marginal gain over `grade` and size in non-linear models.

6. **Feature engineering added value** — `Age_at_Sale` and `Year_since_Renovation` both appeared in the top-10 features for Lasso, confirming that property age and recency of renovation carry pricing information.

---

## Files

| File | Description |
|---|---|
| `Ironkaggle_FeatureEngineering.ipynb` | Main analysis notebook — feature engineering, correlation, feature importance, and model training/evaluation |
| `Presentation_Ironkaggle.pptx` | Project presentation — summarises the dataset, methodology, model results, and key findings |
