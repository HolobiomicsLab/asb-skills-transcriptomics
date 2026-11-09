---
name: pca-dimensionality-reduction-of-sc-rna-seq-counts
description: Use when a high-dimensional normalized scRNA-seq count matrix (genes
  x cells) must be embedded into a low-dimensional PC score space so per-PC mixed
  models can be fit for downstream cell-type distance estimation or inference.
when_to_use_negative:
- The data are raw counts — the article notes scDist cannot currently be applied directly
  to raw counts and recommends normalization (e.g. scTransform) first
- The cell population has been over-clustered — the article shows condition-specific
  effects can cause over-clustering and downward bias in estimated distances, leading
  scDist to false negatives
- The goal is differential abundance rather than differential state — abundance-only
  tools (DA-seq, Milo, Meld) do not use expression-space PC projections
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Wrapper method whose low-dimensional embedding step consumes the PCA loadings
    and PC scores for distance estimation
  repo: github.com/phillipnicol/scDist
- name: irlba
  role: Truncated SVD to compute the top-K PCA loadings from the normalized count
    matrix
- name: lme4
  role: Fits the per-PC linear mixed model on the PC scores corresponding to the k-th
    loading
- name: lmerTest
  role: Estimates Satterthwaite degrees of freedom (νk) for the per-PC Wald tests
- name: scTransform
  role: Recommended normalization (Pearson residuals) of raw counts before PCA projection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:b62fc7dd049f3d9f82434ba9b4172cc00ec782d5fa9f894698e4ce3560b55b36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - irlba
  - lme4
  - lmerTest
  - scDist
  - scTransform
  - simData
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pca-dimensionality-reduction-of-sc-rna-seq-counts

## Summary

Project a normalized single-cell RNA-seq count matrix onto top-K PCA loadings (computed via truncated SVD) to obtain low-dimensional PC scores that serve as a plug-in estimator for the projection matrix U in scDist's per-cell-type distance estimation, enabling computationally efficient mixed-model inference per PC.

## When to use

Use when a high-dimensional normalized scRNA-seq count matrix (genes x cells) must be embedded into a low-dimensional PC score space so per-PC mixed models can be fit for downstream cell-type distance estimation or inference.

## When NOT to use

- The data are raw counts — the article notes scDist cannot currently be applied directly to raw counts and recommends normalization (e.g. scTransform) first
- The cell population has been over-clustered — the article shows condition-specific effects can cause over-clustering and downward bias in estimated distances, leading scDist to false negatives
- The goal is differential abundance rather than differential state — abundance-only tools (DA-seq, Milo, Meld) do not use expression-space PC projections

## Inputs

- Normalized single-cell count matrix (genes x cells), e.g. sim$Y with dims 1000 x 5100 in the demo
- Per-cell metadata data frame with condition (e.g. 'response'), patient identifier, and cluster/cell-type labels
- Choice of number of principal components K (fixed heuristic, e.g. 20 or 50)

## Outputs

- Matrix of top-K PCA loadings (plug-in estimator for U)
- Per-cell PC scores obtained by projecting cells onto the loadings
- Per-PC score vectors suitable for fitting per-PC linear mixed models

## How to apply

Start from a normalized expression matrix (the article recommends scTransform Pearson residuals, though any normalization is compatible) and per-cell metadata recording condition, patient, and cluster. Compute the top-K PCA loadings from the normalized counts using truncated SVD (e.g., via irlba), using the matrix of PCA loadings as a plug-in estimator for U, then project each cell onto the loadings to obtain PC scores (Uzij is the vector of PC scores for cell i in sample j). Fit an independent linear mixed model per PC with lme4, using the fixed effect (e.g., 'response') and random effect (e.g., 'patient'), with Satterthwaite degrees of freedom from lmerTest. Note the article's stated sensitivity: results depend on the choice of K, which is currently a fixed heuristic (20 or 50) rather than an adaptive selection.

## Related tools

- **scDist** (Wrapper method whose low-dimensional embedding step consumes the PCA loadings and PC scores for distance estimation) — github.com/phillipnicol/scDist
- **irlba** (Truncated SVD to compute the top-K PCA loadings from the normalized count matrix)
- **lme4** (Fits the per-PC linear mixed model on the PC scores corresponding to the k-th loading)
- **lmerTest** (Estimates Satterthwaite degrees of freedom (νk) for the per-PC Wald tests)
- **scTransform** (Recommended normalization (Pearson residuals) of raw counts before PCA projection)

## Examples

```
library(scDist); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- The PCA is computed on normalized (not raw) counts, e.g. sim$Y of dimension 1000 x 5100 in the demo
- Each cell is projected onto the top-K loadings so PC scores per cell/sample are obtained for downstream lme4 fits
- On simData(nct=10, N1=5, N2=5), the resulting scDist distances approximate sim$D.true: cell types a and g (true ~31) recovered as ~33 with significant p-values, while unchanged cell type f yields distance 0 and a non-significant p-value
- The downstream results table contains distance, 95% CI (low/upper), and p.val columns per cell type
- Changing K changes results (documented sensitivity), so the chosen K should be recorded; default heuristic is 20 or 50

## Limitations

- Results are sensitive to the choice of the number of principal components K; no adaptive selection procedure exists and the current choice is a fixed heuristic (20 or 50)
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- Double dipping expression data for clustering/annotation and distance estimation can cause over-clustering and downward bias in estimated distances; significant annotation errors cause unavoidable bias
- scDist cannot currently be applied directly to raw counts, so the PCA step requires prior normalization

## Evidence

- [discussion] Use PCA loadings as plug-in estimator for U and fit per-PC mixed models with lme4: "Use the matrix of PCA loadings as a plug in estimator for U. Then Uzij is the vector of PC scores for cell i in sample j."
- [discussion] Per-PC mixed models fit with lme4 on PC scores: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom from lmerTest: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] Normalized count matrix input: "To run `scDist` you will need a normalized expression matrix and metadata assigning each cell to a condition and cluster."
- [discussion] Recommended normalization before embedding: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
