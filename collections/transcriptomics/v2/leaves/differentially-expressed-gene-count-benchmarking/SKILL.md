---
name: differentially-expressed-gene-count-benchmarking
description: Use when benchmarking whether per-PC mixed-model distances (scDist) outperform
  nDEG gene counts on sorted immune cell types with deliberately unbalanced cell-type
  sizes, e.g. after downsampling CD14 monocytes to test dendrogram consistency.
when_to_use_negative:
- 'Data are over-clustered: condition-specific effects cause over-clustering and downward
  bias, driving scDist median distance toward 0 (false negatives), so the benchmark
  comparison would not be meaningful'
- Only raw counts are available without normalization — scDist requires a normalized
  expression matrix (scTransform Pearson residuals) and cannot be applied directly
  to counts
- The goal is differential abundance (changing cell proportions, e.g. DA-seq/Milo/Meld)
  rather than differential state / transcriptomic perturbation distances
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Computes the pairwise perturbation distance between cell types using per-PC
    linear mixed models (fixed condition effect, random patient effect)
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-principal-component mixed-effects model for each cell-type pair's
    distance estimate
- name: lmerTest
  role: Estimates Satterthwaite degrees of freedom for the mixed-model test statistics
- name: ggplot2
  role: Renders the two hierarchical-clustering dendrograms side by side for visual
    benchmark comparison
- name: irlba
  role: Efficient PCA used to obtain the low-dimensional embedding / PC loadings as
    the plug-in estimator for U
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:6b0edfa7e02870b5bf4e002d491e0343fb9d4603cdffdcaf91828341ae3cec91
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differentially-expressed-gene-count-benchmarking

## Summary

Benchmark a mixed-model perturbation distance (scDist) against naive nDEG (count of differentially expressed genes) by computing both pairwise distances across cell types, comparing the resulting hierarchical clusterings (dendrograms), and checking which method recovers biologically expected lineage groupings under an unbalanced cell-type size distribution.

## When to use

Use when benchmarking whether per-PC mixed-model distances (scDist) outperform nDEG gene counts on sorted immune cell types with deliberately unbalanced cell-type sizes, e.g. after downsampling CD14 monocytes to test dendrogram consistency.

## When NOT to use

- Data are over-clustered: condition-specific effects cause over-clustering and downward bias, driving scDist median distance toward 0 (false negatives), so the benchmark comparison would not be meaningful
- Only raw counts are available without normalization — scDist requires a normalized expression matrix (scTransform Pearson residuals) and cannot be applied directly to counts
- The goal is differential abundance (changing cell proportions, e.g. DA-seq/Milo/Meld) rather than differential state / transcriptomic perturbation distances

## Inputs

- Normalized single-cell expression matrix (scTransform Pearson residuals recommended) for eight sorted immune cell types
- Cell-level metadata with condition, patient, and cluster annotations per cell
- Downsampling specification to unbalance the CD14 monocyte population

## Outputs

- Pairwise scDist distance matrix with estimated distance and uncertainty (95% CI, p-value) per cell-type pair
- Pairwise nDEG distance matrix across the same cell-type pairs
- Side-by-side dendrograms (ggplot2) comparing hierarchical clusterings from scDist vs nDEG distances

## How to apply

Load the normalized expression matrix and per-cell metadata (condition, patient, cluster) for the eight sorted immune cell types dataset, then downsample the CD14 monocyte population so the cell-type size distribution is deliberately heterogeneous. For every pair of cell types, run scDist with fixed effects for condition, random effects for patient, and a clusters column — scDist fits a linear mixed model per principal component using lme4 and estimates Satterthwaite degrees of freedom via lmerTest on PCA loadings used as a plug-in estimator for U. For the same pairs, compute the nDEG distance by counting differentially expressed genes between conditions within each cell type. Assemble both pairwise-distance matrices into a results data frame with estimated distance and uncertainty, apply hierarchical clustering to each matrix, and render the two dendrograms side by side with ggplot2. The rationale is that naive approaches lead to false positives due to individual-to-individual and cohort-to-cohort variability, so the mixed-model distance should cluster T cells together while nDEG misplaces monocytes with B and T cells.

## Related tools

- **scDist** (Computes the pairwise perturbation distance between cell types using per-PC linear mixed models (fixed condition effect, random patient effect)) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-principal-component mixed-effects model for each cell-type pair's distance estimate)
- **lmerTest** (Estimates Satterthwaite degrees of freedom for the mixed-model test statistics)
- **ggplot2** (Renders the two hierarchical-clustering dendrograms side by side for visual benchmark comparison)
- **irlba** (Efficient PCA used to obtain the low-dimensional embedding / PC loadings as the plug-in estimator for U)

## Examples

```
out <- scDist(Y, meta.data, fixed.effects = "condition", random.effects = "patient", clusters = "clusters")
```

## Evaluation signals

- Validation criterion from the source card: the scDist dendrogram places T cells in a shared cluster (matching reported Fig. 4C outcome)
- The nDEG dendrogram groups Monocytes together with B and T cells despite different lineages, demonstrating the naive method's false grouping (Fig. 4D)
- The results data frame reports an estimated distance plus 95% CI (low/upper) and p-value for each cell-type pair, with CIs non-degenerate
- Both methods use the same cell-type pairs and downsampled data, so distance matrices are directly comparable element-wise

## Limitations

- Results are sensitive to the choice of the number of principal components used to estimate the distance; the current choice is a fixed heuristic (20 or 50) with no adaptive selection of K
- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances, so scDist can produce false negatives when the data is over-clustered
- Significant clustering/annotation errors could cause unavoidable bias; a cluster-free extension of scDist does not yet exist
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests

## Evidence

- [other] nDEG places the Monocytes in the same cluster as B and T cells, despite these belonging to different lineages: [evidence span withheld — anchor status: not_found]
- [other] scDist produced the expected result of clustering the T cells together: "scDist produced the expected result of clustering the T cells together, whereas nDEG places the Monocytes in the same cluster as B and T cells"
- [discussion] Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] νk is estimated using Satterthwaite's approximation in lmerTest: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] The results data frame gives a summary of the estimated distance and uncertainty for each cell type: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
