---
name: scatter-plot-figure-generation
description: Use when comparing a per-cell quality metric (e.g. silhouette coefficient
  against true cluster labels) between an original data representation and alternative
  feature constructions, and a scatter/diagonal visualization with a fraction-above-diagonal
  summary is needed.
when_to_use_negative:
- The per-cell values to compare are not paired (no original-vs-alternative metric
  for the same cells), so a scatter-vs-diagonal interpretation is meaningless
- The goal is clustering itself or accuracy benchmarking of clustering algorithms
  (e.g. SSCC vs SC3 ARI comparison) rather than visualizing per-cell silhouette accordance
- Only aggregate summary statistics are available — the skill requires per-cell silhouette
  values to draw scatter panels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: sscClust
  role: R package used to load benchmark datasets (e.g. sce.Pollen), construct Spearman-correlation
    and other feature representations, and support silhouette-based scatter visualization
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language for the pipeline (sscClust is an R implementation;
    R linked to optimized BLAS such as MKL/ATLAS is recommended)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:d789b35a9a11717828093d05fe872e8d5ba619372d96fdd1fab5b0df1831c3bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scatter-plot-figure-generation

## Summary

Assembles per-cell scatter panels comparing silhouette values of original scRNA-seq expression profiles against silhouette values of engineered feature representations (Euclidean, cosine, Pearson, Spearman, autoencoder) into a Figure 2-style composite plot, quantifying the fraction of cells above the diagonal.

## When to use

Use when comparing a per-cell quality metric (e.g. silhouette coefficient against true cluster labels) between an original data representation and alternative feature constructions, and a scatter/diagonal visualization with a fraction-above-diagonal summary is needed.

## When NOT to use

- The per-cell values to compare are not paired (no original-vs-alternative metric for the same cells), so a scatter-vs-diagonal interpretation is meaningless
- The goal is clustering itself or accuracy benchmarking of clustering algorithms (e.g. SSCC vs SC3 ARI comparison) rather than visualizing per-cell silhouette accordance
- Only aggregate summary statistics are available — the skill requires per-cell silhouette values to draw scatter panels

## Inputs

- Benchmark scRNA-seq datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) as SingleCellExperiment objects with true cell-type labels (e.g. sce.Pollen from sscClust)
- Per-cell silhouette coefficients computed for the original expression profiles and for each engineered feature representation (Euclidean, cosine, Pearson, Spearman, autoencoder)

## Outputs

- Composite scatter-panel figure (figure2_silhouette_scatter.png) with one panel per dataset showing feature silhouette vs original silhouette
- Fraction of cells above the diagonal per dataset/feature-type comparison

## How to apply

First compute, per dataset (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) and per feature type (Euclidean, cosine, Pearson, Spearman, autoencoder), the silhouette coefficient of each cell against the true cell-type labels alongside the silhouette of the original expression profiles. For each dataset-feature pair, draw a scatter plot with original-data silhouette on one axis and feature silhouette on the other, and count the fraction of cells falling above the diagonal — cells above the diagonal indicate the feature construction improved accordance with true labels. Assemble the per-cell scatter panels into a composite multi-panel figure (Figure 2 style) and export it as figure2_silhouette_scatter.png. Use the sscClust R package (github.com/Japrin/sscClust, with sscVis installed first) for loading the SingleCellExperiment objects and computing/plotting; an optimized BLAS (MKL/ATLAS or Microsoft R open) is recommended for the underlying distance calculations on large data such as Zheng (68,578 cells).

## Related tools

- **sscClust** (R package used to load benchmark datasets (e.g. sce.Pollen), construct Spearman-correlation and other feature representations, and support silhouette-based scatter visualization) — https://github.com/Japrin/sscClust
- **R** (Implementation language for the pipeline (sscClust is an R implementation; R linked to optimized BLAS such as MKL/ATLAS is recommended))

## Examples

```
devtools::install_github("Japrin/sscVis"); devtools::install_github("Japrin/sscClust"); data("sce.Pollen"); sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)
```

## Evaluation signals

- Each panel plots per-cell silhouette of a feature type against silhouette of the original data, with a visible diagonal reference line
- The reported fraction of cells above the diagonal is between 0 and 1 and is computed per dataset and per feature type
- Spearman-correlation features should show the highest fraction-above-diagonal consistently across all five datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng), per the article finding
- Output file figure2_silhouette_scatter.png is produced and contains one scatter panel per dataset-feature combination
- Silhouette values in the panels fall within the valid range [-1, 1]

## Limitations

- The fraction-above-diagonal comparison assumes true cell-type labels are available; silhouette against labels cannot be computed for unlabeled data
- Computing silhouettes on large datasets (e.g. Zheng with 68,578 cells) is expensive; the sscClust README recommends R linked to optimized BLAS (ATLAS, MKL) or Microsoft R open
- sscClust depends on the companion sscVis package, which must be installed first

## Evidence

- [other] For each dataset, draw a scatter plot of silhouette values of the original data versus silhouette values of each feature type, and calculate the fraction of cells falling above the diagonal.: [evidence span withheld — anchor status: not_found]
- [other] Assemble the per-cell scatter panels into the Figure 2-style composite plot and save it as figure2_silhouette_scatter.png.: [evidence span withheld — anchor status: not_found]
- [other] only the Spearman correlation-based method consistently improved the accordance between cell features and true labels across all five benchmark datasets.: [evidence span withheld — anchor status: not_found]
- [readme] data("sce.Pollen"); sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11): "data("sce.Pollen")
sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)"
- [readme] devtools::install_github("Japrin/sscVis"): "devtools::install_github("Japrin/sscVis")"
