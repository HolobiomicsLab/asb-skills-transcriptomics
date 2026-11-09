---
name: cell-type-perturbation-detection
description: Use when you have a normalized scRNA-seq count matrix with per-cell metadata
  assigning condition, patient, and cluster, and need to identify which predefined
  cell types are transcriptomically perturbed between two conditions.
when_to_use_negative:
- Goal is differential abundance (changing cell-type proportions) rather than differential
  state — use DA-seq, Milo, or Meld instead
- Only raw counts are available with no normalized matrix — scDist cannot currently
  be applied directly to raw counts
- Data are heavily over-clustered, since condition-specific effects can cause over-clustering
  and downward bias in estimated distances (false negatives)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: 'Core method: mixed-effects model estimating between-condition distance per
    cell type from PC scores'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-principal-component linear mixed model to estimate Uβ
- name: lmerTest
  role: Provides Satterthwaite's approximation for the degrees-of-freedom parameter
    νk
- name: scTransform
  role: Recommended normalization (Pearson residuals) of counts before running scDist
- name: Augur
  role: Comparator method benchmarked against scDist (~5 min vs ~50 s on a 13-sample
    COVID-19 dataset)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:b8233415d46ad96318d59355a122e71d0b4f08681e02679782b774865afd71a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Augur
  - irlba
  - lme4
  - lmerTest
  - nDEG
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# cell-type-perturbation-detection

## Summary

Detect cell types with distinct transcriptomic profiles between conditions (differential state analysis) using scDist, a mixed-effects model that estimates the distance between condition means in gene-expression space while accounting for individual-to-individual and technical variability.

## When to use

Use when you have a normalized scRNA-seq count matrix with per-cell metadata assigning condition, patient, and cluster, and need to identify which predefined cell types are transcriptomically perturbed between two conditions.

## When NOT to use

- Goal is differential abundance (changing cell-type proportions) rather than differential state — use DA-seq, Milo, or Meld instead
- Only raw counts are available with no normalized matrix — scDist cannot currently be applied directly to raw counts
- Data are heavily over-clustered, since condition-specific effects can cause over-clustering and downward bias in estimated distances (false negatives)

## Inputs

- Normalized scRNA-seq expression matrix (e.g., Pearson residuals from scTransform)
- Per-cell metadata data.frame with columns for condition (fixed effect), patient (random effect), and cluster/cell-type labels
- Number of principal components K (heuristic choice of 20 or 50)

## Outputs

- Results data frame with estimated distance, 95% CI (low/upper), and p-value per cell type
- DistPlot visualization of distances per cell type
- distGenes plot of genes associated with the perturbation for a chosen cluster

## How to apply

Install scDist from GitHub via devtools::install_github("phillipnicol/scDist") and normalize counts with scTransform Pearson residuals. Use the matrix of PCA loadings as a plug-in estimator for U, fit per-PC mixed models with lme4 (Satterthwaite degrees of freedom via lmerTest), and compute an empirical p-value via Monte Carlo draws of the sum of Wald statistics; the number of PCs is a fixed heuristic (20 or 50). Call scDist on the normalized matrix with fixed.effects (e.g., "response"), random.effects ("patient"), and clusters ("clusters") columns; exclude any cell type not present in more than 10% of samples before analysis. Summarize the estimated distance, 95% CI, and p-value per cell type from out$results, and visualize with DistPlot and distGenes. The method is computationally fast (~50 s on a 13-sample COVID-19 dataset vs ~5 min for Augur), which matters when benchmarking or scaling to ~1M-cell simulated datasets.

## Related tools

- **scDist** (Core method: mixed-effects model estimating between-condition distance per cell type from PC scores) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-principal-component linear mixed model to estimate Uβ)
- **lmerTest** (Provides Satterthwaite's approximation for the degrees-of-freedom parameter νk)
- **scTransform** (Recommended normalization (Pearson residuals) of counts before running scDist)
- **Augur** (Comparator method benchmarked against scDist (~5 min vs ~50 s on a 13-sample COVID-19 dataset))

## Examples

```
out <- scDist(sim$Y, sim$meta.data, fixed.effects = "response", random.effects = "patient", clusters = "clusters")
```

## Evaluation signals

- out$results contains one row per cell type with columns for Dist., 95% CI (low), 95% CI (upper), and p.val
- On simulated data (simData with nct=10, N1=5, N2=5), estimated distances should approximate sim$D.true (e.g., estimated 33.04 vs true 31.50 for cell type 'a')
- Cell types absent from more than 10% of samples are excluded before analysis
- Timing sanity check: scDist completes in ~50 s on the 13-sample COVID-19 dataset vs ~5 min for Augur, with ~5-fold average speedup on subsamples and ~10 min on a 1M-cell simulated dataset
- True-negative control: a non-perturbed cell type (e.g., simulated 'f') should show distance near 0 with a non-significant p-value

## Limitations

- Results are sensitive to the choice of the number of principal components K; no adaptive selection procedure exists (fixed heuristic of 20 or 50)
- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- scDist cannot currently be applied directly to raw counts; a cluster-free extension does not yet exist

## Evidence

- [abstract] mixed-effects model estimating distances: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [methods] install from GitHub: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
- [discussion] mixed model fitting via lme4: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] filter rare cell types: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [supplementary] runtime benchmark vs Augur: "scDist completed the analysis in around 50 seconds, while Augur required 5 minutes."
