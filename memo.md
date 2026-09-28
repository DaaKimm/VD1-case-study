# Memo

**To:** Devon Achebe, VP of Customer Retention  
**From:** Dahyun Kim  
**Date:** September 27, 2026  
**RE:** Recommendation for the 20% retention contact list

## Recommendation

Use the boosted-trees model to choose the 20% of customers to contact, but don't launch the offer broadly yet. Run a randomized pilot first. The model shows who is likely to leave, not whether the offer keeps them.

## Supporting Evidence

- Targeting: Overall churn is 26.54%. Observed churn within the selected 20% list is 39.86% for the contract rule and 69.40% for both boosted trees and logistic regression. Test AUC: 0.8497 boosted, 0.8471 logistic, 0.7373 contract rule.

- Model choice: Both models beat the contract rule; those paired bootstrap AUC intervals excluded zero. The paired boosted-minus-logistic AUC interval crossed zero, so the data do not show boosted trees is better. Logistic regression is an acceptable, simpler alternative.

- Break-even: Under fictional assumptions, the offer breaks even at a 13.54% save rate. Per 1,000 contacts: −$1,619.93 at 10%, +$670.11 at 15%, +$2,960.14 at 20%.

## Limitations

The analysis is predictive, not causal. Historical data cannot show how many would-be churners the offer saves, so the save rate and dollar figures are assumptions. Calibration is also imperfect: in the top-20% list the model predicted 64.79% churn versus 69.40% observed. Use it to rank customers, not to quote exact probabilities.

## Next step: randomized pilot

Randomly split the model's top-20% list into an offer group and a no-offer holdout, then compare churn after a set follow-up period. Let C be control churn and T be offer-group churn. The absolute effect is C − T; the save rate is (C − T) / C. Example: 60% control churn and 50% offer churn is a 10-point reduction, but a 16.7% save rate. Size the pilot to distinguish results above and below 13.54%, and record actual offer cost and customer value.

## What would change this recommendation

- Expand if the pilot shows a credible save rate above 13.54% and positive net value.

- Stop or redesign if the save rate is below 13.54% or the offer has no measurable effect.