# Forecasting U.S. Inflation with Machine Learning

A machine learning study of **one-month-ahead U.S. CPI inflation forecasting** using macroeconomic indicators from FRED-MD.

The project asks a simple question:

> **Can machine learning improve short-term inflation forecasts over simply assuming that inflation will remain unchanged from the previous month?**

Multiple statistical and machine learning models are evaluated using time-series cross-validation, feature engineering, feature selection, hyperparameter tuning, and a final untouched holdout period.

The final selected model, **Elastic Net**, reduced holdout MAE by approximately **15.7%** relative to naive persistence.

---

## Project Overview

Inflation forecasting has practical importance for financial markets, monetary policy, businesses, and households. However, inflation is also persistent, which makes a simple forecast surprisingly competitive:

> **Next month's inflation ≈ this month's inflation**

This naive persistence forecast serves as the benchmark throughout the project.

Rather than assuming that greater model complexity leads to better forecasts, this project evaluates whether information from across the U.S. economy can improve upon that benchmark and how complex a model actually needs to be to provide forecasting value.

---

## Forecasting Target

The underlying price series is **CPIAUCSL**, the Consumer Price Index for All Urban Consumers.

The forecasting target is annualized month-over-month CPI inflation:

$begin:math:display$
\\pi\_t \= 1200
\\left\[
\\ln\(CPI\_t\) \- \\ln\(CPI\_\{t\-1\}\)
\\right\]
$end:math:display$

The primary task is to forecast:

$begin:math:display$
\\pi\_\{t\+1\}
$end:math:display$

using macroeconomic information available through month $begin:math:text$t$end:math:text$.

**Forecast horizon:** 1 month ahead  
**Primary evaluation metric:** Mean Absolute Error (MAE)

---

## Data

The project uses **FRED-MD**, a monthly macroeconomic database maintained by the Federal Reserve Bank of St. Louis.

The dataset used for this analysis contains:

- **126 original macroeconomic variables**
- **809 monthly observations**
- Coverage from **1959 through 2026**
- Indicators representing multiple areas of the U.S. economy

These include information related to:

- Labor markets
- Output and income
- Housing
- Consumption
- Prices
- Interest rates
- Money and credit
- Exchange rates
- Financial markets

After preprocessing and feature engineering, the modeling dataset contains **203 engineered predictors**.

### FRED-MD

FRED-MD was developed as a large monthly macroeconomic database for empirical macroeconomic research.

For additional information:

- Federal Reserve Bank of St. Louis, FRED-MD
- McCracken, M. W., & Ng, S. (2016). *FRED-MD: A Monthly Database for Macroeconomic Research*. Journal of Business & Economic Statistics, 34(4), 574–589.

---

## Methodology

The project follows a time-series forecasting workflow designed to prevent future information from leaking into model development.

### 1. Data Preparation

The raw FRED-MD data is cleaned and transformed using the transformation codes supplied with the dataset.

The workflow includes:

- Date validation
- Missing-data analysis
- FRED-MD transformations
- Target construction
- Outlier handling
- Scaling where required

### 2. Time-Aware Validation

The final **60 months** are reserved as an untouched holdout period.

All model selection, feature selection, and hyperparameter tuning decisions are made using only the development data.

Expanding-window time-series cross-validation is used rather than random train/test splitting to preserve the chronological structure of the forecasting problem.

### 3. Feature Engineering

The original macroeconomic variables are expanded using forecasting-oriented features, including:

- Autoregressive inflation features
- Lagged macroeconomic indicators
- Rolling summaries
- Economically motivated spread features

All features used to forecast month $begin:math:text$t\+1$end:math:text$ are constructed only from information available through month $begin:math:text$t$end:math:text$.

### 4. Feature Selection

`SelectKBest` with `f_regression` is evaluated on the strongest baseline models.

The number of retained predictors is treated as a model-specific choice and evaluated using cross-validation MAE.

Feature reduction did **not** improve cross-validation performance for the strongest models, so the final Elastic Net specification retains all **203 engineered predictors**.

### 5. Hyperparameter Tuning

The strongest baseline models are tuned using time-series cross-validation while keeping the final holdout completely separate from model selection.

---

## Models Evaluated

The project compares a range of statistical and machine learning approaches:

| Model | Type |
|---|---|
| Naive Persistence | Benchmark |
| Linear Regression | Linear |
| Ridge | Regularized Linear |
| Lasso | Regularized Linear |
| Elastic Net | Regularized Linear |
| Principal Component Regression (PCR) | Dimensionality Reduction + Linear |
| Partial Least Squares (PLS) | Latent-Factor Regression |
| RBF Support Vector Regression | Nonlinear Kernel |
| Random Forest | Tree Ensemble |
| Gradient Boosting | Boosted Tree Ensemble |
| SARIMAX | Time-Series Model |

The primary model-selection criterion is **cross-validation MAE**.

RMSE and R² are reported as secondary evaluation metrics.

---

## Final Model

### Elastic Net

Elastic Net was selected as the final model based **only on development-set cross-validation performance**.

| Setting | Final Value |
|---|---:|
| Model | Elastic Net |
| Alpha | **0.25** |
| L1 Ratio | **0.50** |
| Engineered Predictors | **203** |
| Forecast Horizon | **1 month** |

The holdout results were examined only after the model-selection decision had been made.

---

## Final Results

### Development Cross-Validation

| Model | Mean CV MAE | Mean CV RMSE | Mean CV R² |
|---|---:|---:|---:|
| **Elastic Net** | **2.0089** | **2.7687** | **0.1550** |
| Gradient Boosting | 2.1078 | 2.9327 | 0.0632 |
| Random Forest | 2.1272 | 3.0033 | 0.0274 |
| PCR | 2.1483 | 2.9349 | 0.0694 |

