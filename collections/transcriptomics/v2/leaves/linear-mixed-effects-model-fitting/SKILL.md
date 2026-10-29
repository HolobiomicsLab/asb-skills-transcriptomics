---
name: linear-mixed-effects-model-fitting
description: Use when detecting perturbed cell types between conditions (e.g., COVID-19
  vs control, immunotherapy responders vs non-responders) from scRNA-seq data with
  replicated patients, where naive per-cell-type tests would inflate false positives.
when_to_use_negative:
- Input is raw (unnormalized) counts — scDist's model requires normalized data and
  cannot yet be applied directly to counts
- The goal is differential abundance (changing cell proportions) rather than differential
  state — DA-seq, Milo, or Meld are the appropriate approaches
- Data are heavily over-clustered — condition-specific effects can cause over-clustering
  and downward bias, with median distance decaying toward 0 (false negatives)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scDist
  role: 'Wraps the mixed-effects distance estimation: fits per-PC mixed models on
    PC scores and returns per-cell-type distances, CIs, and p-values'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the linear mixed-effects model on PC scores corresponding to the k-th
    loading to estimate Uβ
- name: lmerTest
  role: Estimates the degrees of freedom νk via Satterthwaite's approximation for
    Wald tests
- name: scTransform
  role: Recommended normalization producing Pearson residuals used as scDist input
- name: irlba
  role: Computes the low-dimensional PCA embedding used as the plug-in estimator for
    U
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:42432d0c98d7c6acd373c4ba299f310b310f4d0a1bd76d3cd56b3025b36d9658
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - nDEG
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# linear-mixed-effects-model-fitting

## Summary

Fits linear mixed-effects models on PC scores of single-cell RNA-seq data to estimate condition-specific transcriptomic distances per cell type while accounting for patient-to-patient and cohort-to-cohort variability, as implemented in scDist.

## When to use

Use when detecting perturbed cell types between conditions (e.g., COVID-19 vs control, immunotherapy responders vs non-responders) from scRNA-seq data with replicated patients, where naive per-cell-type tests would inflate false positives.

## When NOT to use

- Input is raw (unnormalized) counts — scDist's model requires normalized data and cannot yet be applied directly to counts
- The goal is differential abundance (changing cell proportions) rather than differential state — DA-seq, Milo, or Meld are the appropriate approaches
- Data are heavily over-clustered — condition-specific effects can cause over-clustering and downward bias, with median distance decaying toward 0 (false negatives)

## Inputs

- Normalized scRNA-seq expression matrix (scTransform Pearson residuals recommended; scDist cannot be applied directly to raw counts)
- Cell-level metadata with columns for condition/fixed effect, patient (random effect), and cluster/cell-type annotation
- Number of principal components K (fixed heuristic: 20 or 50)

## Outputs

- Results data frame per cell type with estimated distance, 95% CI (low/upper), and Monte Carlo empirical p-value
- Symmetric pairwise cell-type distance matrix (e.g., pairwise_distances.csv)
- Ranked perturbed-cell-type list and associated gene plots (DistPlot, distGenes)

## How to apply

Normalize counts (scTransform Pearson residuals recommended), exclude any cell type not present in more than 10% of samples, and compute PCA; use the loading matrix as a plug-in estimator for U so that Uzij is the vector of PC scores for each cell. For each cell type and principal-component loading k, fit the linear mixed-effects model with lme4 using the condition (e.g., disease status or responder status) as fixed effect and patient as random effect (e.g., (1|gamma:omega) for nested cohort:patient effects), and estimate the distance between condition means in gene-expression space as a weighted sum over loadings. Obtain degrees of freedom via Satterthwaite's approximation in lmerTest, then compute Monte Carlo empirical p-values from sums of Wald statistics and rank cell types by estimated distance with 95% confidence intervals. Summarize the results data frame (distance, CI low/upper, p-value per cell type) and optionally derive per-gene differential signatures within the top perturbed population.

## Related tools

- **scDist** (Wraps the mixed-effects distance estimation: fits per-PC mixed models on PC scores and returns per-cell-type distances, CIs, and p-values) — github.com/phillipnicol/scDist
- **lme4** (Fits the linear mixed-effects model on PC scores corresponding to the k-th loading to estimate Uβ)
- **lmerTest** (Estimates the degrees of freedom νk via Satterthwaite's approximation for Wald tests)
- **scTransform** (Recommended normalization producing Pearson residuals used as scDist input)
- **irlba** (Computes the low-dimensional PCA embedding used as the plug-in estimator for U)

## Examples

```
out <- scDist(sim$Y, sim$meta.data, fixed.effects = "response", random.effects = "patient", clusters = "clusters"); out$results
```

## Evaluation signals

- Results data frame contains one row per tested cell type with distance, 95% CI (low/upper), and p-value columns
- On the simulated demo (simData, nct=10, N1=N2=5), estimated distances for perturbed clusters (e.g., a≈33, g≈33) are recovered close to the true distances and p-values for true positives are near the Monte Carlo floor (0.0000099999)
- On the COVID-19 dataset, hierarchical clustering of pairwise scDist distances recapitulates lineage structure: lymphoid T/NK together, B cells apart, myeloid DC/monocytes/neutrophils together
- Estimated distance for a non-perturbed cluster is near 0 with a non-significant p-value (e.g., cluster f: 0.000000, p=0.9478205218 in the README demo)
- Top-ranked perturbed populations in immunotherapy cohorts include the FCER1G+ NK cell compartment, validated in bulk transcriptomes from 789 patients

## Limitations

- Results are sensitive to the choice of the number of principal components K; no adaptive selection procedure exists (fixed heuristic of 20 or 50)
- Violations of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- Double dipping (using expression data for both annotation and distance estimation) can cause over-clustering and downward bias in estimated distances
- Sub-optimal normalization other than Pearson residuals may degrade performance; raw counts cannot be used directly (future version)
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free scDist extension does not yet exist

## Evidence

- [discussion] lme4 mixed-model fitting core: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] Recommended normalization: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [readme] README invocation of scDist: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [discussion] 10% sample-presence filter: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
