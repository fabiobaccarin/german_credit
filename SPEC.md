# Credit scoring: Production Model & Data Pipeline Specification

**Spec Version:** 0.0.1  

**Status:** Draft  

**Authors:** Fabio Baccarin  

**Stakeholders:**
  - Product Manager
  - Domain Expert
  - Engineering Lead

**Target Execution Date:** 2026-08-31  

## 1. Executive Summary & Problem Framing

### 1.1 Business Objective & Impact Metric
**Problem Description:** Create a standardized credit-risk scale that will
enable the bank to assess each customer's credit risk and then decide whether
the credit disbursement is within the bank's risk appettite.

**Primary Business Metric:** Average risk-adjusted margin lift against the
benchmark model.

**Target Variable Definition ($Y$):**
  - **Business Definition:** We are predicting whether the customer defaulted
    on their loan
  - **Mathematical Formulation:** $Y \in \{0, 1\}$ where $1 = \text{Default}$

**SLA & Compute Budget Constraints:**
  - **Inference Latency Target:** $< 50\text{ms}$ (p99) for batch / real-time
    REST endpoint
  - **Training SLA:** Max run time $< 4\text{ hours}$ on standard compute node

## 2. Data Contract & Cleaning Specification

### 2.1 Input Feature Expectations & Schema
**Name:** `residence_since`
**Description:** Duration in months
**Type:** Numeric

### 2.2 Data Quality Invariants & Outlier Rules
**Categorical Drift:** During inference, any category code outside the approved 
schema must automatically map to a `missing` token.

**Numeric Outlier Handling:**
  - Lower bound: $\max(x, \text{quantile}(0.005))$
  - Upper bound: $\min(x, \text{quantile}(0.995))$

**String Normalization Rules:** Trim whitespace, convert to lowercase, strip 
non-ASCII characters.

### 2.3 Anti-Leakage & Data Boundary Guarantees
**Temporal Cutoff Rule:** All features must be engineered using data available 
**strictly prior to** $T_{\text{event}}$.

**Explicit Feature Exclusions:** Banned columns (target proxies, future-looking 
state or ethically dubious):
  - `sex`

## 3. Validation Strategy Specification

### 3.1 Splitting Methodology
**Splitting Scheme:** Stratified K-Fold

**Rationale:** Stratified splitting is the gold standard for classification
tasks. It induces variance reduction between validation folds, thus ensuring
stable estimates of performance with a relatively low number of iterations. 

**Temporal Cutoff Date (if applicable):** The dataset does not have this here,
but is very important to do. It is important to make a temporal cut to be able
to prove that your credit score is able to assess credit risk for future
customers as well as it does for current ones.

### 3.2 Metric Acceptance Matrix
**Primary optimization:** Average risk-adjusted margin from credit disbursements

**Technical benchmark:** PR-AUC

**Fairness:**

## 4. Baseline & Candidate Modeling Specification

### 4.1 Baseline Benchmarks (Minimum Floor)
**Heuristic Baseline:** Predict historical majority class probability
$P(Y=1) = \bar{y}_{\text{train}}$

**Minimal Viable Model (MVM):**
  - Ridge Regression / Logistic Regression with Standard Scaler and One-Hot 
    Encoding
  - **Rule:** Candidate models must beat MVM by at least
    **15% relative improvement** on the primary optimization metric

### 4.2 Candidate Architectures & Search Space
**Algorithms under test:** `LightGBM`, `XGBoost`, `CatBoost`, `LogisticRegression` / `Ridge`

**Hyperparameter Tuning Budget:**

**Failure / Rejection Criteria:**

## 5. Definition of Done & Checklist
- [ ] Data cleaning spec enforced via automated Pandera schema validation.
- [ ] Zero leakage confirmed through temporal boundary Pytest suites.
- [ ] Baseline model executed and logged in MLflow experiment tracking.
- [ ] Champion model surpasses MVM threshold and satisfies all operational 
      guardrails.
- [ ] Pipeline serialized and verified against inference latency SLA.