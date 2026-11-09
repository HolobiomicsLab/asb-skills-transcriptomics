---
name: linear-mixed-effects-modeling-of-single-cell-pca-scores
description: Use when estimating the distance between condition means in gene-expression
  space for predefined cell types from scRNA-seq data with multiple replicate patients,
  and per-cell-type inference (CI, p-value) on that distance is needed.
when_to_use_negative:
- The goal is differential abundance (changing cell proportions) rather than differential
  state — use DA-seq, Milo, or Meld instead
- Data are raw (unnormalized) counts or heavily over-clustered data where condition-specific
  effects bias distances downward (double dipping)
- Only a single replicate per condition exists, so patient-level random effects cannot
  be estimated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: R package wrapping the PC-projection, per-PC mixed-model fitting, empirical-Bayes
    shrinkage, and Monte Carlo Wald test
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed model with condition as fixed effect and patient
    as random effect
- name: lmerTest
  role: Estimates Satterthwaite degrees of freedom for the Wald statistics
- name: irlba
  role: Truncated SVD used to compute top-K PCA loadings and project cells onto PC
    scores
- name: simData
  role: Generates fully simulated scRNA-seq data (10 cell types, 5 patients per group)
    with ground-truth distances for validation
  repo: github.com/phillipnicol/scDist
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:7c9c5c5a9db41d31076d1a11048956ccb4c93d1a190ed213ed3a5b75ce97bcd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - irlba
  - lme4
  - lmerTest
  - scDist
  - simData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# linear-mixed-effects-modeling-of-single-cell-pca-scores

## Summary

Fits independent per-principal-component linear mixed models to single-cell PC scores to estimate the between-condition transcriptomic distance for a cell type while accounting for individual-to-individual variability. This is the statistical core of scDist, enabling computationally efficient distance estimation with uncertainty quantification.

## When to use

Use when estimating the distance between condition means in gene-expression space for predefined cell types from scRNA-seq data with multiple replicate patients, and per-cell-type inference (CI, p-value) on that distance is needed.

## When NOT to use

- The goal is differential abundance (changing cell proportions) rather than differential state — use DA-seq, Milo, or Meld instead
- Data are raw (unnormalized) counts or heavily over-clustered data where condition-specific effects bias distances downward (double dipping)
- Only a single replicate per condition exists, so patient-level random effects cannot be estimated

## Inputs

- Normalized single-cell count matrix (e.g., genes x cells, scTransform Pearson residuals recommended)
- Per-cell metadata data frame with condition (fixed effect), patient/sample ID (random effect), and cluster/cell-type annotations (e.g., response, patient, clusters)
- Number of principal components K (fixed heuristic, e.g., 20 or 50)

## Outputs

- Per-cell-type results data frame with estimated distance, 95% CI (low/upper), and Monte Carlo p-value
- Posterior distribution of D_K used for inference on the distance
- Optional visualizations via DistPlot and gene-level plots via distGenes

## How to apply

Compute the top-K PCA loadings from the normalized count matrix (truncated SVD via irlba) and project each cell onto the loadings to obtain PC scores, using the PCA loadings as a plug-in estimator for U. For each of the K PCs, fit an independent linear mixed model with lme4 using the condition (e.g., 'response') as fixed effect and 'patient' as random effect. Apply the spike-slab post-hoc Bayesian shrinkage to the fixed-effect coefficients (U beta)_k to correct upward bias when patient numbers are small, then compute the distance D_K from posterior samples. Obtain p-values from empirical Monte Carlo draws of the sum of Wald statistics, with Satterthwaite degrees of freedom estimated by lmerTest. Run the whole procedure via scDist(Y, meta.data, fixed.effects='response', random.effects='patient', clusters='clusters') and read out$results for distance, 95% CI, and p-value per cell type.

## Related tools

- **scDist** (R package wrapping the PC-projection, per-PC mixed-model fitting, empirical-Bayes shrinkage, and Monte Carlo Wald test) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed model with condition as fixed effect and patient as random effect)
- **lmerTest** (Estimates Satterthwaite degrees of freedom for the Wald statistics)
- **irlba** (Truncated SVD used to compute top-K PCA loadings and project cells onto PC scores)
- **simData** (Generates fully simulated scRNA-seq data (10 cell types, 5 patients per group) with ground-truth distances for validation) — github.com/phillipnicol/scDist

## Examples

```
out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- On simulated data (simData, nct=10, N1=5, N2=5), estimated distances for perturbed cell types should closely match sim$D.true (e.g., cell types a and g with true distances ~31 recovered as ~33 with significant p-values)
- Unchanged cell types should be non-significant (e.g., cell type f with true distance ~2.8 gets estimate 0 and p-value ~0.95)
- The results table must contain columns for distance, 95% CI lower bound, 95% CI upper bound, and p.val for every cluster
- Monte Carlo p-values should fall in [0,1] with a floor around 0.0000099999 (10,000 draws)
- Relative error between estimated distance and D.true should be small for strongly perturbed cell types even with only 5 patients per group

## Limitations

- The MLE of the distance can be substantially upward-biased when the number of patients is small, motivating the post-hoc Bayesian shrinkage
- Results are sensitive to the choice of the number of principal components K; no adaptive selection procedure exists (fixed heuristic 20 or 50)
- Condition-specific effects can cause over-clustering and downward bias in estimated distances (double dipping), producing false negatives
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- scDist cannot currently be applied directly to raw counts; normalization (Pearson residuals via scTransform recommended) is required

## Evidence

- [discussion] PCA loadings as plug-in estimator with per-PC mixed models: "Use the matrix of PCA loadings as a plug in estimator for U. Then Uzij is the vector of PC scores for cell i in sample j."
- [discussion] lme4 fitting of per-PC models: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] Concrete scDist call from README demo: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [methods] Simulation validation with 10 cell types and 5 patients per group: "Generate simulated data with 10 cell types and 5 patients in each group"
- [readme] Results data frame schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
