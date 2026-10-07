# FreshBasket Retail Customer Churn Prediction

## 1. Project Overview

FreshBasket operates a retail loyalty program with Silver, Gold, and
Platinum membership tiers. The business currently lacks a systematic way
to identify members who are likely to churn and therefore may send
retention offers too broadly.

This project develops a customer-level churn prediction workflow to:

-   identify members at elevated near-term churn risk,
-   understand behavioural, engagement, and support signals associated
    with churn,
-   compare predictive performance using a leakage-aware cohort
    validation setup,
-   assess whether behavioural and engagement features add value beyond
    demographics,
-   segment churn risk by membership tier, city, and signup cohort, and
-   demonstrate how a targeted retention intervention could be simulated
    for high-risk members.

The project follows the assessment requirements for **Customer Churn
Prediction for a Retail Loyalty Program**.

## 2. Business Objective

The primary objective is to build a classification-based churn
prediction solution that can estimate the probability that a
loyalty-program customer will churn in the defined churn window.

The analysis focuses on signals available before the churn label period,
with particular attention to purchase behaviour, spending behaviour, app
and email engagement, coupon and promotion usage, customer support
activity, complaint behaviour, membership tier, demographics, and
tenure.

The intended business use is targeted retention rather than sending the
same offer to every loyalty member.

## 3. Dataset

The supplied Excel workbook contains three related tables:

### Customer Profile

Customer ID, signup date, age, gender, city, membership tier, marketing
opt-in, preferred store/category, and home store price tier.

### Monthly Activity

Month, transactions, total spend, average basket value, distinct
categories, promotional transaction percentage, app sessions, emails
opened, coupons redeemed, support tickets, and complaint flag.

### Churn Label

Observation end date, last purchase date, months observed,
insufficient-history flag, and churned.

The activity data covers **18 months from January 2023 through June
2024** and represents approximately 2,600 loyalty members before
modelling exclusions.

The churn definition is based on **zero purchase transactions during
April--June 2024 despite purchase history earlier in the observation
period**.

## 4. Project Structure

``` text
FreshBasket-Churn/
├── data/
│   ├── raw/
│   │   └── FreshBasket_Loyalty_Churn_Dataset.xlsx
│   └── processed/
├── notebooks/
│   ├── 01_data_quality.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_validation_setup.ipynb
│   ├── 05_models.ipynb
│   ├── 06_explainability.ipynb
│   ├── 07_ablation.ipynb
│   └── 08_retention_simulation.ipynb
├── outputs/
└── README.md
```

## 5. Data Quality Checks and Cleaning

The data-quality stage checked duplicate records, duplicate
customer-months, missing values, negative spend, negative transactions,
invalid basket values, zero-transaction anomalies, missing activity
months, unmatched customer IDs, categorical consistency, and spend
outliers.

### Key findings

-   Customer profile rows: **2,600**
-   Monthly activity rows before modelling cleaning: **28,015**
-   Churn label rows before insufficient-history filtering: **2,600**
-   Duplicate profile customers: **0**
-   Duplicate churn customers: **0**
-   Duplicate customer-month records: **2**
-   Negative spend rows: **24**
-   Negative transaction rows: **0**
-   Negative basket values: **0**
-   Zero-transaction rows: **2,330**
-   Zero transactions with non-zero spend: **0**
-   Activity months present: **18 out of 18**
-   Customers with internal activity-month gaps: **769**

### Cleaning decisions

1.  Negative total-spend records were removed.
2.  Duplicate customer-month records were resolved after negative-spend
    filtering by retaining the record with the highest total spend.
3.  `HOME_STORE_PRICE_TIER` was removed because it was completely
    missing.
4.  Missing age values were imputed using the median age.
5.  Missing gender values were represented as `Unknown`.
6.  Promotional transaction percentage was constrained to the valid 0--1
    range.
7.  Average basket value was recalculated from total spend and
    transactions when transactions were greater than zero.
8.  Customers flagged with insufficient history were excluded from the
    modelling label set.

These are documented modelling assumptions and should be reviewed if the
source system provides additional data-quality definitions.

## 6. Exploratory Data Analysis

