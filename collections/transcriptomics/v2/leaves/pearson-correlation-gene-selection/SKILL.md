---
name: pearson-correlation-gene-selection
description: Use when imputing dropout-affected zero entries in an scRNA-seq expression
  matrix and you must choose which genes from a pre-defined gene subspace serve as
  regression predictors for a target gene.
when_to_use_negative:
- The entry is not flagged as imputable — under-representation-test-significant (trustworthy)
  zeros must be left untouched; scISR only alters entries identified by hypothesis
  testing as dropout-affected.
- The target gene has no assigned subspace or the subspace contains fewer than 10
  candidate training genes with measurable Pearson correlation, so the top-10 selection
  is not well-defined.
- 'You need a whole-matrix smoothing/denoising imputation: scISR deliberately imputes
  only identified dropout zeros rather than all zero-valued entries, including true
  zeros.'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scISR
  role: Package implementing the subspace-regression imputation pipeline whose third
    module performs the Pearson-correlation predictor selection; available on CRAN
    and GitHub
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Supplies the lm function used to fit the generalized linear regression on
    the selected 10 predictor genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:2c06ca7fbb51a8018d182036294ef4d57cfe2e2969727b9328d60f8a111201cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# pearson-correlation-gene-selection

## Summary

Predictor-selection step from scISR's subspace-regression imputation module: within a gene's assigned subspace, compute Pearson correlation between the target gene y and each candidate training gene x_i and keep the 10 most correlated genes as predictors for the linear model used to estimate dropout-affected zero entries.

## When to use

Use when imputing dropout-affected zero entries in an scRNA-seq expression matrix and you must choose which genes from a pre-defined gene subspace serve as regression predictors for a target gene.

## When NOT to use

- The entry is not flagged as imputable — under-representation-test-significant (trustworthy) zeros must be left untouched; scISR only alters entries identified by hypothesis testing as dropout-affected.
- The target gene has no assigned subspace or the subspace contains fewer than 10 candidate training genes with measurable Pearson correlation, so the top-10 selection is not well-defined.
- You need a whole-matrix smoothing/denoising imputation: scISR deliberately imputes only identified dropout zeros rather than all zero-valued entries, including true zeros.

## Inputs

- Preprocessed gene-by-cell expression matrix (log2-rescaled if value range > 100)
- Imputable-entry mask (zero-valued entries flagged as dropouts by the hyper-geometric/over-representation test at significance threshold 0.01)
- Gene-to-subspace assignment from the data-perturbation grouping module

## Outputs

- Per-target-gene predictor set of the 10 most Pearson-correlated genes within the assigned subspace
- Fitted linear regression model (R stats::lm) used to estimate dropout-affected values
- Imputed expression matrix with nonzero fitted values replacing imputable zeros (e.g., written to imputed_expression_matrix.csv)

## How to apply

After the data-perturbation module assigns each imputable gene to its nearest gene subspace (by Euclidean distance to the subspace centroid), restrict candidate predictors to genes within that subspace. Compute the Pearson correlation between the target gene y and each candidate training gene x_i, rank candidates by correlation, and select the 10 genes with the highest correlation as predictors. On log2-rescaled data (applied when the value range exceeds 100), fit a regression with the lm function from the R stats package and use the fitted model to estimate the zero-valued, dropout-affected entries of the target gene. Limiting predictors to the top 10 correlates keeps the regression local to genes with similar expression patterns and avoids fitting on unrelated genes.

## Related tools

- **scISR** (Package implementing the subspace-regression imputation pipeline whose third module performs the Pearson-correlation predictor selection; available on CRAN and GitHub) — https://github.com/duct317/scISR
- **R stats** (Supplies the lm function used to fit the generalized linear regression on the selected 10 predictor genes)

## Examples

```
imputed <- scISR(data = raw)
```

## Evaluation signals

- Output imputed matrix has the same gene-by-cell dimensions as the input matrix.
- All entries flagged as imputable hold nonzero fitted values produced by the stats::lm regression.
- Exactly 10 predictors are selected per target gene (the 10 genes with highest Pearson correlation within the assigned subspace).
- Downstream acceptance: k-means clustering (true number of cell types k) on the imputed data yields improved ARI versus raw data, e.g., 0.6855 versus 0.5124 on the Goolam (E-MTAB-3321) dataset.
- Imputation only touches hyper-geometric-test-flagged entries; trustworthy (under-representation-significant) zeros remain unchanged.

## Limitations

- The scISR paper/repo states no version for the R stats package or the scISR software, limiting reproducibility.
- Quality of the fitted regression depends on the preceding hypothesis-testing module correctly identifying imputable entries; altering all zero entries including true zeros would introduce false signals.
- The paper's acceptance criterion is indirect — improved k-means ARI on datasets such as Goolam — rather than a direct check of imputed values.

## Evidence

- [methods] Selection of the 10 genes with highest Pearson correlation for the regression model: [evidence span withheld — anchor status: not_found]
- [results] Regression fit on log2-rescaled data with stats::lm: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [intro] Subspace grouping precedes predictor selection: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [intro] scISR restricts imputation to identified dropout entries: "methods in both categories attempt to alter the expression of all zero-valued entries, including those not affected by dropout events."
- [readme] README description of training data and regression-based imputation: "Based on the result obtained from hyper-geometric testing, the original dataset is segregated into two including training data and imputable data."
