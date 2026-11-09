---
name: multi-metric-similarity-feature-construction
description: Use when a raw scRNA-seq expression matrix with known cell-type labels
  is available and you must compare which cell-to-cell similarity metric (Euclidean,
  cosine, Pearson, Spearman, autoencoder) aligns engineered features with true labels
  via silhouette values.
when_to_use_negative:
- Input is already a reduced/feature space or similarity matrix rather than the original
  scRNA-seq expression profiles, since silhouette comparisons require the original
  data as the baseline representation
- No true cluster labels exist for the cells, because silhouette coefficients here
  are computed against known cell-type labels, not against predicted clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: sscClust (SSCC)
  role: R package providing the Spearman-correlation feature projection, clustering
    pipeline, and example datasets (sce.Pollen) used to build and evaluate the feature
    representations
  repo: https://github.com/Japrin/sscClust
- name: sscVis
  role: Required companion R package for sscClust visualization/installation
  repo: https://github.com/Japrin/sscVis
- name: R
  role: Implementation language for the sscClust pipeline; optimized BLAS (MKL/ATLAS,
    Microsoft R Open on Windows) is recommended for large datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:a15e525599584f15a5797fd1314f67237cd8b45f97a959a0b77624a7033d08b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust (SSCC)
  - sscVis
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-metric-similarity-feature-construction

## Summary

Constructs alternative feature representations of scRNA-seq cells — Euclidean, cosine, Pearson-correlation, Spearman-correlation, and autoencoder-derived spaces — and judges each by silhouette coefficients computed against true cell-type labels, as in the SSCC framework's Figure 2 analysis.

## When to use

Use when a raw scRNA-seq expression matrix with known cell-type labels is available and you must compare which cell-to-cell similarity metric (Euclidean, cosine, Pearson, Spearman, autoencoder) best aligns engineered features with true labels via silhouette values.

## When NOT to use

- Input is already a reduced/feature space or similarity matrix rather than the original scRNA-seq expression profiles, since silhouette comparisons require the original data as the baseline representation
- No true cluster labels exist for the cells, because silhouette coefficients here are computed against known cell-type labels, not against predicted clusters

## Inputs

- scRNA-seq expression matrices (SingleCellExperiment objects such as sce.Pollen in sscClust) for benchmark datasets Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng
- True cell-type labels for each cell in each dataset

## Outputs

- Per-cell silhouette coefficient values for each feature representation (Euclidean, cosine, Pearson, Spearman, autoencoder) and the original data
- Fraction of cells above the diagonal in each original-vs-feature silhouette scatter panel
- Figure 2-style composite scatter plot (e.g. figure2_silhouette_scatter.png)

## How to apply

Load each benchmark scRNA-seq dataset (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with its true cell-type labels via the sscClust R package. For every dataset, build the original expression representation plus five alternative feature spaces: Euclidean-based, cosine-based, Pearson-correlation-based, Spearman-correlation-based, and autoencoder-derived features. Compute silhouette coefficients of each representation against the true labels, then for each dataset plot per-cell silhouette values of the original data versus each feature type and calculate the fraction of cells falling above the diagonal. A feature space whose fraction above the diagonal exceeds 0.5 on the scatter panels improves accordance between cell features and true labels; in the source study only the Spearman correlation-based construction did so consistently across all five datasets, which is the rationale for SSCC's use of Spearman correlations as its feature engineering and projecting method.

## Related tools

- **sscClust (SSCC)** (R package providing the Spearman-correlation feature projection, clustering pipeline, and example datasets (sce.Pollen) used to build and evaluate the feature representations) — https://github.com/Japrin/sscClust
- **sscVis** (Required companion R package for sscClust visualization/installation) — https://github.com/Japrin/sscVis
- **R** (Implementation language for the sscClust pipeline; optimized BLAS (MKL/ATLAS, Microsoft R Open on Windows) is recommended for large datasets)

## Examples

```
install.packages("devtools"); devtools::install_github("Japrin/sscVis"); devtools::install_github("Japrin/sscClust"); data("sce.Pollen"); sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)
```

## Evaluation signals

- Silhouette scatter panels should show the fraction of cells above the diagonal exceeding 0.5 for the Spearman correlation-based features on all five datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng)
- Silhouette values must be computable for all six representations per dataset (original + 5 feature constructions) against the same true labels, enabling direct pairwise scatter panels
- Composite output plot (figure2_silhouette_scatter.png) must contain per-cell scatter panels, one per dataset per feature type, reproducing the Figure 2 layout
- sscClust pipeline runs (e.g. on sce.Pollen) should complete without error after installing both sscVis and sscClust
- Non-Spearman metrics (Euclidean, cosine, Pearson, autoencoder) should not consistently improve accordance, matching the study's reported finding

## Limitations

- Only the Spearman correlation-based feature construction consistently improved accordance between cell features and true labels; Euclidean, cosine, Pearson, and autoencoder representations did not improve consistency across all five benchmark datasets
- Subsampling strategies (as in the SCC strategy) can introduce biases that decrease clustering accuracy and robustness, even though they reduce computational complexity from O(n2) to O(n)
- Computation becomes expensive as dataset size grows; the sscClust README recommends R linked to optimized BLAS (MKL, ATLAS) or Microsoft R Open on Windows

## Evidence

- [abstract] Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
- [readme] Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc.: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
