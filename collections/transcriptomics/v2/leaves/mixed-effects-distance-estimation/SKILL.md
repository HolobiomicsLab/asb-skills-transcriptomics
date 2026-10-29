---
name: mixed-effects-distance-estimation
description: Use when detecting differentially perturbed cell types in scRNA-seq with
  patient replicates, especially when naive methods (Augur AUC, nDEG counts) risk
  false positives from individual-to-individual variability or cell-count confounding.
when_to_use_negative:
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly
  to counts and requires normalized data
- Data are heavily over-clustered or clustering/annotation was informed by the same
  expression data (double dipping), which biases estimated distances downward toward
  false negatives
- Goal is differential abundance (changing cell proportions) rather than differential
  state — use DA-seq, Milo, or Meld instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: 'Core method: mixed-effects distance estimation between condition means per
    cell type on PC scores'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed-effects model to estimate Ubeta_k
- name: lmerTest
  role: Estimates Satterthwaite degrees of freedom nu_k for the Wald statistics
- name: scTransform
  role: Recommended normalization producing Pearson residual counts used as scDist
    input
- name: Augur
  role: Comparison baseline whose AUC-based cell-type scores inflate under null conditions
    (false positives)
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:35e52fddd57b045d246f540b1256259ba1edf43cc860deac31481e1d5a6fd686
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Augur
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mixed-effects-distance-estimation

## Summary

Estimate transcriptomic distances between cell-type populations across conditions using scDist's linear mixed-effects model on PC scores, correcting for individual-to-individual variability while controlling false positives. Provides a statistically rigorous, >60x faster alternative to gene-wise nDEG counting, which is confounded by cells-per-cell-type.

## When to use

Use when detecting differentially perturbed cell types in scRNA-seq with patient replicates, especially when naive methods (Augur AUC, nDEG counts) risk false positives from individual-to-individual variability or cell-count confounding.

## When NOT to use

- Input is raw (unnormalized) counts — scDist cannot currently be applied directly to counts and requires normalized data
- Data are heavily over-clustered or clustering/annotation was informed by the same expression data (double dipping), which biases estimated distances downward toward false negatives
- Goal is differential abundance (changing cell proportions) rather than differential state — use DA-seq, Milo, or Meld instead

## Inputs

- Normalized single-cell expression matrix (genes x cells), scTransform Pearson residuals recommended
- Metadata data frame with columns for condition (fixed effect), patient (random effect), and cluster/cell type
- Optional: number of PCs K (heuristic: 20 or 50)

## Outputs

- Per-cell-type results data frame with estimated distance, 95% CI (low/upper), and p-value
- DistPlot visualization ranking cell types by estimated distance
- distGenes plot of genes associated with the perturbation for a chosen cluster

## How to apply

Install scDist from GitHub (devtools::install_github("phillipnicol/scDist")) and normalize counts with scTransform Pearson residuals. Use the matrix of PCA loadings as a plug-in estimator for U so that Uzij is the vector of PC scores for cell i in sample j, with a fixed heuristic of K = 20 or 50 PCs. For each cell type, fit a per-PC linear mixed model with lme4 (fixed.effects = condition, random.effects = patient, clusters = cell type), estimate Satterthwaite degrees of freedom via lmerTest, and compute an empirical p-value from Monte Carlo draws of the sum of Wald statistics. The results data frame returns estimated distance, 95% CI, and p-value per cell type, which can be visualized with DistPlot and explored per-gene with distGenes. Rationale: the mixed model treats patient effects as random, so the distance between condition means is not inflated by sample-to-sample variability, unlike Augur's AUC which converges toward 1 under null conditions as tau^2 grows.

## Related tools

- **scDist** (Core method: mixed-effects distance estimation between condition means per cell type on PC scores) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed-effects model to estimate Ubeta_k)
- **lmerTest** (Estimates Satterthwaite degrees of freedom nu_k for the Wald statistics)
- **scTransform** (Recommended normalization producing Pearson residual counts used as scDist input)
- **Augur** (Comparison baseline whose AUC-based cell-type scores inflate under null conditions (false positives))

## Examples

```
out <- scDist(sim$Y, sim$meta.data, fixed.effects = "response", random.effects = "patient", clusters = "clusters"); out$results
```

## Evaluation signals

- Under null simulation (simData with beta=0, tau^2 varied 0-1), the estimated distance should not grow with tau^2 and the false-positive rate should be controlled
- Distance estimates should remain approximately constant when cells per cell type are subsampled from 100 to 10,000, while nDEG counts increase monotonically
- Runtime should be at least 60x shorter than gene-wise pseudobulk nDEG differential testing
- Demo: simulated distances (e.g., a=33.04, 95% CI 32.28-33.83) should recover true distances (a=31.50) with valid 95% CIs
- Every cell type in the results data frame should carry a finite distance, CI bounds, and p-value; clusters lacking samples in a condition are excluded per the >10%-of-samples filter

## Limitations

- Results are sensitive to the number of principal components K; no adaptive selection procedure exists (fixed heuristic of 20 or 50 PCs)
- Condition-specific effects combined with double dipping (clustering on the same data) cause over-clustering and downward bias in estimated distances, producing false negatives
- Violation of the spherical-normal (shared variance) assumption can miscalibrate standard errors and hypothesis tests
- Performance may degrade if data are normalized by a sub-optimal approach other than Pearson residuals; significant clustering/annotation errors cause unavoidable bias, and a cluster-free extension does not yet exist

## Evidence

- [abstract] mixed-effects model for robust distance: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [methods] installation via devtools: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
- [discussion] per-PC mixed model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] PC loadings as plug-in estimator: "Use the matrix of PCA loadings as a plug in estimator for U. Then Uzij is the vector of PC scores for cell i in sample j."
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] scDist call signature in README demo: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [abstract] false positives from naive approaches: "naive approaches based on current computational tools lead to false positive findings"
- [discussion] over-clustering false negatives: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
