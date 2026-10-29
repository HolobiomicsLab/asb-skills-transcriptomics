---
name: silhouette-coefficient-computation
description: Use when scRNA-seq clustering output must be validated against known
  cell-type labels, or when testing whether engineered features (e.g., Spearman-correlation
  features from sscClust) remain consistent across subsampling rates.
when_to_use_negative:
- No ground-truth cell-type labels are available, so silhouette cannot be computed
  against true labels (external validation is impossible)
- The goal is to compare two unsupervised clusterings to each other rather than clusters
  to reference labels (silhouette against true labels answers a different question)
- Data are not cell-level single-cell clustering results (e.g., bulk RNA-seq or feature-level
  matrices)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: sscClust
  role: R package used to construct Spearman-correlation features, cluster (with optional
    subsampling via ssc.run), and from whose clustering output silhouette values are
    computed
  repo: https://github.com/Japrin/sscClust
- name: SSCC
  role: Spearman subsampling-clustering-classification framework whose features/clusters
    are evaluated via silhouette values against true labels
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language for sscClust and the silhouette/subsampling-robustness
    analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:e3e201e95f4746aefc12d97cf52d1f682146f9f7c4f1ffde09ec0539471a9dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# silhouette-coefficient-computation

## Summary

Computes per-cell silhouette values comparing unsupervised cluster assignments (e.g., from SSCC Spearman-correlation features) against true cell-type labels, to quantify how consistent constructed features/clusters are with ground truth and how robust that consistency is to subsampling.

## When to use

Use when scRNA-seq clustering output must be validated against known cell-type labels, or when testing whether engineered features (e.g., Spearman-correlation features from sscClust) remain consistent across subsampling rates.

## When NOT to use

- No ground-truth cell-type labels are available, so silhouette cannot be computed against true labels (external validation is impossible)
- The goal is to compare two unsupervised clusterings to each other rather than clusters to reference labels (silhouette against true labels answers a different question)
- Data are not cell-level single-cell clustering results (e.g., bulk RNA-seq or feature-level matrices)

## Inputs

- scRNA-seq count matrix (e.g., SingleCellExperiment object such as sce.Pollen bundled with sscClust)
- True cell-type labels for each cell
- Cluster assignments from the clustering run (e.g., SSCC/sscClust output)
- Optional: a set of cell-subsampling rates (e.g., 10%, 20%, 30%, 40%, 50%)

## Outputs

- Per-cell silhouette values
- Silhouette-versus-subsampling-rate scatter plots (one per dataset)
- Pearson correlation coefficient between silhouette values and subsampling rate per dataset

## How to apply

Load the scRNA-seq count matrix together with its true cell-type labels and cluster the cells (e.g., with the sscClust ssc.run pipeline, optionally with subsampling at rates such as 10%-50%). For each cell, compute the silhouette value using the cluster assignment as group and the true label as reference class, so each cell's silhouette reflects agreement of its neighborhood structure with known cell types. To test robustness, repeat feature construction, clustering, and silhouette computation across multiple cell-subsampling rates (10%, 20%, 30%, 40%, 50%), then plot silhouette value versus subsampling rate as scatter plots, one per dataset. Rationale: the SSCC paper uses per-cell silhouette against true labels to show that Spearman-correlation-based features stay consistent with cluster labels regardless of the fraction of cells subsampled, with near-zero Pearson correlation between silhouette values and subsampling rate indicating subsampling-independence.

## Related tools

- **sscClust** (R package used to construct Spearman-correlation features, cluster (with optional subsampling via ssc.run), and from whose clustering output silhouette values are computed) — https://github.com/Japrin/sscClust
- **SSCC** (Spearman subsampling-clustering-classification framework whose features/clusters are evaluated via silhouette values against true labels) — https://github.com/Japrin/sscClust
- **R** (Implementation language for sscClust and the silhouette/subsampling-robustness analysis)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)  # then compute per-cell silhouette against true labels for each sub.frac
```

## Evaluation signals

- Per-cell silhouette values are nearly unchanged across subsampling rates (10%-50%) for each benchmark dataset
- Pearson correlation coefficients between silhouette values and subsampling rates are near zero for all five benchmark datasets
- Silhouette is computed per cell, so output length equals the number of cells in each subsample
- Scatter plots of silhouette value (X) versus subsampling rate (Y) are produced, one per dataset
- ssc.run pipeline completes for each subsample (e.g., sub.frac values matching the chosen rates) without errors before silhouette computation

## Limitations

- Requires reliable true cell-type labels; silhouette against labels is only a proxy for clustering correctness
- The SCC (subsampling-clustering-classification) strategy can introduce subsampling biases that decrease clustering accuracy and robustness, which confounds silhouette-based comparisons if not controlled
- Large datasets make computation expensive; the sscClust README recommends R linked to an optimized BLAS library (ATLAS, MKL; Microsoft R open on Windows)

## Evidence

- [abstract] Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] clustering with subsampling and classfification which make it feasible to process thousands cells' data: "clustering with subsampling and classfification which make it feasible to process thousands cells' data"
- [readme] we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc.: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