Naive persistence CV MAE:

**2.4816**

Elastic Net improvement over persistence:

**19.0%**

### Final 60-Month Holdout

| Metric | Elastic Net |
|---|---:|
| MAE | **2.0572** |
| RMSE | **2.9765** |
| R² | **0.2202** |
| Naive Persistence MAE | **2.4409** |
| Improvement vs. Persistence | **15.72%** |

Elastic Net ranked first during development cross-validation and maintained strong performance on the previously unseen holdout period.

---

## Predictor Importance

Predictor importance was examined using multiple complementary approaches to reduce reliance on any single importance measure.

The analysis showed that **recent inflation history provides much of the forecasting signal**.

Among the strongest predictors were:

| Predictor | Economic Interpretation |
|---|---|
| `inflation_t` | Current inflation |
| `inflation_lag_1` | Lagged inflation |
| `OILPRICEx` | Oil prices |
| `COMPAPFFx` | Interest-rate spread |
| `CPITRNSL` | Transportation CPI |

The broader macroeconomic feature set contributes additional information, but recent inflation behavior remains especially important for one-month-ahead forecasting.

---

## Robustness Analysis

The final Elastic Net model was evaluated across several alternative conditions.

### Forecast Horizons

Elastic Net outperformed naive persistence at all horizons tested:

- 1 month
- 3 months
- 6 months
- 12 months

Forecast errors increased at longer horizons, supporting the choice of **one month ahead** as the project's primary forecasting task.

### Alternative Information Sets

One of the strongest findings was the importance of inflation history.

| Feature Set | Holdout MAE | Improvement vs. Naive |
|---|---:|---:|
| All Features | 2.0572 | 15.72% |
| Inflation History Only | **2.0180** | **17.33%** |
| Macroeconomic Features Only | 2.4733 | -1.33% |
| All Except Current Inflation | 2.1223 | 13.06% |

Inflation-history features alone performed slightly better than the complete feature set on the holdout, while macroeconomic predictors without inflation history failed to outperform persistence.

This suggests that **recent inflation dynamics are the dominant source of short-term predictive information**.

### Different Inflation Regimes

Performance varied substantially across the holdout period.

The most difficult period was **2021–2022**, when inflation changed rapidly following the pandemic.

The model performed considerably better during the more stable inflation periods that followed.

This highlights an important limitation: **sudden inflation shocks remain difficult to forecast even when average forecasting performance is strong.**

---

## Key Findings

1. **Elastic Net produced the strongest overall forecasting performance.**

2. The final model improved holdout MAE by approximately **15.7%** relative to naive persistence.

3. **Greater model complexity did not necessarily improve forecasting accuracy.** A regularized linear model outperformed the more complex tree-based models evaluated in the final comparison.

4. **Recent inflation history was the strongest source of predictive information.**

5. Broader macroeconomic indicators provided additional signals, but were substantially less effective without inflation-history features.

6. Forecasting became more difficult as the prediction horizon increased.

7. Sudden inflation changes, particularly during **2021–2022**, remained difficult for the model to anticipate.

---

## Repository Structure

```text
cpi-inflation-forecasting/
│
├── Final_CPI_Prediction_Project.ipynb
├── README.md
└── .gitignore
```

`Final_CPI_Prediction_Project.ipynb` contains the complete analysis, including preprocessing, exploratory analysis, feature engineering, model comparison, feature selection, hyperparameter tuning, final evaluation, predictor analysis, and robustness testing.

---

## Running the Notebook

### Google Colab

The notebook can be uploaded directly to Google Colab and executed in a hosted Python environment.

### Local Environment

The notebook can also be opened with:

- JupyterLab
- Jupyter Notebook
- Visual Studio Code with Jupyter support

A Python environment with the required scientific-computing and machine-learning packages must be available.

The notebook itself documents the required imports and workflow.

> **Note:** Reproducing the project requires access to the corresponding FRED-MD source data used by the notebook. The repository does not redistribute third-party source data unless explicitly included.

---

## Reproducibility and Validation

Several design choices were used to keep the evaluation realistic:

- Chronological rather than random data splitting
- An untouched final holdout period
- Expanding-window cross-validation
- Fold-specific preprocessing and feature selection
- Development-only model selection
- Persistence benchmarking
- Separate final holdout evaluation

The final model was selected **before examining its holdout performance**, reducing the risk of choosing a model based on test-set results.

---

## Limitations

The results should be interpreted with several limitations in mind:

- Historical relationships between macroeconomic indicators and inflation may change over time.
- Sudden economic shocks are difficult to anticipate from historical data.
- Some macroeconomic series may be released with delays or subsequently revised.
- Strong holdout performance does not guarantee equivalent performance in future inflation regimes.
- The project evaluates forecasting performance using historical data and should not be interpreted as a production forecasting system or financial advice.

---

## Team

**California State University — COMP 542**

- Byron Najera
- Glenville Pecor
- Meet Akbari
- Josue Sanchez

---

## References

McCracken, M. W., & Ng, S. (2016). *FRED-MD: A Monthly Database for Macroeconomic Research*. **Journal of Business & Economic Statistics, 34**(4), 574–589.

Federal Reserve Bank of St. Louis. *FRED-MD: A Monthly Database for Macroeconomic Research.*

---

## Disclaimer

This repository was created as an academic machine learning project. The forecasts and results are provided for educational and research purposes only and should not be interpreted as financial, investment, or policy advice.
