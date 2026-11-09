---
name: linear-mixed-effects-modeling-of-patient-variability
description: Use when estimating cell-type-specific transcriptomic perturbation distances
  between conditions in single-cell RNA-seq data where multiple patients/replicates
  per condition require correcting sample-to-sample variability.
when_to_use_negative:
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly
  to raw counts and requires prior normalization (e.g., scTransform Pearson residuals)
- Data are heavily over-clustered — condition-specific effects can cause over-clustering
  and downward bias in estimated distances, with median distance decaying toward 0
  as cluster number increases
- The goal is differential abundance (changing cell proportions) rather than differential
  state — DA-seq, Milo, or Meld are the appropriate tools, not scDist
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: R package implementing the mixed-effects perturbation-distance estimation
    between cell-type condition means
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: fits the per-PC linear mixed-effects model (model 6) on PC scores
- name: lmerTest
  role: estimates νk using Satterthwaite's approximation for the degrees of freedom
- name: scTransform
  role: recommended normalization of counts to Pearson residuals before distance estimation
- name: ggplot2
  role: renders side-by-side dendrograms of scDist vs nDEG distance matrices
- name: irlba
  role: computes the PCA embedding / loadings used as plug-in estimator for U
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:933bad0cb459ad4ab89ca274b32301e1d676728428aa1f8f6949df2b580feca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# linear-mixed-effects-modeling-of-patient-variability

## Summary

Fits per-PC linear mixed-effects models with lme4 (fixed effect for condition, random effect for patient) and Satterthwaite degrees of freedom via lmerTest, as implemented in scDist, to estimate perturbation distances between cell types while accounting for individual-to-individual variability in scRNA-seq data.

## When to use

Use when estimating cell-type-specific transcriptomic perturbation distances between conditions in single-cell RNA-seq data where multiple patients/replicates per condition require correcting sample-to-sample variability.

## When NOT to use

- Input is raw (unnormalized) counts — scDist cannot currently be applied directly to raw counts and requires prior normalization (e.g., scTransform Pearson residuals)
- Data are heavily over-clustered — condition-specific effects can cause over-clustering and downward bias in estimated distances, with median distance decaying toward 0 as cluster number increases
- The goal is differential abundance (changing cell proportions) rather than differential state — DA-seq, Milo, or Meld are the appropriate tools, not scDist

## Inputs

- normalized count/expression matrix (genes x cells; scTransform-normalized counts)
- cell-level metadata data frame with condition, patient, and cluster annotations per cell
- model specification: fixed.effects column (e.g., 'response'), random.effects column (e.g., 'patient'), clusters column

## Outputs

- results data frame per cell type/pairwise comparison: estimated distance, 95% CI (low/upper), p-value
- DistPlot visualization of distances and DistGenes plot of genes associated with the perturbation
- pairwise distance matrices suitable for hierarchical clustering dendrograms

## How to apply

Start from a normalized expression matrix (scTransform Pearson residuals are recommended) plus per-cell metadata with condition, patient, and cluster columns. scDist projects cells onto a low-dimensional PCA embedding, using the loading matrix as a plug-in estimator for U, and fits a mixed-effects model to the PC scores for each of the K principal components with lme4; Satterthwaite's approximation in lmerTest estimates the degrees-of-freedom parameter νk. Fixed effects encode the condition (e.g., response), random effects encode patient, and the clusters column defines cell types. Monte Carlo draws of the sum of Wald statistics yield an empirical p-value, and the results data frame reports the estimated distance, 95% CI, and p-value per cell type. For pairwise comparisons across cell types (e.g., in the CD14-monocyte downsampling benchmark), distances are assembled into a matrix that can be clustered hierarchically; the resulting dendrogram should group T cells together, unlike nDEG-based clustering which misplaces monocytes with B and T cells.

## Related tools

- **scDist** (R package implementing the mixed-effects perturbation-distance estimation between cell-type condition means) — https://github.com/phillipnicol/scDist
- **lme4** (fits the per-PC linear mixed-effects model (model 6) on PC scores)
- **lmerTest** (estimates νk using Satterthwaite's approximation for the degrees of freedom)
- **scTransform** (recommended normalization of counts to Pearson residuals before distance estimation)
- **ggplot2** (renders side-by-side dendrograms of scDist vs nDEG distance matrices)
- **irlba** (computes the PCA embedding / loadings used as plug-in estimator for U)

## Examples

```
out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- Results data frame contains estimated distance, 95% CI lower/upper, and p-value for every cell type in the clusters column
- In the downsampled CD14-monocyte benchmark, the scDist dendrogram clusters T cells together while the nDEG dendrogram incorrectly groups monocytes with B and T cells (matching Fig. 4C, D)
- Estimated distances agree with simulated ground truth in the demo (e.g., cluster 'a' distance ~33 vs true 31.5; cluster 'f' ~0 vs true 2.76)
- Estimated distances and standard errors are stable when patient is included as a random effect; naive fixed-effect-only approaches produce false positives
- Installation completes via devtools::install_github and the simData demo runs in under a minute on a standard machine

## Limitations

- Sensitivity to the choice of the number of principal components K used to estimate the distance; current choice is a fixed heuristic (20 or 50) with no adaptive selection
- Condition-specific effects can cause over-clustering and downward bias in estimated distances, producing false negatives
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free extension of scDist does not yet exist
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- Cannot be applied directly to raw counts; performance may suffer if data are normalized by a sub-optimal approach other than Pearson residuals

## Evidence

- [readme] scDist is an R package that estimates the distance between cell populations in high-dimensional gene-expression space.: "scDist is an R package that estimates the distance between cell populations in high-dimensional gene-expression space. It can be used to measure which cell types change the most between two"
- [readme] mixed-effects model corrects patient variability: "When there are multiple replicates (i.e., patients), `scDist` uses linear-mixed effects models to correct for sample-to-sample variability."
- [discussion] lme4 fits model 6 per PC: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] over-clustering false negatives: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
- [discussion] normalization recommendation: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [abstract] naive approaches false positives: "detecting changing cell types is challenging due to individual-to-individual and cohort-to-cohort variability and naive approaches based on current computational tools lead to false positive findings"
