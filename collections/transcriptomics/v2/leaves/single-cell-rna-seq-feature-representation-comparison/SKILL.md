---
name: single-cell-rna-seq-feature-representation-comparison
description: Use when benchmarking whether feature construction/extraction methods
  improve the accordance between engineered cell features and true cluster labels
  in scRNA-seq data, measured via silhouette values against original data.
when_to_use_negative:
- No true cell-type labels are available for the dataset, since silhouette coefficients
  must be computed against true cluster labels
- The goal is to compare clustering algorithms themselves (e.g. SC3 vs. Seurat vs.
  SIMLR accuracy/runtime) rather than feature construction methods within one clustering
  framework
- Input is already a reduced/feature-space representation; this skill constructs and
  compares representations from original expression profiles
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SSCC
  role: The clustering framework whose Spearman-correlation feature construction motivates
    the silhouette comparison; provides the feature-space methods evaluated
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language; the sscClust R package loads benchmark datasets and
    supports feature construction, silhouette computation, and plotting
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:82a8a09a523c7ee4b54a65c881fa8c39da77a2b82362b00cfed55ce7452952e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-rna-seq-feature-representation-construction-comparison

## Summary

Reproduces the SSCC paper's Figure 2 analysis, which compares alternative feature representations of scRNA-seq data (Euclidean-, cosine-, Pearson-, Spearman-correlation-based, and autoencoder-derived features) against the original expression profiles by computing silhouette coefficients against true cell-type labels on five benchmark datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng).

## When to use

Use when benchmarking whether feature construction/extraction methods improve the accordance between engineered cell features and true cluster labels in scRNA-seq data, measured via silhouette values against original data.

## When NOT to use

- No true cell-type labels are available for the dataset, since silhouette coefficients must be computed against true cluster labels
- The goal is to compare clustering algorithms themselves (e.g. SC3 vs. Seurat vs. SIMLR accuracy/runtime) rather than feature construction methods within one clustering framework
- Input is already a reduced/feature-space representation; this skill constructs and compares representations from original expression profiles

## Inputs

- Five benchmark scRNA-seq datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with true cell-type labels, loadable via sscClust (e.g. sce.Pollen SingleCellExperiment object)
- Original gene expression profile matrix per dataset
- True cluster/cell-type label vector per dataset

## Outputs

- Silhouette coefficient values for each of the six representations (original + 5 feature types) per dataset
- Per-cell scatter panels of original vs. feature-space silhouette values per dataset
- Fraction of cells above the diagonal per dataset and feature type
- Composite Figure 2-style plot saved as figure2_silhouette_scatter.png

## How to apply

Load each of the five benchmark scRNA-seq datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with their true cell-type labels using the sscClust R package (devtools::install_github("Japrin/sscClust"), which requires the sscVis package). For each dataset, construct five alternative feature representations: Euclidean-based, cosine-based, Pearson-correlation-based, Spearman-correlation-based, and autoencoder-derived features, keeping the original expression profiles as the reference. Compute silhouette coefficients of each feature representation against the true cluster labels. For every dataset, scatter-plot per-cell silhouette values of the original data (x-axis) versus silhouette values of each feature type (y-axis) and calculate the fraction of cells falling above the diagonal; cells above the diagonal indicate the feature construction improved label accordance. Assemble the per-cell scatter panels into a Figure 2-style composite plot and save it as figure2_silhouette_scatter.png; the expected qualitative result is that only the Spearman correlation-based method consistently improves accordance across all five datasets.

## Related tools

- **SSCC** (The clustering framework whose Spearman-correlation feature construction motivates the silhouette comparison; provides the feature-space methods evaluated) — https://github.com/Japrin/sscClust
- **R** (Implementation language; the sscClust R package loads benchmark datasets and supports feature construction, silhouette computation, and plotting) — https://github.com/Japrin/sscClust

## Examples

```
library(sscClust); data("sce.Pollen"); sce <- ssc.run(sce.Pollen, subsampling=TRUE, sub.frac=0.8, k.batch=11)
```

## Evaluation signals

- Silhouette coefficients are computable for all six representations (original plus Euclidean, cosine, Pearson, Spearman, autoencoder features) on all five datasets
- Per-dataset scatter panels render original silhouette values on one axis and feature-type silhouette values on the other with a diagonal reference line
- Fractions of cells above the diagonal are reported per dataset and per feature type; the Spearman-based feature should have the highest fraction consistently across Kolodziejczyk, Pollen, Usoskin, Zeisel, and Zheng
- Final composite figure is saved as figure2_silhouette_scatter.png and visually matches the Figure 2 layout
- sscClust loads datasets with correct true labels attached (e.g. sce.Pollen from the package data)

## Limitations

- Conclusions are based on only five benchmark datasets covering a range of scRNA-seq technologies and sequencing depths; only the Spearman correlation-based method consistently improved accordance across all of them
- Computation becomes expensive for large datasets; the sscClust README recommends R linked to an optimized BLAS library (ATLAS, MKL) and Microsoft R Open on Windows
- The autoencoder-derived features depend on training details not fully specified in the cited sections; the discussion notes 'No parameter settings or implementation details for SSCC (e.g., how Spearman correlations are computed) given in this section'

## Evidence

- [other] only the Spearman correlation-based method consistently improved the accordance between cell features and true labels across all five benchmark datasets: [evidence span withheld — anchor status: not_found]
- [discussion] SSCC Spearman feature space: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] sscClust installation: "devtools::install_github("Japrin/sscVis")
devtools::install_github("Japrin/sscClust")"
- [readme] sscClust capabilities: "such as projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [readme] ssc.run example: "data("sce.Pollen")
sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)"
