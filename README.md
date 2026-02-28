# Regime-Aware-Credit-PD-Pipeline

- Instructions to co-pilot

Good.
We’re building a lean, working prototype in 2–3 days.
No fluff. No overengineering.
Just: macro → HMM → regime → PD model → calibration → compare.
Below is copy-paste ready Copilot instructions, step-by-step.

🚀 PROJECT: Regime-Aware Credit PD Prototype
We will:
Load borrower + macro data
Fit HMM to macro variables
Infer regimes
Merge regime into borrower data
Train baseline PD model
Train regime-aware PD model
Calibrate probabilities
Compare results

📁 STEP 0 — Setup Environment
Copy this into Copilot:
Create a new Python project.

Install required libraries:
- pandas
- numpy
- scikit-learn
- hmmlearn
- xgboost
- matplotlib
- seaborn

Set up a Jupyter notebook called:
regime_pd_prototype.ipynb


📊 STEP 1 — Load Borrower Dataset
Use any public dataset (LendingClub / synthetic if needed).
Copilot instruction:
Load a credit dataset with:
- Numerical financial features
- A binary default column (1 = default, 0 = no default)
- A date column (monthly or yearly)

Ensure:
- No missing target values
- Date column converted to datetime
- Data sorted by date

If you don’t have macro-aligned data, create synthetic macro time series.

🌍 STEP 2 — Load Real Macro Data from World Bank
Copilot instruction:
Fetch macroeconomic time series from the World Bank API (no synthetic macro generation).

Use these indicators:
- GDP growth (annual %) → `NY.GDP.MKTP.KD.ZG`
- Unemployment, total (% of labor force) → `SL.UEM.TOTL.ZS`
- Lending interest rate (%) → `FR.INR.LEND`

Use API endpoint pattern:
`https://api.worldbank.org/v2/country/{country_code}/indicator/{indicator_code}?format=json&per_page=20000`

Steps:
1. Download each indicator as JSON
2. Parse year + value pairs into DataFrames
3. Convert dates to year-end datetime (`YYYY-12-31`)
4. Merge all indicators on date (inner join)
5. Align date range to borrower dataset before HMM fitting

Store macro data in DataFrame:
date, gdp_growth, unemployment, interest_rate

🧠 STEP 3 — Fit Hidden Markov Model
Copilot instruction:
Using hmmlearn, fit a GaussianHMM with 3 hidden states.

Steps:
1. Standardize macro variables
2. Fit HMM with n_components=3
3. Use model.predict() to infer hidden state per date
4. Add inferred regime as a new column in macro dataframe

Print:
- Transition matrix
- Average macro values per regime

Check regimes look different.

🔗 STEP 4 — Merge Regime with Borrower Data
Copilot instruction:
Merge borrower dataset with macro dataset on date.

For each borrower observation:
- Add corresponding macro regime label

Final dataset must contain:
- Financial features
- Regime column
- Default target


📉 STEP 5 — Train Baseline PD Model (No Regime)
Copilot instruction:
Train a baseline logistic regression model using:
- Only borrower financial features
- Exclude regime column

Split data into:
- Train: first 70% by date
- Test: last 30% by date

Evaluate:
- ROC AUC
- Confusion matrix
- Default rate vs predicted PD (basic calibration check)

Store AUC.

📈 STEP 6 — Train Regime-Aware PD Model
Option A (Faster)
Copilot instruction:
Train an XGBoost classifier using:
- Borrower financial features
- Add regime column as categorical feature

Use:
- max_depth=3
- learning_rate=0.05
- n_estimators=100

Evaluate:
- ROC AUC on test set
- Compare to baseline AUC


Option B (Stronger for Interviews)
Copilot instruction:
Split dataset by regime.

For each regime:
- Train separate XGBoost model using borrower features only

When predicting:
- Use regime of observation to choose correct model

Evaluate overall test AUC.
Compare with baseline.


📏 STEP 7 — Add Monotonic Constraints (Important)
Copilot instruction:
Re-train XGBoost with monotonic constraints.

Assume:
- Feature 1 (leverage) → monotonic increasing
- Feature 2 (interest coverage) → monotonic decreasing
- Others → no constraint

Use monotone_constraints parameter.

Evaluate AUC again.

This shows economic reasoning.

🎯 STEP 8 — Probability Calibration
Copilot instruction:
Use sklearn's CalibratedClassifierCV with method='isotonic'.

Calibrate XGBoost probabilities on training data.

Plot:
- Calibration curve
- Predicted PD vs actual default rate (bin into deciles)

Compare:
Before calibration
After calibration

📊 STEP 9 — Compare Results
Create final comparison table:
Model | AUC | Comments
Baseline Logistic | X.XX | No regime
XGBoost | X.XX | With regime
Calibrated XGBoost | X.XX | With regime + calibration


🧾 STEP 10 — Print Insights
Add summary cell:
Print:
- Regime transition matrix
- Default rate per regime
- AUC comparison
- Key features importance from XGBoost


🏁 DONE
You now have:
Working HMM
Regime inference
Regime-aware PD model
Calibration
Performance comparison
This is fully explainable in interviews.

🔥 If You Want Next
I can give you:
Exact minimal code template
Synthetic dataset generator
How to package this for GitHub
How to defend it in JPM/Nomura interviews
Common questions they will attack you with
Tell me your level of seriousness.


