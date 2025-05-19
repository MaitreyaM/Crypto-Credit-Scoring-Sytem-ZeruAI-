# Compound V2 Wallet Credit Scoring System

**Developed for Zeru Finance by Maitreya Mishra**
**Date:** May 10, 2025

## 1. Project Overview

This project implements an AI-powered, decentralized credit scoring system for wallets interacting with the Compound V2 protocol. The primary objective is to assign a credit score between 0 and 100 to each wallet based *solely* on its historical on-chain transaction behavior. Higher scores are indicative of reliable and responsible protocol usage, while lower scores reflect risky, potentially bot-like, or exploitative behavior.

This was an end-to-end modeling task undertaken without predefined labels, requiring independent criteria definition for "good" and "bad" wallet behavior, extensive feature engineering from raw transaction logs, and the design of a custom credit scoring system.

## 2. Methodology

The approach is unsupervised and feature-driven, culminating in a weighted rule-based scoring mechanism. Key methodological steps include:

### 2.1. Data Ingestion and Preprocessing
*   **Data Source:** Raw JSON transaction-level data from Compound V2 (deposits, borrows, repays, withdraws, liquidations).
*   **Selection:** Utilized the three largest data chunks to ensure a representative dataset.
*   **Processing:**
    *   Parsing and consolidation of transaction types into Pandas DataFrames.
    *   Flattening of nested JSON structures.
    *   Type conversion (numeric, datetime).
    *   Creation of a unified transaction log for common actions.

### 2.2. Feature Engineering
A comprehensive set of over 40 wallet-level features was engineered to capture diverse behavioral patterns:
*   **Activity & Profile:** `total_transactions`, `wallet_age_days`, `active_days`, `avg_txns_per_active_day`, recency metrics like `days_since_last_activity`, etc.
*   **Transaction Aggregates:** Counts, total USD values, min/max/avg USD values for deposits, borrows, repays, withdraws.
*   **Risk & Repayment Ratios:** `repay_to_borrow_ratio_usd`, `borrow_to_deposit_ratio_usd`, `withdraw_to_deposit_ratio_usd`.
*   **Net Value Metrics:** `net_deposit_usd`, `net_borrow_activity_usd`.
*   **Liquidation History:** `times_liquidates_count` (as liquidatee), `total_usd_liquidated_as_liquidatee`.
*   **Behavioral Flags:** `percentage_small_usd_txns` (potential bot activity).

### 2.3. Data Treatment
*   **Missing Values:** Handled by filling with 0 (for counts/sums) or dataset-age based values (for "days since" features where an action never occurred).
*   **Outlier Capping:** Key financial ratios were capped at their 1st and 99th percentiles to mitigate the impact of extreme values before scaling.
*   **Feature Scaling:** All numeric features were standardized using `StandardScaler` to have a mean of ~0 and a standard deviation of ~1.

### 2.4. Scoring Logic: Weighted Rule-Based System
*   A custom set of weights was defined for selected scaled features. Positive weights reward responsible behavior (e.g., high repayment ratio, older wallet age), while negative weights penalize risky behavior (e.g., high liquidation count, high leverage).
    *   *(Example of key weighted features: `times_liquidates_count` with a weight of -2.5, `repay_to_borrow_ratio_usd` with +2.0, etc. - **Maitreya, you should list a few of your key final weights here as examples**)*
*   A `raw_score` was calculated for each wallet by summing the (scaled feature value * weight).
*   These `raw_scores` were then scaled to a final 0-100 range using `MinMaxScaler`.

### 2.5. (Optional) Exploratory Clustering
*   Initial K-Means clustering (K=3) was performed on an earlier version of scaled features to gain insights into natural behavioral segments, which helped inform feature importance and the definition of risk criteria.

## 3. Outputs & Deliverables

*   **Methodology Document:** A detailed explanation of the scoring logic and rationale (this README serves as a summary; a more detailed version is also available).
*   **Code Submission:** The Jupyter Notebook (`ZeruAI.ipynb`) containing all data processing, feature engineering, and scoring logic.
*   **CSV Output:** A file (`top_1000_wallet_scores_final.csv`) containing the wallet addresses and their assigned credit scores (0-100, rounded to whole numbers) for the top 1,000 wallets, sorted by score.
*   **Wallet Analysis:** A one-page document analyzing five high-scoring and five low-scoring wallets, explaining the observed patterns and justifying their scores based on the engineered features and scoring model.

## 4. Results & Interpretation

The model successfully assigns credit scores between 0 and 100. The final score distribution based on the [mention the data files used, e.g., "three largest chunks"] shows:
*   **Mean Score:** ~58.05
*   **Standard Deviation:** ~1.41
*   **Score Range:** 0 to 100

This indicates that while the model clearly differentiates wallets at the extremes of risk and reliability, a significant majority of wallets in this dataset exhibit a relatively similar "average" profile according to the defined features and weighting scheme, leading to a concentration of scores around the mean. The wallet analysis document provides specific examples of these profiles.

*(**Maitreya, you can add a sentence or two here about any key insights from your wallet analysis if you wish, e.g., "High-scoring wallets typically showed consistent repayment and no liquidations, while low-scoring wallets often had multiple liquidations and poor repayment histories."**)*

## 5. How to Run

1.  **Environment:** Python 3.x with libraries: pandas, numpy, scikit-learn, matplotlib, glob.
2.  **Data:** Place the Compound V2 transaction JSON files (e.g., `compoundV2_transactions_ethereum_chunk_0.json`, etc.) in the same directory as the notebook or update file paths accordingly.
3.  **Execution:** Run the cells in the `ZeruAI.ipynb` Jupyter Notebook sequentially.
4.  **Output:** The script will generate `top_1000_wallet_scores_final.csv`.

## 6. Potential Future Enhancements

*   **Weight Optimization:** Explore techniques to optimize feature weights if labeled data or more direct business feedback becomes available.
*   **Non-Linear Feature Transformations:** Investigate non-linear transformations for certain features (e.g., log transforms for monetary values, binning for counts) to potentially improve score discrimination.
*   **Advanced Modeling:** If labels were available, supervised learning models could be trained. For unsupervised improvements, more sophisticated clustering or anomaly detection techniques could be explored.
*   **Dynamic Time Windowing:** Analyze wallet behavior over different time windows to capture evolving risk profiles.
*   **Incorporating External Data:** If permissible, enrich with other on-chain or off-chain data sources.

---
