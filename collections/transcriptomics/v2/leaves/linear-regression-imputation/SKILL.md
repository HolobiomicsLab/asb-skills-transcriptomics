---
name: linear-regression-imputation
description: Use when a scRNA-seq count matrix already has an imputable-entry mask
  from the upstream hypothesis-testing module and you need to fill those dropout-affected
  zeros with fitted values via subspace regression.
when_to_use_negative:
- Zeros that the under-representation test marked as trustworthy (true zeros) — scISR's
  imputation module only alters entries flagged imputable by the hypothesis-testing
  module
- Data still containing unfiltered genes (genes expressed in fewer than two cells
  or with less than one percent non-zero entries) — preprocessing must occur first
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: R package implementing the three-module pipeline; its third module performs
    the subspace regression imputation of imputable data using a generalized linear
    regression model
  repo: https://github.com/duct317/scISR
- name: R stats (lm)
  role: supplies the lm function used to fit the regression model that estimates dropout-affected
    values within each gene subspace
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:fdae3b53bb4a002ef81645108458d3e0e32bcde1c66bc7fe99809903442625c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - R stats (lm)
  - scISR
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# linear-regression-imputation

## Summary

A subspace-regression imputation step from scISR in which each imputable gene is assigned to its closest gene subspace, the 10 most Pearson-correlated genes within that subspace are selected as predictors, and a generalized linear regression (stats::lm) on log2-rescaled data estimates dropout-affected zero entries.

## When to use

Use when a scRNA-seq count matrix already has an imputable-entry mask from the upstream hypothesis-testing module and you need to fill those dropout-affected zeros with fitted values via subspace regression.

## When NOT to use

- Zeros that the under-representation test marked as trustworthy (true zeros) — scISR's imputation module only alters entries flagged imputable by the hypothesis-testing module
- Data still containing unfiltered genes (genes expressed in fewer than two cells or with less than one percent non-zero entries) — preprocessing must occur first

## Inputs

- preprocessed gene-by-cell scRNA-seq expression matrix (e.g. Goolam counts in Goolam$data)
- imputable-entry mask (zero entries flagged as dropouts by the upstream over-representation test at significance threshold 0.01)
- data-perturbation gene grouping / subspace centroids from the second scISR module

## Outputs

- fully imputed gene-by-cell expression matrix (imputed_expression_matrix.csv) where imputable entries hold nonzero lm-fitted values
- imputed matrix for downstream PCA + k-means clustering and ARI assessment

## How to apply

Load the preprocessed expression matrix plus the imputable-entry mask produced by the hyper-geometric hypothesis-testing module; if the value range exceeds 100, rescale with a base-2 logarithmic transformation. Using the data-perturbation grouping, assign each imputable gene to the nearest gene subspace by Euclidean distance to the subspace centroid. Within the assigned subspace, compute the Pearson correlation between the target gene y and each candidate training gene x_i and keep the 10 most correlated genes as predictors. Fit a regression with the lm function from the R stats package on the log2-rescaled data and use the fitted model to replace the flagged zero entries with nonzero fitted values, writing the fully imputed matrix (e.g. imputed_expression_matrix.csv). Rationale: restricting regression to the gene's own subspace learns local gene patterns instead of altering all zero entries, avoiding false signals on true zeros.

## Related tools

- **scISR** (R package implementing the three-module pipeline; its third module performs the subspace regression imputation of imputable data using a generalized linear regression model) — https://github.com/duct317/scISR
- **R stats (lm)** (supplies the lm function used to fit the regression model that estimates dropout-affected values within each gene subspace)

## Examples

```
library(scISR); data('Goolam'); imputed <- scISR(data = Goolam$data)
```

## Evaluation signals

- Output matrix exists and has the same gene-by-cell dimensions as the input matrix
- Entries flagged as imputable now hold nonzero fitted values produced by stats::lm regression
- Downstream k-means ARI on the imputed data improves over raw data (e.g. ARI 0.6855 vs 0.5124 on the Goolam dataset)
- All zero entries outside the imputable mask remain unchanged (no over-smoothing of true zeros)

## Limitations

- No version stated for the R stats package or the scISR software in the article
- Key robustness evidence is deferred to supplementary figures (S19, S21, S24) not included in the main text
- The paper's downstream acceptance criterion is qualitative alongside the ARI improvement rather than a direct per-entry error bound

## Evidence

- [readme] training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data: "training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data"
- [discussion] We calculate the Pearson correlation between y and xi and then select the 10 genes: [evidence span withheld — anchor status: not_found]
- [discussion] In our implementation, we adopt the lm function that is available in the stats R package.: "In our implementation, we adopt the lm function that is available in the stats R package."
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
