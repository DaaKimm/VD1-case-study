# Results

## Recorded Choice

Boosted trees

## Validation AUC Results

| Method | Validation AUC |
|---|---:|
| Boosted trees | 0.845558 |
| Logistic regression | 0.838402 |
| Contract rule | 0.742724 |

My validation-stage choice is boosted trees because it has the highest validation AUC: 0.845558, compared with 0.838402 for logistic regression and 0.742724 for the contract rule.

## Data Cleaning Decisions

- The supplied CSV contains 7,043 rows and 21 columns. `Churn` counts are 5,174 `No` and 1,869 `Yes`.
- There are 11 blank `TotalCharges` values. All occur where `tenure` is 0; the script converts them to `0.0`.
- No rows are removed. The script does not remove, cap, or otherwise treat outliers.
- The three numeric inputs (`tenure`, `MonthlyCharges`, and `TotalCharges`) are standardized, and the four categorical inputs (`Contract`, `InternetService`, `PaperlessBilling`, and `PaymentMethod`) are one-hot encoded. These preprocessing steps are fit using training rows only.

The 11 customers already had tenure = 0, so the script simply replaced their blank TotalCharges values with 0 and kept the rows, after confirming all 11 blanks belonged to zero-tenure cases. Outliers were left untouched, since the script applied no outlier detection or treatment rule. The three numeric inputs were standardized to make their scales comparable for logistic regression. Finally, the script used training data only, since training-only preprocessing prevents validation or test information from leaking into model fitting.

## Data Separation

| Partition | Rows | Churn Yes | Churn No |
|---|---:|---:|---:|
| Training | 4,225 | 1,121 | 3,104 |
| Validation | 1,409 | 374 | 1,035 |
| Final test | 1,409 | 374 | 1,035 |

All 7,043 source rows appeared exactly once, with no overlap among the training, validation, and final-test partitions. Preprocessing and model fitting used training rows only.

## Final-Test AUC Results

| Method | Final-test AUC | 95% bootstrap interval | Top-20% observed churn rate |
|---|---:|---:|---:|
| Contract rule | 0.737265 | [0.716259, 0.755731] | 39.86% |
| Logistic regression | 0.847143 | [0.825415, 0.869946] | 69.40% |
| Boosted trees | 0.849665 | [0.828267, 0.873164] | 69.40% |

Boosted trees had the highest AUC at 0.849665, followed closely by logistic regression at 0.847143. Both models scored well above the contract rule's 0.737265, a gap of about 0.11 AUC points.

Both prediction models' top-20% lists had a 69.40% churn rate—about 2.6 times the overall 26.54% rate and 29.54 percentage points above the contract rule's 39.86%.

This means the models concentrate far more historical churners within Devon's 20% contact limit than the contract rule does, though whether contacting them actually reduces churn still needs to be tested.

## Bootstrap Uncertainty

| Paired AUC difference | Estimate | 95% bootstrap interval | Interval crosses zero? |
|---|---:|---:|---|
| Logistic regression − Contract rule | 0.109878 | [0.093136, 0.128450] | No |
| Boosted trees − Contract rule | 0.112400 | [0.095891, 0.131117] | No |

The paired boosted-trees-minus-logistic interval is [-0.004862, 0.010284]. Because this interval crosses zero, the data do not establish that boosted trees outperforms logistic regression.

By contrast, the paired comparisons against the contract rule tell a clearer story. The logistic-minus-contract interval is [0.093136, 0.128450], and the boosted-trees-minus-contract interval is [0.095891, 0.131117]; neither crosses zero. This indicates both logistic regression and boosted trees rank churn risk better than the contract rule.

## Method Comparison and Carry-Forward Decision

| Method | Final-test AUC | ΔAUC vs. contract | 95% interval for ΔAUC | Carry forward? | Reason |
|---|---:|---:|---:|---|---|
| Contract rule | 0.737265 | — | — | No | Lowest AUC; baseline being evaluated against. |
| Logistic regression | 0.847143 | 0.109878 | [0.093136, 0.128450] | No | Comparable with boosted trees; can be an alternative choice. |
| Boosted trees | 0.849665 | 0.112400 | [0.095891, 0.131117] | Yes | Highest validation AUC; selected before final evaluation. |

## Probability Accuracy

| Scope | Customers | Mean predicted churn | Observed churn | Prediction − observed |
|---|---:|---:|---:|---:|
| Overall final test | 1,409 | 27.05% | 26.54% | +0.50 pp |
| Top-20% contact list | 281 | 64.79% | 69.40% | −4.61 pp |

| Probability group | Customers | Mean prediction | Observed churn | Prediction − observed |
|---|---:|---:|---:|---:|
| 0.0–0.2 | 724 | 7.96% | 7.18% | +0.78 pp |
| 0.2–0.4 | 272 | 29.78% | 25.37% | +4.41 pp |
| 0.4–0.6 | 236 | 49.63% | 52.12% | −2.49 pp |
| 0.6–0.8 | 142 | 67.69% | 69.01% | −1.33 pp |
| 0.8–1.0 | 35 | 83.45% | 91.43% | −7.98 pp |

Overall, the model's average predicted probability was close to the observed rate: 27.05% predicted versus 26.54% observed. But this closeness masks weaker calibration within specific risk groups. Among the top-20% list of 281 customers ranked at highest churn risk, predicted churn was 64.79% versus an observed 69.40%, an underestimate of 4.61 percentage points.

The five probability-group table shows the mismatch grows further within the highest probability bucket, which contains only 35 customers and was underestimated by 7.98 percentage points.

In short, the overall averages were close, but calibration was not equally accurate across risk groups, and the largest mismatch came from the 35-customer bucket, too small a group to support a confident conclusion about systematic underestimation at that extreme.

## Value Scenarios

The selected boosted-trees list had an observed churn rate of 69.40%.

| Assumed save rate | Net value per 1,000 contacts |
|---:|---:|
| 10% | −$1,619.93 |
| 15% | $670.11 |
| 20% | $2,960.14 |

The break-even save rate is 13.54%.

The calculation uses:

`Net value per 1,000 contacts = 1,000 × (churn rate × save rate × $66 − $6.20)`

**Independent check of the 15% scenario:**

- The selected list contains 281 customers, including 195 observed churners.
- At a 15% save rate: `195 × 0.15 = 29.25` assumed saves.
- Retention value: `29.25 × $66 = $1,930.50`.
- Contact cost: `281 × $6.20 = $1,742.20`.
- Net for 281 contacts: `$1,930.50 − $1,742.20 = $188.30`.
- Scaled to 1,000 contacts: `($188.30 / 281) × 1,000 = $670.11`.

This count-based calculation matches the supplied scenario result.

At a 10% assumed save rate, the retained value per contact (0.693950 × 0.10 × $66 ≈ $4.58) is smaller than the $6.20 cost per contact, so the campaign loses money—net value comes out negative at −$1,619.93 per 1,000 contacts. At 15%, the retained value per contact (0.693950 × 0.15 × $66 ≈ $6.87) exceeds the $6.20 cost, flipping the campaign to a net positive $670.11 per 1,000 contacts. The break-even point, 13.54%, is the save rate at which retained value exactly offsets the contact cost.

The data do not tell us the true save rate—the fraction of contacted customers who otherwise would have churned but are retained because of the offer. The 10%, 15%, and 20% figures illustrate how sensitive profitability is to that unknown. The $66 retained-value figure and $6.20 contact cost are also fictional planning assumptions, so these scenarios frame possible outcomes rather than predict an actual financial result.
