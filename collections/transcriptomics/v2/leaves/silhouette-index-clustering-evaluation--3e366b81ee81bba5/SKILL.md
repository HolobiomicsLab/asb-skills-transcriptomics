---
name: silhouette-index-clustering-evaluation
description: Use when benchmarking a scRNA-seq feature selection or clustering method
  and you need a label-based quantitative score of cluster separation across feature
  set sizes (e.g., 50-4000 genes).
when_to_use_negative:
- No independent cell type annotations are available — the silhouette index requires
  known labels and cannot be computed purely unsupervised
- You need a label-free proxy for choosing feature set size — use the Density Index
  instead, which the paper validated as concordant with SI in 5 of 7 datasets
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Generates the correlation-based ranked feature lists whose clustering quality
    is scored by the silhouette index
  repo: https://github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Recommended framework for normalization (NormalizeData, LogNormalize), PCA,
    and clustering of cells in reduced PC space prior to silhouette computation
  repo: https://satijalab.org/seurat/
- name: R
  role: Implementation environment; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:47337d3a7914c73e64b78eef0d26f593c0d29cb2d03deba049e6c8ccab28f26b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prabhakarlab/DUBStepR
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# silhouette-index-clustering-evaluation

## Summary

Evaluates feature selection and clustering quality in scRNA-seq pipelines by computing the mean scaled silhouette index of cell clusters in PCA space against known cell type labels, as used in the DUBStepR benchmark across 7 datasets.

## When to use

Use when benchmarking a scRNA-seq feature selection or clustering method and you need a label-based quantitative score of cluster separation across feature set sizes (e.g., 50-4000 genes).

## When NOT to use

- No independent cell type annotations are available — the silhouette index requires known labels and cannot be computed purely unsupervised
- You need a label-free proxy for choosing feature set size — use the Density Index instead, which the paper validated as concordant with SI in 5 of 7 datasets

## Inputs

- Log-normalized scRNA-seq count matrix (genes x cells)
- Ranked feature list per feature selection method (e.g., DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop/DANB, GiniClust)
- Independent cell type annotations for each cell
- Feature set sizes to sweep (e.g., 50 to 4000 genes)
- Number of PCs (DUBStepR default: num.pcs = 20, k = 10)

## Outputs

- Mean scaled silhouette index per method per feature set size per dataset (e.g., Fig 4a,b / Supp. Table S6 CSV table)
- Per-dataset ranking of feature selection methods by silhouette index
- Silhouette curve (SI vs. feature set size) identifying the SI-maximizing feature set size

## How to apply

After filtering low-quality genes and log-normalizing each count matrix (e.g., DUBStepR's logNormalize or Seurat's LogNormalize), run the feature selection method to obtain a ranked gene list, then for each feature set size perform PCA (DUBStepR default: num.pcs = 20, k = 10) and cluster cells in PC space using Euclidean distance. Compute the silhouette index of the resulting clustering against independent cell type annotations, and report the mean scaled silhouette index versus feature set size (the DUBStepR paper sweeps 50 to 4000 genes). Decision point: a good feature selection algorithm selects cell-type-specific DE genes as features and optimizes separation between biologically distinct cell clusters, so expect the silhouette curve to peak at a small feature count — DUBStepR approaches its maximum silhouette index value at 200-500 feature genes. Ranking methods by silhouette across datasets (DUBStepR was top-ranked on 5 of 7 benchmark datasets) provides the comparative evaluation output.

## Related tools

- **DUBStepR** (Generates the correlation-based ranked feature lists whose clustering quality is scored by the silhouette index) — https://github.com/prabhakarlab/DUBStepR
- **Seurat** (Recommended framework for normalization (NormalizeData, LogNormalize), PCA, and clustering of cells in reduced PC space prior to silhouette computation) — https://satijalab.org/seurat/
- **R** (Implementation environment; DUBStepR requires R >= 3.5.0)

## Examples

```
DUBStepR(input.data, min.cells = 0.05*ncol(input.data), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- Silhouette curve should reach its maximum at a relatively small feature set size (200-500 feature genes for DUBStepR, per Supp. Fig. S3)
- Method-by-method ranking should reproduce the reported benchmark: DUBStepR was top-ranked on 5 of the 7 benchmark datasets across the 50-4000 gene range
- SI-argmax feature set size should agree with the label-free Density Index argmax in 5 of 7 datasets (Fig 5c,d concordance)
- Clustering must be performed in the same PC space (num.pcs = 20, Euclidean distance) as the silhouette computation for scores to be comparable
- Output table must contain one mean scaled silhouette value per (method, feature set size, dataset) triple with no missing entries

## Limitations

- Silhouette evaluation depends on the quality and granularity of the independent cell type annotations; the paper notes cells were mis-classified (e.g., as T/NK by SingleR due to absence of platelet and RBC transcriptomes in the reference panel)
- Key results such as the silhouette-maximizing feature set sizes are deferred to supplementary figures (Supp. Fig. S3) and notes not included in the main text
- Existing feature selection methods perform inconsistently across datasets, occasionally even resulting in poorer clustering accuracy than without feature selection — baselines must be run per dataset, not assumed uniform

## Evidence

- [intro] the algorithm should select features that optimize the separation between biologically distinct cell clusters: "More importantly, the algorithm should select features that optimize the separation between biologically distinct cell clusters."
- [discussion] DUBStepR approaches its maximum silhouette index value at 200-500 feature genes: "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [readme] Feature selection is a critical step in unsupervised clustering of scRNA-seq data: "Feature selection, i.e. determining the optimal subset of genes to cluster cells into cell types, is a critical step in the unsupervised clustering of scRNA-seq data."
- [methods] DUBStepR function signature and defaults: "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [intro] cells are clustered based on Euclidean distance: "cells are clustered based on their distance from one another (typically, Euclidean distance)"