The cleaned churn label population contained:

  Outcome         Customers    Share
  ------------- ----------- --------
  Not churned         1,782   68.54%
  Churned               818   31.46%

Churn was compared across membership tier, city, and signup cohort.

The EDA showed, for example, churn rates of 37.59% in Austin, 36.19% in
Columbus, 33.33% in Nashville, and 31.98% in Boise. These are
descriptive associations and should not be interpreted as causal
effects.

## 7. Feature Engineering

A customer-level modelling dataset was constructed using information
available up to the end of **March 2024**, before the April--June 2024
churn label window.

Features included:

-   total and average spend,
-   spend variability,
-   total and average transactions,
-   average basket value,
-   distinct categories,
-   active and observed months,
-   recent spend and transactions,
-   spend and transaction change,
-   app sessions,
-   emails opened,
-   coupons redeemed,
-   support tickets,
-   complaint rate,
-   support-ticket trend,
-   customer demographics and tier,
-   marketing opt-in and preferred category,
-   tenure,
-   explicit `months_since_last_purchase`,
-   rolling three-month spend, transactions, app sessions, emails, and
    support tickets.

The final customer feature dataset contains **2,225 customers and 51
columns**.

## 8. Validation Strategy

A random train/test split was not used as the primary validation
strategy.

Customers were ordered by signup date and divided using an
**80th-percentile signup-date cohort split**.

  Dataset        Customers   Churned   Churn rate
  ------------ ----------- --------- ------------
  Train              1,780       615       34.55%
  Validation           445        47       10.56%

The lower validation churn rate represents cohort distribution shift and
is an important limitation.

## 9. Modelling

Two primary approaches were evaluated:

-   **Logistic Regression** as a transparent baseline.
-   **Random Forest** as a nonlinear classifier with class balancing.

The earlier cohort-validation comparison was:

  Model                   Accuracy   Precision   Recall       F1   PR-AUC
  --------------------- ---------- ----------- -------- -------- --------
  Logistic Regression       0.8652      0.4217   0.7447   0.5385   0.6204
  Random Forest             0.7596      0.2581   0.6809   0.3743   0.6619

Because recency and rolling features were subsequently added, the
**latest regenerated files in `outputs/` should be treated as the final
model results if they differ from these earlier values**.

Class imbalance was addressed through class balancing and evaluation
with precision, recall, F1, and PR-AUC rather than accuracy alone.

A **0.70 probability threshold** was used for the high-risk operating
scenario after threshold analysis.

## 10. Explainability

Feature importance analysis showed that recent behavioural changes and
engagement signals were among the strongest predictive variables.

Earlier Random Forest importance results included:

1.  `transaction_change`
2.  `spend_change`
3.  `recent_transactions`
4.  `app_session_change`
5.  `recent_spend`
6.  `recent_app_sessions`
7.  `recent_coupons`
8.  `recent_emails_opened`
9.  `active_months`
10. `total_transactions`

Additional SHAP outputs are available in:

-   `outputs/shap_feature_importance.csv`
-   `outputs/top_20_shap_features.csv`

Feature importance indicates predictive association, not causality.

## 11. Customer Segmentation

The project compared churn across membership tier, city, and signup
cohort.

In the final feature population:

  Tier         Customers   Churned   Churn rate
  ---------- ----------- --------- ------------
  Gold               697       195       27.98%
  Platinum           260        77       29.62%
  Silver           1,268       390       30.76%

The most recent signup cohort had lower observed churn than older
cohorts, but newer customers also have shorter tenure, so this
comparison requires caution.

## 12. Ablation Study

The ablation experiment compared demographics/profile variables against
behavioural, engagement, and support features.

At the selected 0.70 threshold:

  ---------------------------------------------------------------------------
  Feature set          Precision         Recall             F1         PR-AUC
  --------------- -------------- -------------- -------------- --------------
  Demographics            0.0000         0.0000         0.0000         0.0965
  only                                                         

  Behavioural +           0.8846         0.4894         0.6301         0.6619
  Engagement +                                                 
  Support                                                      
  ---------------------------------------------------------------------------

This supports the conclusion that behavioural, engagement, and support
signals provide substantially more useful predictive information than
demographics alone in this modelling setup.

