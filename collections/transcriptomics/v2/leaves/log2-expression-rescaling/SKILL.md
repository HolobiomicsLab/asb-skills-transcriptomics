---
name: log2-expression-rescaling
description: Use when a scRNA-seq expression matrix (e.g., FPKM/RPKM/CPM values from
  plate-based protocols like SUPeR-seq or Smart-Seq2) has a value range larger than
  100 and will feed into subspace-regression imputation of dropout-affected zero entries.
when_to_use_negative:
- The expression matrix already has a value range of 100 or below (scISR applies the
  log2 transformation only for datasets with a value range larger than 100).
- Values are already log-scale normalized (e.g., log-transformed RPKM/CPM), in which
  case re-applying log2 would distort the scale.
- The goal is downstream cluster assessment of the raw data rather than scISR imputation
  — the README's k-means/ARI evaluation operates on the raw or fully imputed matrix,
  not on the intermediate rescaled matrix.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Package whose imputation pipeline includes the log2 rescaling as a preprocessing
    step before subspace regression; applied via scISR(data = raw)
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the lm function that fits the generalized linear regression on the
    log2-rescaled data used to estimate dropout values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:54dcbe05fa192339aa428238bc4e3d72e8254e3b6906fc1c3ac2bef57090ad31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# log2-expression-rescaling

## Summary

A preprocessing step in the scISR imputation workflow that rescales a single-cell gene-by-cell expression matrix using a base-2 log transformation whenever the value range exceeds 100, so that downstream generalized linear regression (stats::lm) in the subspace-regression imputation module operates on a comparable numeric scale.

## When to use

Use when a scRNA-seq expression matrix (e.g., FPKM/RPKM/CPM values from plate-based protocols like SUPeR-seq or Smart-Seq2) has a value range larger than 100 and will feed into subspace-regression imputation of dropout-affected zero entries.

## When NOT to use

- The expression matrix already has a value range of 100 or below (scISR applies the log2 transformation only for datasets with a value range larger than 100).
- Values are already log-scale normalized (e.g., log-transformed RPKM/CPM), in which case re-applying log2 would distort the scale.
- The goal is downstream cluster assessment of the raw data rather than scISR imputation — the README's k-means/ARI evaluation operates on the raw or fully imputed matrix, not on the intermediate rescaled matrix.

## Inputs

- Preprocessed gene-by-cell scRNA-seq expression matrix (genes expressed in at least two cells and with >=1% non-zero entries)
- Imputable-entry mask from the upstream hyper-geometric hypothesis-testing module

## Outputs

- log2-rescaled gene-by-cell expression matrix (same dimensions as input, only if value range > 100; otherwise the original matrix)
- Rescaled matrix consumed as training data for the stats::lm-based subspace regression imputation

## How to apply

Load the preprocessed gene-by-cell expression matrix (after the standard gene filtering: drop genes expressed in fewer than two cells and genes with less than one percent non-zero entries). Compute the range of the expression values; if the range is larger than 100, apply a logarithmic transformation with base 2 to rescale the data. The rationale is that wide-dynamic-range count-like values (e.g., Goolam CPM values) are stabilized onto a log2 scale before the subspace-regression module assigns imputable genes to the nearest gene subspace by Euclidean distance to the subspace centroid and fits a generalized linear model with the lm function from the R stats package. If the value range is 100 or below, skip the transformation and pass the matrix through unchanged.

## Related tools

- **scISR** (Package whose imputation pipeline includes the log2 rescaling as a preprocessing step before subspace regression; applied via scISR(data = raw)) — https://github.com/duct317/scISR
- **R stats** (Provides the lm function that fits the generalized linear regression on the log2-rescaled data used to estimate dropout values)

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; imputed <- scISR(data = raw)
```

## Evaluation signals

- Verify the trigger condition: log2 rescaling is applied only when the range of the expression values is larger than 100; otherwise the matrix should pass through unmodified.
- Check that the rescaled matrix retains the exact gene-by-cell dimensions of the input matrix.
- Confirm that values estimated by the downstream stats::lm regression (nonzero fitted values for imputable entries) are produced on the log2-rescaled scale.
- Confirm genes used as training data were already filtered: genes expressed in fewer than two cells and genes with less than one percent of non-zero-valued entries must be absent before rescaling.

## Limitations

- The range > 100 threshold is a fixed rule of thumb; no adaptive or user-adjustable cutoff is described in the article.
- No version is stated for the R stats package or the scISR software, limiting reproducibility of the rescaling/regression environment.
- The article notes that methods in both categories of imputation alter zero-valued entries including true zeros — the rescaling step does not itself distinguish true zeros from dropout-affected zeros; that is left to the upstream hyper-geometric testing module.

## Evidence

- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [other] If the range of values exceeds 100, rescale the data using a base-2 logarithmic transformation.: [evidence span withheld — anchor status: not_found]
- [discussion] In our implementation, we adopt the lm function that is available in the stats R package.: "In our implementation, we adopt the lm function that is available in the stats R package."
- [readme] training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data: "training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data"
