# ML Workflow Assignment

## Task 1 — Label and Data Leakage

**Label column:** `repeat_purchase_flag`
This column directly represents the outcome we are trying to predict — whether a customer made a repeat purchase within 30 days — making it the target variable for the model.

**Data leakage column:** `discount_used_on_repeat_order`
This column reveals information that only exists *after* the repeat purchase has already happened, so including it as a feature would leak future information into training and make the model useless in real-world prediction.

---

## Task 2 — Missing ML Workflow Steps

**Step 1 — Exploratory Data Analysis (EDA)**
Before training any model, EDA should be done to understand the distribution of each feature, check for missing values, spot outliers, and examine class imbalance in `repeat_purchase_flag` — skipping this means the model may be trained on dirty or misleading data without anyone realizing it.

**Step 2 — Feature Engineering and Data Preprocessing**
Raw columns like `days_since_last_order` and `avg_order_value` may need scaling, transformation, or derived features before being fed into any model — jumping straight to gradient boosting without this risks poor model performance and makes it impossible to fairly compare against simpler baselines.
