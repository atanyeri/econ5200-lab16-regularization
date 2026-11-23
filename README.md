# Regularization: Selection, Shrinkage and Valid Intervals

## Objective
I measured what happens to a treatment estimate and its confidence interval when LASSO chooses the controls from the same data, and I compared that with a method designed to keep the interval honest.

## Methodology
- Simulated a treatment effect of 1.0 among 100 candidate controls, only a few of which were true confounders.
- Ran the common procedure: LASSO picks the controls, then OLS on the controls it kept. I recorded how often its 95% interval covered the true effect across 200 samples.
- Wrote double selection (Belloni, Chernozhukov and Hansen 2014). It runs one LASSO of the outcome on the controls and one of the treatment on the controls, then fits OLS on the union of the two sets of kept controls.
- Ran double selection on the same 200 samples and recorded its coverage and the average number of controls it kept.
- On a second simulated design with known true coefficients, compared each LASSO coefficient with the truth to measure shrinkage, then refitted OLS on the kept variables (post-LASSO).
- Chose the LASSO penalty by 5-fold cross-validation, with my own loop over the folds. I fitted the scaler on each fold's training rows only, then checked the result against scikit-learn's GridSearchCV.
- Packaged the support function, double selection and a coverage study into a module, `src/sparse_tools.py`.

## Key Findings
- **Choosing controls with LASSO breaks the interval.** The "95%" interval from LASSO followed by OLS covered the true effect in only 82.5% of 200 samples. The OLS standard error treats the chosen controls as fixed in advance, so it ignores that they were picked using the same data.
- **Double selection helps, but is not a full repair here.** Coverage rose to 88.5%, with 26.5 controls kept on average. Keeping the controls that predict the treatment protects against dropping confounders. But many of the kept controls were noise picked up from this sample, so the interval still fell short of 95%.
- **LASSO coefficients are shrunk, so they are not effects.** The mean shrinkage of the true coefficients was 4.6%, and the smallest coefficients were shrunk the most. Post-LASSO OLS brought the sizes back close to the truth. A raw LASSO coefficient should not be quoted as an effect or an elasticity.
- **Cross-validation done by hand matches the library.** My loop chose lambda = 0.0501 with a CV MSE of 1.0173, matching GridSearchCV exactly. That only works when the scaler is fitted inside every fold.
