# Credit-card-fraud-detection
Fraud detection pipeline, built and actually run against the well-known ULB/Kaggle Credit Card Fraud dataset — 284,807 European cardholder transactions from September 2013, with 492 confirmed frauds (just 0.17% of all transactions). 

Step 1 — Load and explore
No missing data, but extreme class imbalance — this is the central challenge of fraud detection. With imbalance this severe, a model that predicts "not fraud" every single time would still score 99.83% accuracy while catching zero fraud. So accuracy is useless here; we need precision, recall, and PR-AUC instead.

Step 2 — Preprocess
The 28 V1–V28 columns are already PCA-transformed (anonymized for privacy) and roughly standardized. Only Time and Amount are on raw scales, so those need scaling

Step 3 — Stratified train/test split
Random splitting could easily leave the test set with too few (or zero) fraud cases, so we stratify on the target:
Output: Train: 227,845 rows (0.1729% fraud) · Test: 56,962 rows (0.1720% fraud) — the imbalance ratio is preserved in both.

Step 4 — Baseline model: Logistic Regression
class_weight='balanced' pushes the model to catch fraud (91.8% recall — only 8 frauds missed out of 98), but precision craters to 6%: it's flagging 1,389 legitimate transactions as fraud to catch those 90. That's a lot of false alarms for a real system.

Step 5 — Better model: Random Forest
Tree ensembles handle this kind of nonlinear, imbalanced problem much better:
Much more usable: 84% precision, 82% recall — it catches 80 of 98 frauds while only false-flagging 15 legitimate transactions out of nearly 57,000.
Top fraud signals (feature importance): V14, V4, V10, V17, V12 dominate — these are anonymized PCA components, so we can't name what they represent, but they're consistently the strongest fraud indicators across the literature on this dataset.

Step 6 — Tune the decision threshold
The 0.5 cutoff isn't necessarily optimal — in production you'd tune this based on the real cost of a missed fraud vs. a false alarm

![image alt](https://github.com/Frankynuel/Credit-card-fraud-detection/blob/53f6fa59620d0397703c622315d121bfb9e1987d/Screen%20Shot%202026-06-20%20at%206.11.24%20PM.png)






This is the real lever in a fraud system: lower the threshold to catch more fraud at the cost of more false alarms (more customer friction), or raise it to only flag near-certain fraud (less friction, more fraud slips through). The right point depends on your cost of a missed fraud vs. cost of a false decline.