The result does not mean demographics are completely unrelated to churn;
the demographic-only model simply produced no positive predictions at
the selected threshold and had much lower PR-AUC.

Detailed results are in `outputs/ablation_results.csv`.

## 13. Churn Prediction Outputs

Customer-level predictions are available in:

-   `outputs/churn_predictions.csv`
-   `outputs/churn_predictions_validation.csv`
-   `outputs/churn_predictions_with_reasons.csv`

These contain customer identifiers, churn probabilities, predicted
labels, and available driver/reason information.

## 14. Retention Scenario Simulation

A retention scenario was created for customers with predicted churn
probability of at least 0.70.

The simulation assumed that a targeted intervention could reduce each
high-risk customer's predicted risk by **20% relative to baseline**.

  Metric                                                         Result
  ------------------------------------------- -------------------------
  High-risk customers                                                26
  Average churn probability before                               88.09%
  Average churn probability after                                70.47%
  Average risk reduction                        17.62 percentage points
  Estimated customers moved below 0.70 risk                          13

Customer-level results are in
`outputs/retention_simulation_customers.csv`.

Additional outputs:

-   `outputs/retention_offer_simulation.csv`
-   `outputs/retention_risk_summary.csv`

**Important:** this is a scenario analysis, not causal evidence. The 20%
reduction is an assumption. Actual intervention effectiveness must be
measured through a controlled campaign or experiment.

## 15. Business Recommendations

1.  Prioritise recent behavioural deterioration when identifying
    customers for retention.
2.  Use app and email engagement as early-warning indicators.
3.  Monitor support-ticket activity and consider proactive outreach for
    customers with elevated support activity.
4.  Target retention offers toward high-probability churn customers
    rather than sending offers indiscriminately.
5.  Use different retention strategies by risk level.
6.  Select the operating threshold based on campaign economics and the
    relative cost of false positives and missed churners.
7.  Measure actual intervention outcomes experimentally so simulated
    risk reduction can eventually be replaced with observed treatment
    effectiveness.

## 16. Limitations and Assumptions

-   The validation cohort has a much lower churn rate than the training
    cohort.
-   Missing customer-month records exist and activity completeness
    should be reviewed with the source-system owner.
-   Negative-spend removal and duplicate resolution are modelling
    assumptions.
-   Median age imputation and the `Unknown` gender category may affect
    results.
-   The all-missing home-store-price-tier variable was removed.
-   Feature importance indicates predictive association, not causality.
-   The retention analysis is hypothetical and does not prove that an
    offer will reduce churn.
-   Metrics should be interpreted alongside business costs of false
    positives and false negatives.
-   The model should be retrained and monitored when new behaviour and
    confirmed churn outcomes become available.

## 17. Reproducibility

Run the notebooks in this order:

``` text
01_data_quality.ipynb
02_eda.ipynb
03_feature_engineering.ipynb
04_validation_setup.ipynb
05_models.ipynb
06_explainability.ipynb
07_ablation.ipynb
08_retention_simulation.ipynb
```

The raw workbook is expected at:

``` text
data/raw/FreshBasket_Loyalty_Churn_Dataset.xlsx
```

The notebooks generate cleaned datasets under `data/processed/` and
analysis/model outputs under `outputs/`.

## 18. Key Deliverables

The project provides:

-   reproducible Python notebooks,
-   cleaned customer-level modelling data,
-   data-quality and EDA outputs,
-   model comparison results,
-   churn probability predictions,
-   feature-importance and SHAP outputs,
-   customer-level churn reasons where available,
-   ablation results,
-   retention scenario analysis, and
-   this README documenting assumptions, execution, limitations, and
    recommendations.

## 19. Conclusion

This project demonstrates a complete FreshBasket churn analytics
workflow, from data-quality assessment and EDA through leakage-aware
feature engineering, cohort validation, model comparison,
explainability, ablation testing, and retention scenario analysis.

The central finding is that **recent changes in customer behaviour and
engagement provide substantially more useful predictive information than
demographics alone** in this modelling setup. This supports a targeted
retention strategy in which FreshBasket prioritises customers showing
elevated behavioural risk rather than treating the entire loyalty base
equally.

For production use, the model should next be validated on future time
periods and the actual effectiveness and ROI of targeted retention
interventions should be measured.
