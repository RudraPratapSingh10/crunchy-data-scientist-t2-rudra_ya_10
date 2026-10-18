# Model Risk Report: Customer Churn Prediction

## 1. Executive Summary

This project evaluates a class-weighted Logistic Regression model for customer churn prediction using the Telco Customer Churn dataset.

The dataset contains 7,043 customers: 5,174 non-churners (73.46%) and 1,869 churners (26.54%). The target classes are imbalanced, but this is not a 95/5 split.

The data was divided using a stratified train/test split, with 1,409 customers in the held-out test set. Threshold selection was explored separately using a validation set.

At the default classification threshold of 0.5, the model achieved the following test-set results:

- Precision: 0.5034
- Recall: 0.7834
- F1-score: 0.6130
- PR-AUC (Average Precision): 0.6324
- ROC-AUC: 0.8414
- Balanced accuracy: 0.7521

Among the thresholds tested on the validation set (0.3, 0.4, 0.5, 0.6, and 0.7), a threshold of 0.6 achieved the highest validation F1-score (0.6338), narrowly above the score at 0.5 (0.6331).

The selected validation threshold of 0.6 was then evaluated on the held-out test set. It achieved precision of 0.5388, recall of 0.7059, F1-score of 0.6111, and balanced accuracy of 0.7438. Compared with threshold 0.5, it produced higher precision but lower recall and a slightly lower F1-score on the test set. This difference illustrates why validation results may not exactly match test results.

Calibration analysis on the test set suggested that predicted churn probabilities may be higher than observed churn rates in some plotted bins. Subgroup analysis at threshold 0.5 also revealed differences in performance across contract types.

Accuracy is not treated as the sole measure of model quality. The findings highlight threshold trade-offs, probability-calibration risks, and the need to investigate subgroup performance before deployment.

## 2. Confusion Matrix and Accuracy Risk

This section reports the model's results at the default classification threshold of 0.5 on the held-out test set of 1,409 customers.

### Confusion Matrix at Threshold 0.5

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

Therefore, accuracy alone does not demonstrate that a churn model is useful. Precision, recall, F1-score, PR-AUC, ROC-AUC, balanced accuracy, and the confusion matrix provide additional information about performance.

## 3. Evaluation Metrics and Business Trade-offs

The following metrics describe the test-set performance at the default threshold of 0.5.

| Metric | Score | Interpretation |
|---|---:|---|
| Precision | 0.5034 | About 50.34% of customers predicted to churn actually churned. |
| Recall | 0.7834 | The model identified about 78.34% of actual churners. |
| F1-score | 0.6130 | Balances precision and recall. |
| PR-AUC (Average Precision) | 0.6324 | Summarizes precision-recall performance across probability thresholds. |
| ROC-AUC | 0.8414 | Measures how well the model ranks churners above non-churners. |
| Balanced accuracy | 0.7521 | Averages recall for both classes. |

### Business Trade-offs

False positives occur when customers are predicted to churn but would stay. Contacting these customers may waste retention resources or lead to unnecessary discounts.

False negatives occur when actual churners are predicted not to churn. The business may lose these customers because they do not receive timely retention offers.

If retaining customers is the priority, higher recall may be desirable. However, higher recall can also increase false positives and retention costs.

### Recommended Optimization Metric

PR-AUC is useful because the dataset has fewer churners than non-churners. Recall should also be monitored because missing actual churners can lead to lost revenue.

The final decision threshold should depend on the cost of false positives, the cost of false negatives, and the available retention budget. It should be selected using validation data rather than repeatedly optimized against the test set.

## 4. Threshold Tuning and Business Impact

The classification threshold determines the probability at which a customer is classified as likely to churn.

A lower threshold generally identifies more potential churners, increasing recall but potentially generating more false positives. A higher threshold generally reduces false positives but may miss more actual churners.

### Validation Threshold Comparison

Threshold selection was explored using a validation set, rather than using the test set for tuning.

| Threshold | Precision | Recall | F1-score |
|---|---:|---:|---:|
| 0.3 | 0.4222 | 0.9164 | 0.5781 |
| 0.4 | 0.4643 | 0.8696 | 0.6054 |
| 0.5 | 0.5241 | 0.7993 | 0.6331 |
| 0.6 | 0.5730 | 0.7090 | 0.6338 |
| 0.7 | 0.6568 | 0.5953 | 0.6246 |

Among the tested thresholds, 0.6 achieved the highest validation F1-score (0.6338), narrowly outperforming 0.5 (0.6331).

### Final Test Evaluation at Threshold 0.6

After selecting 0.6 based on the validation results, the threshold was evaluated on the held-out test set.

