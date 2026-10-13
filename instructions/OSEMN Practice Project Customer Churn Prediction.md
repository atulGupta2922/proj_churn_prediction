# OSEMN Practice Project: Customer Churn Prediction

Oct 12, 2026 · @Atul Gupta

A 2–3 day practice project: take a raw, messy marketing/customer dataset from Kaggle, clean it, build a churn classifier, and turn the result into a business report with recommended actions. Obtain is skipped, so the work runs Scrub → Explore → Model → iNterpret.

## Project setup

**Business question:** which customers are likely to stop buying or cancel in the next period, and what should marketing do about it? Target = a binary churn flag; success = a model that beats a naive baseline and produces a ranked list of at-risk customers.

Pick one dataset that is genuinely raw. These are common Kaggle choices from memory, so check the current listing and file version before committing.

| Dataset (search on Kaggle) | Messiness | Churn label | Best for practice |
| --- | --- | --- | --- |
| Online Retail / Online Retail II (UCI transactions) | High: missing CustomerID, cancellations, negative quantities, duplicates, odd stock codes | You build it (e.g. no purchase in the last 90 days) | Heavy cleaning plus feature engineering (RFM) |
| Telco Customer Churn (IBM sample) | Low–medium: TotalCharges stored as text with blanks, inconsistent categories | Provided (Churn) | Quickest end-to-end run |
| Customer Personality Analysis (marketing campaign data) | Medium: missing income, date formats, outliers, redundant columns | Campaign response (Response) | Campaign-targeting angle |

Decision: Online Retail II (UCI). It gives maximum cleaning practice, and you define churn yourself from transaction history (cutoff date, then no purchase in the next 90 days).

**Setup checklist**

1. Python environment with pandas, numpy, scikit-learn, matplotlib/seaborn, plus Jupyter or VS Code.
2. Git repo with folders: `data/raw`, `data/clean`, `notebooks`, `src`, `reports`.
3. Never edit the raw file; every cleaning step is code that reads raw and writes clean.
4. A `data_quality_log.md` where each issue found and fix applied is recorded.

## Scrub (clean the data)

Goal: a clean, documented table where one row = one customer, ready for modeling. Do the steps in this order and log every decision.

1. **Profile first.** Shape, dtypes, `df.info()`, `describe()`, null counts per column, duplicate counts. Write down what looks wrong before changing anything.
2. **Fix types.** Dates parsed to datetime, numbers stored as text converted (watch blank strings), IDs kept as strings, categories set as `category`.
3. **Standardize text.** Trim whitespace, unify case, merge variants ("N/A", "na", "" → real nulls; "No internet service" vs "No").
4. **Handle duplicates.** Exact duplicates vs repeated records that are legitimate; decide per case.
5. **Handle missing values.** For each column decide: drop, impute (median/mode), or add a missing-indicator flag. Record why, and check missingness by churn group before imputing.
6. **Treat outliers and invalid values.** Negative quantities, impossible ages, extreme spend. Decide: cap, remove, or keep as real signal (e.g. bulk buyers).
7. **Build the target.** If the label is not provided, define churn from a cutoff date (e.g. no purchase in the 90 days after the cutoff) using only data before the cutoff for features.
8. **Aggregate to customer level** if the data is transactional: recency, frequency, monetary value, average basket, tenure, return rate.
9. **Validate and save.** Assertions (no nulls in key columns, unique customer IDs, valid ranges), then write `data/clean/customers.csv`.

**Deliverable:** cleaning notebook or script plus a data-quality log table with columns: issue, where found, how many rows, fix applied, reason.

**Online Retail II specifics to expect:**

- Missing Customer ID on a large share of rows (guest checkouts). They cannot be used for customer-level churn, so drop them for modeling but note the share lost.
- Cancellations: invoices starting with "C" carry negative quantities. Treat them as returns, not purchases, and build a return-rate feature.
- Non-product stock codes (POST, D, M, BANK CHARGES and similar) that are not real sales.
- Zero or negative prices and quantities, and possible duplicate rows where the two Excel sheets overlap. Verify this on your copy.
- Inconsistent country values (EIRE, Unspecified); group rare countries.
- Churn target: set a cutoff 90 days before the last invoice date, keep customers active in the 180 days before it, and label churn = no purchase after the cutoff. Build features only from rows before the cutoff.
- Seasonality: the label window covers the busy autumn season, so mention it as a caveat in the report.

## Explore (EDA and features)

Goal: understand what separates churners from stayers, and turn that into features. Write each finding as a one-sentence insight with the chart that shows it.

**Questions to answer**

1. What is the churn rate overall, and is the data imbalanced enough to affect modeling?
2. How do tenure, recency, frequency and spend differ between churners and non-churners?
3. Which categorical segments (plan, region, channel, product category) churn most?
4. Which numeric features are correlated with each other (multicollinearity) or with the target?
5. Are there time patterns (seasonality, cohort retention curves)?

