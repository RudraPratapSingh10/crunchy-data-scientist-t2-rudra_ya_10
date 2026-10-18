# Customer Churn Prediction — Model Risk Analysis

## 📌 Project Overview

This project evaluates a machine learning classification model for predicting customer churn using the Telco Customer Churn dataset.

The main objective is to evaluate the model beyond accuracy by examining precision, recall, F1-score, ROC-AUC, PR-AUC, classification thresholds, probability calibration, and subgroup performance.

The project highlights why machine learning predictions should be evaluated carefully before being used for real business decisions.

## 🎯 Objectives

- Build a baseline model for comparison.
- Develop a Logistic Regression classification pipeline.
- Evaluate model performance using multiple metrics.
- Analyze the impact of classification thresholds.
- Investigate probability calibration risks.
- Compare performance across customer subgroups.
- Document model limitations and recommendations for responsible use.

## 📂 Project Structure

crunchy-data-scientist-t2-rudra_ya_10/
│
├── data/
│   └── raw/
│       └── WA_Fn-UseC_-Telco-Customer-Churn.csv
│
├── notebooks/
│   └── model_risk_analysis.ipynb
│
├── reports/
│   └── MODEL_RISK.md
│
└── README.md

## 🛠️ Tools and Technologies

- Python
- Pandas
- NumPy
- Matplotlib
- Scikit-learn
- Jupyter Notebook
- Git and GitHub

## 📊 Dataset

The dataset contains 7,043 customer records.

| Customer Category | Count | Percentage |
|---|---:|---:|
| No Churn | 5,174 | 73.46% |
| Churn | 1,869 | 26.54% |

## 🤖 Model Development

A Logistic Regression model was developed using a Scikit-learn Pipeline.

Key steps included:

1. Data cleaning and preparation.
2. Handling missing values.
3. Numerical feature scaling.
4. Categorical feature encoding.
5. Stratified train-test splitting.
6. Baseline model evaluation.
7. Logistic Regression model training.
8. Classification threshold comparison.
9. Model risk and subgroup analysis.

## 📈 Model Evaluation

The following results were obtained on the test set at the default threshold of 0.5.

| Metric | Score |
|---|---:|
| Precision | 0.5034 |
| Recall | 0.7834 |
| F1-score | 0.6130 |
| PR-AUC | 0.6324 |
| ROC-AUC | 0.8414 |
| Balanced Accuracy | 0.7521 |

### Threshold Analysis

A threshold of 0.6 achieved the highest validation F1-score among the tested thresholds.

On the held-out test set, threshold 0.6 achieved:

| Metric | Score |
|---|---:|
| Precision | 0.5388 |
| Recall | 0.7059 |
| F1-score | 0.6111 |
| Balanced Accuracy | 0.7438 |

## 🔍 Model Risk Findings

- **Accuracy Risk:** Accuracy alone can be misleading when the dataset contains more non-churning customers than churning customers.
- **Threshold Trade-offs:** Lower thresholds can increase recall but may generate more false positives.
- **Probability Calibration:** The calibration curve raised concerns that predicted probabilities may be higher than observed churn rates in some bins.
- **Subgroup Performance:** Churn detection was particularly weak for one-year and two-year contract customers at the default threshold.

## 💡 Recommendations

- Evaluate precision and recall alongside accuracy.
- Select classification thresholds using validation data and business costs.
- Assess probability calibration before relying on predicted probabilities.
- Investigate subgroup performance and error rates.
- Monitor the model using additional data before deployment.

## 📁 Project Files

- **Notebook:** `notebooks/model_risk_analysis.ipynb`
- **Detailed Report:** `reports/MODEL_RISK.md`

## 🏁 Conclusion

This project demonstrates the importance of evaluating machine learning models beyond accuracy.

Although the Logistic Regression model identifies many actual churners, its precision, threshold-dependent performance, calibration concerns, and subgroup differences require further investigation.

The model should be treated as a decision-support tool for further testing rather than a fully validated production system.