| Metric | Test score at threshold 0.6 |
|---|---:|
| Precision | 0.5388 |
| Recall | 0.7059 |
| F1-score | 0.6111 |
| Balanced accuracy | 0.7438 |
| PR-AUC (Average Precision) | 0.6324 |
| ROC-AUC | 0.8414 |

Confusion matrix at threshold 0.6:

| Actual / Predicted | Predicted No Churn | Predicted Churn |
|---|---:|---:|
| Actual No Churn | 809 | 226 |
| Actual Churn | 110 | 264 |

At threshold 0.6, precision was higher than at 0.5, but recall was lower. The test F1-score was 0.6111, slightly below the 0.6130 obtained at 0.5. The threshold was selected using validation data; the test result is reported as a final evaluation, not as a reason to retune the threshold on the test set.

### Business Recommendation

- **If missing a potential churner is expensive:** A lower threshold may be considered to increase recall.
- **If retention campaigns are expensive:** A higher threshold may be considered to reduce false positives.
- **For production use:** Choose the threshold using validation data and business-cost estimates, then evaluate it once on an untouched test set.

The best threshold depends on business costs and operational constraints. The highest validation F1-score is a useful comparison criterion, but it does not automatically mean that threshold is best for every business objective.

## 5. Probability Calibration and Imbalance Handling

### Probability Calibration

A calibration curve was created using the final model's predicted churn probabilities on the test set and the corresponding actual churn outcomes. The curve is a visual diagnostic comparing predicted probabilities with observed churn rates.

The visual analysis suggested that predicted churn probabilities were generally higher than observed churn rates across some plotted bins. This suggests that the model may overestimate churn probabilities in some parts of the probability range. This is a visual interpretation, not proof that every probability is overestimated.

A model can rank customers effectively while still producing probabilities that do not accurately represent their actual likelihood of churning. Therefore, ROC-AUC alone is not sufficient to establish that predicted probabilities are reliable.

Before using these probabilities for business decisions, calibration should be assessed using separate validation data. Techniques such as sigmoid calibration or isotonic regression may be considered if further testing confirms a calibration problem.

### Handling Class Imbalance

The dataset contains 5,174 non-churning customers (73.46%) and 1,869 churning customers (26.54%). Since the classes are imbalanced, the model uses class-weighted Logistic Regression:

`LogisticRegression(max_iter=2000, class_weight="balanced", random_state=42)`

Class weighting assigns greater importance to the minority class during model training. It does not change the original class distribution.

The model's performance is assessed using precision, recall, F1-score, PR-AUC, balanced accuracy, and the confusion matrix rather than relying only on accuracy.

### Limitations

- The calibration curve is a visual diagnostic and does not prove that every predicted probability is overestimated.
- Class weighting may improve minority-class detection but does not guarantee better probability calibration.
- Calibration methods and class-imbalance techniques should be compared using validation data before deployment.

## 6. Subgroup Performance Analysis

The model was evaluated across three customer attributes: contract type, senior-citizen status, and gender. The following subgroup metrics were calculated at the default threshold of 0.5, so they should not be interpreted as subgroup metrics at the selected threshold of 0.6.

### 6.1 Contract Type

| Contract Type | Customers | Actual Churners | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Month-to-month | 773 | 329 | 0.5144 | 0.8663 | 0.6455 |
| One year | 300 | 36 | 0.2857 | 0.2222 | 0.2500 |
| Two year | 336 | 9 | 0.0000 | 0.0000 | 0.0000 |

**Observation:** The model performed best on month-to-month customers in terms of F1-score. It failed to identify churners in the two-year contract group at threshold 0.5. However, this group contained only nine actual churners, so the result should be interpreted cautiously.

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

At the default threshold of 0.5, the class-weighted Logistic Regression model achieved recall of 0.7834, F1-score of 0.6130, PR-AUC of 0.6324, and ROC-AUC of 0.8414. Its precision of 0.5034 indicates that many customers predicted to churn did not actually churn.

A threshold of 0.6 achieved the highest F1-score among the tested validation thresholds. When evaluated on the held-out test set, it achieved precision of 0.5388, recall of 0.7059, and F1-score of 0.6111. Compared with threshold 0.5, it favored higher precision over recall and had a slightly lower test F1-score. This illustrates the practical trade-off between identifying more potential churners and avoiding unnecessary retention campaigns.

Calibration analysis on the test set raised concerns that predicted churn probabilities may be higher than observed churn rates in some bins. Subgroup analysis at threshold 0.5 revealed particularly weak churn detection for customers with one-year and two-year contracts.

Before deployment, the model should undergo further validation, including threshold selection based on business costs, calibration assessment on validation data, and continued monitoring of subgroup performance.

**Final recommendation:** Treat this model as a decision-support tool for further testing, not as a fully trusted production system. Business costs, probability reliability, and subgroup performance must be considered before using its predictions to make customer-retention decisions.
