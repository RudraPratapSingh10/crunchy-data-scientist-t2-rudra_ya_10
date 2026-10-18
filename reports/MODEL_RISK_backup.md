# Model Risk Report: Customer Churn Prediction

## 1. Executive Summary

This project evaluates a Logistic Regression model for customer churn
using the Telco Customer Churn dataset. The dataset contains 7,043
customers: 5,174 non-churners (73.46%) and 1,869 churners (26.54%). The
classes are imbalanced, but this is not a 95/5 split.

The project examines class imbalance, threshold selection, probability
calibration, and subgroup performance. Accuracy is not used as the sole
measure of model quality.

## 2. Data Preparation

-   Dataset size: 7,043 rows and 21 columns.
-   No duplicate rows were found.
-   `TotalCharges` was converted to numeric.
-   Eleven missing `TotalCharges` values were found; these rows had zero
    tenure and no churn.
-   Missing numeric values were handled through median imputation inside
    a scikit-learn pipeline.
-   A stratified train/test split was used.
-   Preprocessing was kept inside the pipeline to reduce data-leakage
    risk.

## 3. Baseline and Accuracy Pitfall

A majority-class `DummyClassifier` predicts every customer as a
non-churner.

  Actual / Predicted     No Churn   Churn
  -------------------- ---------- -------
  Actual No Churn           1,035       0
  Actual Churn                374       0

This baseline achieves approximately 73.46% accuracy, but identifies
zero churners. Churn precision, recall, and F1-score are all zero. This
demonstrates why accuracy alone can be misleading on an imbalanced
dataset.

## 4. Initial Model Metrics

The initial Logistic Regression pipeline used `class_weight="balanced"`
and was evaluated at the default 0.5 threshold.

-   Precision: 0.503
-   Recall: 0.783
-   F1-score: 0.613
-   Balanced accuracy: 0.752
-   Average Precision (PR-AUC summary): 0.6324
-   ROC-AUC: 0.8414

Precision measures how many predicted churners actually churn. Recall
measures how many actual churners are identified. F1-score balances
precision and recall. Average Precision summarizes precision-recall
performance across thresholds, while ROC-AUC measures ranking ability
across thresholds. With an imbalanced target, the precision-recall view
is especially useful alongside ROC-AUC.

## 5. Threshold Selection and Held-Out Test Results

A validation split was created from the training data. The threshold was
selected by maximizing validation F1-score, without using test labels
for threshold selection.

-   Selected threshold: **0.628**
-   Validation precision: 0.603
-   Validation recall: 0.692
-   Validation F1-score: 0.645

The model fitted on the fit subset was then evaluated on the held-out
test set at threshold 0.628:

-   Precision: 0.555
-   Recall: 0.684
-   F1-score: 0.613
-   Balanced accuracy: 0.743
-   Accuracy: 0.770

  Actual / Predicted     No Churn   Churn
  -------------------- ---------- -------
  Actual No Churn             830     205
  Actual Churn                118     256

At this operating point, the model identifies 256 of the 374 actual
churners and misses 118.

**Important comparison limitation:** the initial 0.5-threshold metrics
above came from a model trained on the full training split. The
threshold-tuned model was fitted on a smaller fit subset to preserve a
separate validation set. Therefore, these results are not a controlled
comparison of threshold 0.5 versus 0.628 for the same fitted model.
Differences cannot be attributed to threshold alone. For a clean
threshold comparison, evaluate both thresholds using the same fitted
model and same test probabilities.

## 6. Business Trade-offs

A lower threshold generally flags more customers, potentially increasing
recall but also false positives and unnecessary retention offers. A
higher threshold generally flags fewer customers, potentially reducing
unnecessary interventions but missing more customers who would churn.

The right threshold depends on the cost of contacting a customer who
would not churn, the value of retaining a customer, the cost of missing
a churner, and retention-team capacity. Maximizing F1 is a technical
starting point, not necessarily the threshold that maximizes business
profit.

## 7. Calibration Risk

A calibration curve was generated from test-set predicted probabilities
using quantile bins. It compares average predicted probability with the
observed fraction of churners in each bin. The exact shape of the curve
is not described here; it should be inspected directly.

Because the model uses `class_weight="balanced"`, predicted
probabilities should not automatically be assumed to be calibrated
real-world churn probabilities. If probabilities will inform financial
or high-impact decisions, assess calibration and, if needed, fit a
calibration method using validation data---not the test set.

## 8. Subgroup Performance

These metrics were calculated on the held-out test set at threshold
0.628.

### Contract Type

  ---------------------------------------------------------------------------------
  Contract            Customers       Actual    Precision       Recall     F1-score
                                    Churners                           
  ---------------- ------------ ------------ ------------ ------------ ------------
  Month-to-month            773          329        0.558        0.769        0.647

  One year                  300           36        0.375        0.083        0.136

  Two year                  336            9        0.000        0.000        0.000
  ---------------------------------------------------------------------------------

The model identifies only 3 of 36 actual churners in the one-year group
and none of the 9 actual churners in the two-year group. The two-year
result is based on only nine actual churners and has considerable
uncertainty, but the results warrant investigation before relying on the
model across contract groups.

### Senior Citizen Status

  Group          Customers   Actual Churners   Precision   Recall   F1-score
  ------------ ----------- ----------------- ----------- -------- ----------
  Non-senior         1,187               276       0.538    0.645      0.586
  Senior               222                98       0.600    0.796      0.684

Recall is higher for senior customers in this test set. This difference
should be monitored and investigated; these metrics alone do not
establish whether the model is fair or unfair.

### Gender

  Gender     Customers   Actual Churners   Precision   Recall   F1-score
  -------- ----------- ----------------- ----------- -------- ----------
  Female           687               193       0.579    0.684      0.627
  Male             722               181       0.532    0.685      0.599

Recall is similar for female and male customers in this test set.
Similar recall alone does not establish overall fairness.

## 9. Limitations

-   Evaluation uses one held-out test split; results may vary across
    samples.
-   Subgroup metrics can be unstable when churner counts are small.
-   The threshold maximizes validation F1, not measured business value.
-   The default-threshold and tuned-threshold headline results came from
    differently fitted models, so they are not a direct comparison of
    threshold changes.
-   Calibration was inspected visually, but no recalibration was
    performed.
-   Historical customer behavior may not represent future behavior.
-   Contract-level performance differences require further
    investigation.

## 10. Recommended Risk Controls

1.  Investigate low recall for one-year and two-year contract customers.
2.  Compare threshold choices on the same fitted model and same test
    predictions.
3.  Select a production threshold using documented business costs and
    campaign capacity.
4.  Assess and, if needed, calibrate probabilities using validation
    data.
5.  Monitor precision, recall, F1-score, PR-AUC, calibration, and
    subgroup performance over time.
6.  Re-evaluate when customer behavior or data distribution changes.
7.  Use the model as decision support, not as the sole basis for
    automatic customer decisions.

## 11. Conclusion

The model shows useful ability to rank churn risk, but aggregate metrics
hide substantial differences between contract groups. The majority-class
baseline also shows that accuracy can look reasonable while detecting no
churners.

The model should not be treated as an unquestioned source of truth.
Before deployment, make a same-model threshold comparison, investigate
subgroup differences, and assess calibration and business costs.
