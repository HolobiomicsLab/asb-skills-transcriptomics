---
name: single-cell-rnaseq-distance-estimation
description: Use when you have a normalized scRNA-seq expression matrix plus per-cell
  metadata (condition, patient, cluster/cell-type label) with replicates, and you
  need a statistically rigorous per-cell-type distance between condition means rather
  than a differential-abundance or classifier-based.
when_to_use_negative:
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly
  to counts; normalize first
- The question is differential abundance (changing cell proportions) — use DA-seq,
  Milo, or Meld instead of differential-state distance estimation
- Clustering/annotation is unreliable or heavily over-clustered — condition-specific
  effects can cause over-clustering and downward-biased distances, producing false
  negatives
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: 'Core method: mixed-effects-model distance estimation between condition means
    per cell type'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the linear mixed-effects model on PC scores for each principal-component
    loading
- name: lmerTest
  role: Estimates degrees of freedom via Satterthwaite's approximation for the mixed
    models
- name: scTransform
  role: Recommended normalization (Pearson residuals) of counts before distance estimation
- name: ggplot2
  role: Renders dendrograms from hierarchically clustered pairwise scDist distance
    matrices
- name: irlba
  role: Used in the downsampled clustering comparison workflow for low-dimensional
    (PCA) embedding
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:027884beee91dd4c6e0d510457d00691406d07f710b6a254249026628580ab6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rnaseq-distance-estimation

## Summary

Estimate transcriptomic perturbation distances between conditions for predefined cell types in single-cell RNA-seq data using scDist, which fits per-principal-component linear mixed-effects models (lme4, Satterthwaite df via lmerTest) to correct for individual-to-individual and technical variability.

## When to use

Use when you have a normalized scRNA-seq expression matrix plus per-cell metadata (condition, patient, cluster/cell-type label) with replicates, and you need a statistically rigorous per-cell-type distance between condition means rather than a differential-abundance or classifier-based perturbation score.

## When NOT to use

- Input is raw (unnormalized) counts — scDist cannot currently be applied directly to counts; normalize first
- The question is differential abundance (changing cell proportions) — use DA-seq, Milo, or Meld instead of differential-state distance estimation
- Clustering/annotation is unreliable or heavily over-clustered — condition-specific effects can cause over-clustering and downward-biased distances, producing false negatives

## Inputs

- Normalized gene-expression count matrix (genes x cells; scTransform Pearson residuals recommended, not raw counts)
- Cell-level metadata data frame with condition, patient/sample ID, and cell-type cluster annotation columns
- Optionally, paired single-cell datasets for pairwise cell-type-vs-cell-type distance matrices

## Outputs

- Results data frame per cell type with estimated distance, 95% CI (low/upper), and p-value
- Pairwise distance matrix (e.g., pairwise_distances.csv) when looping over cell-type pairs
- Dendrogram (e.g., dendrogram.png via ggplot2) from hierarchical clustering of the distance matrix
- DistPlot visualization of distances and distGenes plot of perturbation-associated genes

## How to apply

Normalize counts first (scTransform Pearson residuals are recommended; scDist is compatible with any normalization but cannot run on raw counts). Install scDist from GitHub and call it on the normalized matrix with fixed.effects = condition, random.effects = patient, and clusters = cell-type annotation; scDist projects cells onto PCA loadings (plug-in estimator for U), fits a mixed model on PC scores for each loading with lme4, and estimates degrees of freedom with Satterthwaite's approximation in lmerTest. The results data frame reports the estimated distance, 95% CI, and p-value (empirical p-value via Monte Carlo draws of summed Wald statistics) per cell type. For pairwise cell-type comparisons, loop scDist over all cell-type pairs, assemble the values into a symmetric distance matrix, and optionally cluster it hierarchically (e.g., stats::hclust with complete linkage) to check lineage structure. Visualize with DistPlot per cell type and inspect perturbation-associated genes with distGenes. Validate that estimated distances align with biology (e.g., hierarchical clustering of the COVID-19 pairwise distances recapitulates immune lineages: T/NK lymphoid cells together, B cells apart, DC/monocytes/neutrophils grouped) and that results are robust when cell-type sizes are unbalanced, where nDEG-based distances can misplace cells across lineages.

## Related tools

- **scDist** (Core method: mixed-effects-model distance estimation between condition means per cell type) — github.com/phillipnicol/scDist
- **lme4** (Fits the linear mixed-effects model on PC scores for each principal-component loading)
- **lmerTest** (Estimates degrees of freedom via Satterthwaite's approximation for the mixed models)
- **scTransform** (Recommended normalization (Pearson residuals) of counts before distance estimation)
- **ggplot2** (Renders dendrograms from hierarchically clustered pairwise scDist distance matrices)
- **irlba** (Used in the downsampled clustering comparison workflow for low-dimensional (PCA) embedding)

## Examples

```
devtools::install_github("phillipnicol/scDist"); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters"); out$results
```

## Evaluation signals

- Distance matrix is symmetric with zero diagonal when built from pairwise scDist runs
- Hierarchical clustering (complete linkage) of COVID-19 pairwise distances reproduces lineage structure: T and NK (lymphoid) together, B cells apart, DC/monocytes/neutrophils (myeloid) grouped, matching Fig. 3D
- On the downsampled (unbalanced CD14 monocyte) eight-cell-type dataset, the scDist dendrogram clusters T cells together while the nDEG dendrogram incorrectly places monocytes with B and T cells (Fig. 4C, D outcome)
- Results data frame contains distance, 95% CI bounds, and p-value per cell type; empirical p-values are Monte Carlo-based (min ~0.00001 in the demo)
- Sanity check against known ground truth: simData demo reports true distances (e.g., a≈31.5, f≈2.76) that scDist's estimates should approximate

## Limitations

- Results are sensitive to the number of principal components used; no adaptive selection of K exists (fixed heuristic of 20 or 50 PCs)
- Double dipping expression data for annotation and distance estimation can cause over-clustering and downward-biased distances (false negatives as cluster number increases)
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free extension of scDist does not yet exist
- Raw counts are not supported; performance may suffer with sub-optimal normalization other than Pearson residuals
- Violation of the spherical-normal (shared variance) assumption can miscalibrate standard errors and hypothesis tests, though it should not affect estimation

## Evidence

- [abstract] Mixed-effects distance model rationale: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [discussion] Mixed-model fitting on PC scores: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] Normalization recommendation: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [methods] Installation from GitHub: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
- [readme] Canonical scDist invocation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [readme] Output schema of results data frame: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
- [discussion] Cell-type filter threshold: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
