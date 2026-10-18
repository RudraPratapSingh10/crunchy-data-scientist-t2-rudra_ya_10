# Model Risk Report: Customer Churn Prediction

## 1. Executive Summary

This project evaluates a Logistic Regression model for customer churn prediction using the Telco Customer Churn dataset.

The dataset contains 7,043 customers: 5,174 non-churners (73.46%) and 1,869 churners (26.54%). The target is imbalanced, but it is not a 95/5 split.

The model uses class-weighted Logistic Regression and was evaluated on a stratified held-out test set of 1,409 customers.

At the default classification threshold of 0.5, the model achieved:

- Precision: 0.5034
- Recall: 0.7834
- F1-score: 0.6130
- PR-AUC (Average Precision): 0.6324
- ROC-AUC: 0.8414
- Balanced accuracy: 0.7521

Threshold comparison showed that the highest F1-score among the tested thresholds (0.3, 0.4, 0.5, 0.6, and 0.7) was 0.6130 at threshold 0.5.

Calibration analysis indicated that predicted churn probabilities were generally higher than observed churn rates in the plotted bins. Subgroup analysis also revealed substantial performance differences across contract types.

Accuracy is not treated as the sole measure of model quality. The findings highlight threshold trade-offs, probability calibration risks, and the need to investigate subgroup performance before deployment.

## 2. Confusion Matrix and Accuracy Risk

The model was evaluated on 1,409 customers using a classification threshold of 0.5.

### Confusion Matrix

| Actual / Predicted | Predicted No Churn | Predicted Churn |
|---|---:|---:|
| Actual No Churn | 746 | 289 |
| Actual Churn | 81 | 293 |

### Interpretation

- **True Negatives (TN):** 746 customers were correctly predicted not to churn.
- **False Positives (FP):** 289 customers were predicted to churn but did not.
- **False Negatives (FN):** 81 customers were predicted not to churn but actually churned.
- **True Positives (TP):** 293 customers were correctly identified as churners.

### Why Accuracy Can Be Misleading

The dataset contains 73.46% non-churning customers and 26.54% churning customers. A model that predicts "No Churn" for every customer would achieve approximately 73.46% accuracy while failing to identify any churners.

Therefore, accuracy alone does not demonstrate that a churn model is useful. Precision, recall, F1-score, PR-AUC, and ROC-AUC provide additional information about its performance.

## 3. Evaluation Metrics and Business Trade-offs

The model was evaluated using multiple metrics to understand its ability to identify customers who may churn.

| Metric | Score | Interpretation |
|---|---:|---|
| Precision | 0.5034 | About 50.34% of customers predicted to churn actually churned. |
| Recall | 0.7834 | The model identified about 78.34% of actual churners. |
| F1-score | 0.6130 | Balances precision and recall. |
| PR-AUC | 0.6324 | Measures precision-recall performance across thresholds. |
| ROC-AUC | 0.8414 | Measures how well the model ranks churners above non-churners. |
| Balanced Accuracy | 0.7521 | Averages recall for both classes. |

### Business Trade-offs

False positives occur when customers are predicted to churn but would stay. Contacting these customers may waste retention resources or lead to unnecessary discounts.

False negatives occur when actual churners are predicted not to churn. The business may lose these customers because they do not receive timely retention offers.

If retaining customers is the priority, higher recall may be desirable. However, higher recall can also increase false positives and retention costs.

### Recommended Optimization Metric

PR-AUC is a useful metric because the dataset has fewer churners than non-churners. Recall should also be monitored because missing actual churners can lead to lost revenue.

The final decision threshold should depend on the cost of false positives, the cost of false negatives, and the available retention budget. It should be selected using validation data rather than repeatedly optimizing against the test set.

## 4. Threshold Tuning and Business Impact

The classification threshold determines the probability at which a customer is classified as likely to churn.

A lower threshold generally identifies more potential churners, increasing recall but potentially generating more false positives. A higher threshold generally reduces false positives but may miss more actual churners.

### Threshold Comparison

| Threshold | Precision | Recall | F1-score |
|---|---:|---:|---:|
| 0.3 | 0.4295 | 0.9278 | 0.5871 |
| 0.4 | 0.4655 | 0.8663 | 0.6056 |
| 0.5 | 0.5034 | 0.7834 | 0.6130 |
| 0.6 | 0.5388 | 0.7059 | 0.6111 |
| 0.7 | 0.6016 | 0.6016 | 0.6016 |

Among the tested thresholds, 0.5 achieved the highest F1-score of 0.6130.

### Business Recommendation

