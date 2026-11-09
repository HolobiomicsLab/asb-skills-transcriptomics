---
name: imbalanced-cell-type-downsampling
description: Use when stress-testing a cell-type perturbation method (scDist vs nDEG)
  against unbalanced cell-type sample sizes in single-cell RNA-seq data with condition,
  patient, and cluster annotations per cell.
when_to_use_negative:
- 'Data is over-clustered: the article shows condition-specific effects can cause
  over-clustering and downward bias in scDist distances, with median distance decaying
  toward 0 as cluster number increases'
- 'Only raw (unnormalized) counts are available: scDist cannot currently be applied
  directly to raw counts and recommends normalization such as scTransform Pearson
  residuals first'
- The goal is differential abundance (changing cell-type proportions, e.g. DA-seq,
  Milo, Meld) rather than differential state (transcriptomic perturbation within a
  fixed cell type)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDist
  role: Computes pairwise perturbation distances between cell types via per-PC linear
    mixed models that account for individual-to-individual variability
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC mixed-effects models on PC scores within each pairwise cell-type
    comparison
- name: lmerTest
  role: Estimates Satterthwaite degrees of freedom for the per-PC mixed models
- name: ggplot2
  role: Renders the scDist and nDEG dendrograms side by side for visual comparison
- name: irlba
  role: Computes the low-dimensional PCA embedding/loadings used as the plug-in estimator
    for U in scDist
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:a1b126ee833a6ac76b32b441f77cb21cc563f5df3dc589fee30578d21eca41b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# imbalanced-cell-type-downsampling

## Summary

A validation benchmark skill that deliberately downsamples the CD14 monocyte population in the eight sorted immune cell types dataset to create an unbalanced cell-type size distribution, then compares hierarchical clusterings built from scDist versus nDEG pairwise perturbation distances.

## When to use

Use when stress-testing a cell-type perturbation method (scDist vs nDEG) against unbalanced cell-type sample sizes in single-cell RNA-seq data with condition, patient, and cluster annotations per cell.

## When NOT to use

- Data is over-clustered: the article shows condition-specific effects can cause over-clustering and downward bias in scDist distances, with median distance decaying toward 0 as cluster number increases
- Only raw (unnormalized) counts are available: scDist cannot currently be applied directly to raw counts and recommends normalization such as scTransform Pearson residuals first
- The goal is differential abundance (changing cell-type proportions, e.g. DA-seq, Milo, Meld) rather than differential state (transcriptomic perturbation within a fixed cell type)

## Inputs

- Normalized single-cell RNA-seq expression matrix (genes x cells) for eight sorted immune cell types
- Cell-level metadata with condition, patient, and cluster annotations per cell
- Downsampling specification for the CD14 monocyte population to induce an unbalanced cell-type size distribution

## Outputs

- Results data frame of pairwise scDist estimated distances with 95% CI and p-values per cell-type pair
- Results data frame of pairwise nDEG distances per cell-type pair
- Two hierarchical clustering dendrograms (scDist and nDEG distance matrices) rendered side by side with ggplot2

## How to apply

Load the normalized expression matrix and cell-level metadata for the eight sorted immune cell types dataset (condition, patient, cluster per cell), then downsample CD14 monocytes so cell-type sizes are deliberately unbalanced. For every pair of cell types, run scDist with fixed effects for condition, random effects for patient, and a clusters column; internally scDist uses PCA loadings as a plug-in estimator for U and fits per-PC mixed models with lme4, estimating degrees of freedom by Satterthwaite's approximation in lmerTest. In parallel, compute the nDEG distance by counting differentially expressed genes between conditions within each cell type for the same pairs. Assemble both sets of pairwise distances into a results data frame summarizing estimated distance and uncertainty per comparison, then apply hierarchical clustering to each distance matrix and render the two dendrograms side by side with ggplot2. Correct application reproduces the reported Fig. 4C, D outcome: scDist clusters T cells together (correct lineage structure) despite the size imbalance, while nDEG misplaces monocytes in the same cluster as B and T cells.

## Related tools

- **scDist** (Computes pairwise perturbation distances between cell types via per-PC linear mixed models that account for individual-to-individual variability) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC mixed-effects models on PC scores within each pairwise cell-type comparison)
- **lmerTest** (Estimates Satterthwaite degrees of freedom for the per-PC mixed models)
- **ggplot2** (Renders the scDist and nDEG dendrograms side by side for visual comparison)
- **irlba** (Computes the low-dimensional PCA embedding/loadings used as the plug-in estimator for U in scDist)

## Examples

```
out <- scDist(Y, meta.data, fixed.effects="condition", random.effects="patient", clusters="clusters"); dist <- as.dist(pairwise_from(out$results))
```

## Evaluation signals

- The scDist dendrogram places all T cells in a shared cluster despite the downsampled CD14 monocyte imbalance
- The nDEG dendrogram groups monocytes together with B and T cells, matching the reported Fig. 4C, D outcome and demonstrating its sensitivity to unbalanced sizes
- Per-pair results data frame contains an estimated distance plus 95% CI bounds and a p-value for each cell-type pair (schema as in the scDist README's out$results)
- scDist runs completed for every pair of the eight cell types with fixed effects for condition, random effects for patient, and the clusters column supplied

## Limitations

- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances
- Results are sensitive to the choice of the number of principal components used to estimate the distance; no adaptive selection of K exists and a fixed heuristic (20 or 50) is used
- Violations of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- A cluster-free extension of scDist does not yet exist, so significant clustering/annotation errors cause unavoidable bias

## Evidence

- [discussion] per-PC lme4 fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite df via lmerTest: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] over-clustering bias: "condition-specific effects can result in over-clustering and downward bias in the estimated distances"
- [methods] scDist invocation pattern: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response", random.effects="patient", clusters="clusters")"
- [readme] required inputs: "To run `scDist` you will need a normalized expression matrix and metadata assigning each cell to a condition and cluster."
- [readme] results schema: "The results data frame gives a summary of the estimated distance and uncertainty for each cell type"