**Charts to practice:** target distribution, histograms and boxplots split by churn, grouped bar charts of churn rate by segment, correlation heatmap, cohort retention heatmap (if transactional).

**Feature engineering ideas:** RFM scores, tenure buckets, spend trend (last 30 days vs prior 30), average days between purchases, share of discounted purchases, return rate, contract or plan type encodings.

**Watch for leakage:** any feature computed using data after the churn cutoff, or a column that is effectively the label (e.g. a cancellation date), must be removed.

**Deliverable:** EDA notebook with 5–8 labeled insights and a final feature list with a one-line rationale per feature.

## Model

Goal: a validated classifier that beats a baseline, with a threshold chosen for a business reason.

1. **Split correctly.** Stratified train/test (e.g. 80/20). For time-based churn, split by time instead of randomly. Fit all preprocessing inside a scikit-learn `Pipeline` so nothing leaks from test data.
2. **Baseline.** Predict the majority class, then a simple logistic regression. Every later model must beat these.
3. **Candidates.** Logistic regression (interpretable), random forest, gradient boosting (XGBoost or LightGBM). Compare with 5-fold stratified cross-validation.
4. **Handle imbalance.** Try class weights first; try SMOTE only inside the cross-validation pipeline.
5. **Tune.** Randomized or grid search on the best one or two models; keep the search small.
6. **Pick metrics that fit the problem.** Accuracy misleads on imbalanced churn, so use these:

| Metric | Why it matters here |
| --- | --- |
| Recall (churners) | How many at-risk customers you catch |
| Precision | How much retention budget is wasted on stayers |
| F1 / PR-AUC | Balance of the two when churn is rare |
| ROC-AUC | Overall ranking quality |
| Calibration | Whether a 30% score really means about 30% |

7. **Choose a threshold** from a simple cost assumption (e.g. offer cost vs customer value) rather than defaulting to 0.5.
8. **Final check** on the untouched test set, once.

**Deliverable:** modeling notebook, a model comparison table, confusion matrix, PR curve, and the saved model (`joblib`) with its pipeline.

## iNterpret (use it in real life)

Goal: turn model output into decisions a marketing manager could act on, in a short report.

1. **Explain the model.** Feature importance and SHAP (or logistic coefficients) to show the top churn drivers; one or two example customers explained individually.
2. **Segment the at-risk list.** Group high-risk customers by value and by driver (e.g. high-value and recently inactive vs low-value and price-sensitive).
3. **Recommend actions per segment.** Win-back offer, onboarding fix, loyalty perk, or no action; state the assumed cost and expected effect.
4. **Estimate impact.** Simple arithmetic: customers flagged × expected save rate × customer value − offer cost. Label every assumption.
5. **Define how to test it.** An A/B test plan: treatment vs control among flagged customers, success metric, sample size, duration.
6. **State limits.** Data period, what the model cannot know, fairness or bias checks across segments, and when to retrain.

**Report outline (3–5 pages):** problem and decision, data and cleaning summary, key findings, model and performance in plain language, recommended actions with estimated impact, risks, next steps.

**Stretch goal:** wrap the saved model in a small Streamlit app that scores a customer from form inputs, and a script that scores a CSV of new customers into a ranked list.

## Schedule

Three working days, about 6.5 focused hours each, compressed from the original ten-day plan. Cleaning keeps the largest single block. If time runs short, cut in this order: the stretch app, hyperparameter tuning, extra candidate models. Never cut the cleaning log.

| Day | Time | Phase | Work | Output |
| --- | --- | --- | --- | --- |
| 1 | 1 h | Setup | Choose dataset, create repo, define business question and churn rule | README with problem statement |
| 1 | 5.5 h | Scrub | Profile, fix types, standardize text, duplicates, missing values, outliers, build target, aggregate to customer level, validate | Clean customer table, data-quality log |
| 2 | 1.5 h | Explore | Segment analysis, correlations, cohorts | 5–8 labeled insights with charts |
| 2 | 1 h | Explore | Feature engineering, leakage check, modeling plan | Final feature list |
| 2 | 4 h | Model | Split, baseline, cross-validated candidates | Model comparison table |
| 3 | 1.5 h | Model | Tuning, imbalance handling, threshold, one test-set check | Saved model and metric plots |
| 3 | 3 h | iNterpret | SHAP or feature importance, segments, impact estimate, report | Business report |
| 3 | 2 h | iNterpret (stretch) | Scoring script or Streamlit app, A/B test plan | Working scorer and pilot plan |

## Open questions

Answers to these will tighten the plan.

- [x] Which dataset do you want: Online Retail II (heavier cleaning, you define churn) or Telco (label provided)?
- [x] **~~Python~~** or R, and notebooks or scripts?
- [ ] Do you want the Streamlit app as part of the scope, or only the business report?
- [ ] Who is the audience for the report (portfolio, class, interview)? This sets the tone and length.
- [ ] Do you want code starter templates (cleaning checklist notebook, modeling pipeline) added as a second tab?