- **If missing a potential churner is expensive:** Consider a lower threshold to increase recall.
- **If retention campaigns are expensive:** Consider a higher threshold to reduce false positives.
- **For production use:** Select the threshold using validation data and business cost estimates, then evaluate the final model on an untouched test set.

**Important limitation:** The comparison above uses the held-out test set for exploration. These results should not be treated as an unbiased final threshold-selection procedure because the test labels were used to compare thresholds.

## 5. Probability Calibration and Imbalance Handling

### Probability Calibration

A calibration curve was created using the model's predicted churn probabilities and the actual churn outcomes.

The visual analysis indicates that predicted churn probabilities were generally higher than the observed churn rates across the plotted bins. This suggests that the model may overestimate churn probabilities for some groups of customers.

A model can rank customers effectively while still producing probabilities that do not accurately represent their actual likelihood of churning. Therefore, ROC-AUC alone is not sufficient to establish that predicted probabilities are reliable.

Before using these probabilities for business decisions, calibration should be evaluated on separate validation data. Techniques such as sigmoid calibration or isotonic regression may be considered if further testing confirms a calibration problem.

### Handling Class Imbalance

The dataset contains 5,174 non-churning customers and 1,869 churning customers. Since the classes are imbalanced, the model uses class-weighted Logistic Regression:

`LogisticRegression(max_iter=2000, class_weight="balanced", random_state=42)`

Class weighting gives greater importance to the minority class during model training without changing the original class distribution.

The model's performance should be assessed using precision, recall, F1-score, PR-AUC, and the confusion matrix rather than relying only on accuracy.

### Limitations

- The calibration curve is a visual diagnostic and does not prove that every predicted probability is overestimated.
- Class weighting may improve minority-class detection but does not guarantee better calibration.
- Calibration methods and class-imbalance techniques should be compared using validation data before deployment.

## 6. Subgroup Performance Analysis

The model was evaluated across three customer attributes: contract type, senior-citizen status, and gender. Precision, recall, and F1-score were compared at the default threshold of 0.5.

### 6.1 Contract Type

| Contract Type | Customers | Actual Churners | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Month-to-month | 773 | 329 | 0.5144 | 0.8663 | 0.6455 |
| One year | 300 | 36 | 0.2857 | 0.2222 | 0.2500 |
| Two year | 336 | 9 | 0.0000 | 0.0000 | 0.0000 |

**Observation:** The model performed best on month-to-month customers in terms of F1-score. It failed to identify churners in the two-year contract group at the selected threshold. However, this group contained only nine actual churners, so the result should be interpreted cautiously.

### 6.2 Senior-Citizen Status

| Group | Customers | Actual Churners | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Non-senior | 1,187 | 276 | 0.4709 | 0.7319 | 0.5730 |
| Senior | 222 | 98 | 0.5948 | 0.9286 | 0.7251 |

**Observation:** Recall was higher for senior citizens than for non-senior customers. This indicates that the model identified a larger proportion of actual churners in the senior group. Further validation is needed before drawing conclusions about differences in overall model reliability.

### 6.3 Gender

| Gender | Customers | Actual Churners | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Female | 687 | 193 | 0.5282 | 0.7772 | 0.6289 |
| Male | 722 | 181 | 0.4799 | 0.7901 | 0.5971 |

**Observation:** Recall was similar across female and male customers, while precision and F1-score differed slightly. These differences should be monitored rather than assumed to be statistically significant.

### Subgroup Analysis Limitations

- Subgroup performance can vary because of sample size and differences in customer behavior.
- The two-year contract group had only nine actual churners, making its recall estimate particularly sensitive to individual predictions.
- These comparisons identify areas for further investigation; they do not, by themselves, establish unfair bias.
- Before deployment, evaluate subgroup performance on additional data and investigate possible differences in error rates.

## 7. Conclusion

This project demonstrates why a classification model should not be trusted based on accuracy alone.

The class-weighted Logistic Regression model achieved a recall of 0.7834 and an F1-score of 0.6130 at the default threshold of 0.5. Its PR-AUC was 0.6324 and ROC-AUC was 0.8414. However, its precision of 0.5034 indicates that many customers predicted to churn did not actually churn.

Threshold analysis highlighted the trade-off between identifying more potential churners and avoiding unnecessary retention campaigns. Calibration analysis also raised concerns that predicted churn probabilities may be higher than observed churn rates. Subgroup analysis revealed particularly weak churn detection for customers with one-year and two-year contracts.

Before deployment, the model should undergo further validation, including threshold selection using validation data, calibration assessment, and continued monitoring of subgroup performance.

**Final recommendation:** Treat this model as a decision-support tool for further testing, not as a fully trusted production system. Business costs, probability reliability, and subgroup performance must be considered before using its predictions to make customer-retention decisions.