# Predicting Biological Age from DNA Methylation

A regression project predicting biological age from DNA methylation (CpG) biomarkers. This notebook began as an assignment for my Data Science class at UoM; I extended it by comparing against Steve Horvath's 2013 epigenetic clock and some additional cleanup for personal use. I plan on extending this even further by implementing Lasso/Ridge regularization and adding some visualizations.

## The dataset

The dataset contains ~27,500 features (CpG methylation values, each between 0 and 1) across thousands of samples, alongside each sample's chronological age. Every column beyond metadata is a specific, standardized CpG site (e.g. `cg00075967`), which is what makes it possible to directly compare selected features against externally published clocks like Horvath's.

## Approach

1. **Data cleaning** — removed redundant/leakage-prone metadata columns (`Age_group`, `Age_group (2 groups)`, `GSE_number`, `Dataset_ID`), handled missing target values
2. **Train/val/test split** — 60/20/20
3. **Missing value imputation** — median imputation on features (robust to outliers; low missing-value ratio meant mean vs. median made little practical difference here)
4. **Feature scaling** — standardization (z-score), computed from train-set statistics and applied consistently across train/val/test
5. **Feature selection** — correlation-based thresholding against the target, threshold chosen by testing a range of values (0.1–0.55) against validation performance
6. **Modeling** — linear regression baseline, then polynomial regression (degree selected via validation RMSE across degrees 2–4)
7. **Evaluation** — MSE, RMSE, R² reported on held-out test data

### Results

| Model | Train R² | Val R² | Test R² | RMSE (test) |
|---|---|---|---|---|
| Linear Regression (10 features, threshold 0.5) | 0.51 | 0.48 | — | — |
| Polynomial Regression (degree 2, 5 features, threshold 0.52) | 0.53 | 0.45 | 0.48 | 11.90 |

The polynomial model's test performance landed close to the linear model's validation performance, with only a marginal RMSE difference (11.90 vs. 11.85). The gap between polynomial training R² (0.53) and test R² (0.48) points to mild overfitting, likely driven by the small selected feature set (5–10 CpGs) combined with the added flexibility of polynomial expansion.

## Extension: Comparison against the Horvath 2013 clock

Beyond the original assignment scope, I compared my correlation-selected CpG features against the published 353-CpG Horvath epigenetic clock (sourced from the `biolearn` Python package's reference implementation of Horvath's original 2013 coefficients).

**Result:** 2 of my 5 selected features (`cg25809905`, `cg16744741`) are also part of the validated Horvath clock.

**Interpretation:** With ~27,500 candidate CpG sites and only 353 in Horvath's clock, the base rate of a randomly chosen feature landing in the Horvath set is under 2%. Finding 2 overlaps out of just 5 selected features suggests my feature selection is picking up age-informative signal, not just noise, despite using a far simpler method. This is a small-sample result, but a nice sanity check that the model is learning something real rather than spurious dataset-specific correlations.

## Limitations

- Small selected feature set (5–10 features) compared to Horvath's 353 
- Single dataset/cohort 
- No adjustment for confounders

## Planned Additions

- Compare correlation-based selection against Lasso/Ridge regularized feature selection
- Extend the Horvath overlap comparison
- Visualize predicted vs. actual age and residuals across the test set

## Tech stack

Python, pandas, NumPy, scikit-learn (LinearRegression, PolynomialFeatures, train_test_split, SimpleImputer), `biolearn` (for reference Horvath clock coefficients)

## Known issues / changelog

Fixed several bugs from the original coursework version before treating this as portfolio-ready:
- Validation metrics in the threshold-selection loop were previously computed from a stale variable rather than per-threshold validation predictions
- Polynomial degree selection loop was previously reusing training-set metrics instead of validation-set metrics per degree