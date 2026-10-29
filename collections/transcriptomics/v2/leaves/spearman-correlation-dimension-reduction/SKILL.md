---
name: spearman-correlation-dimension-reduction
description: Use when clustering large-scale scRNA-seq data with a subsampling-clustering-classification
  strategy (e.g. k-means on 20-80% subsampled cells) and you need a dimension-reduced
  feature space that retains clustering accuracy (NMI/ARI) better than PCA or raw
  expression.
when_to_use_negative:
- The dataset is small enough that direct clustering on all cells without subsampling
  is computationally feasible and PCA projection already suffices.
- The input is already a reduced/feature table (e.g. pre-computed PCA embeddings)
  rather than a raw gene-by-cell expression matrix.
- The task requires de novo cell-type discovery where Spearman correlation structure
  may not capture the relevant biology.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: sscClust
  role: R package implementing the SSCC pipeline, including variable gene identification,
    Spearman-correlation ('iCor') dimension reduction, subsampling-clustering-classification
    via ssc.run, and NMI/ARI evaluation utilities
  repo: https://github.com/Japrin/sscClust
- name: sscVis
  role: Required dependency package for sscClust installation and visualization
  repo: https://github.com/Japrin/sscVis
- name: R
  role: Implementation language; README recommends R linked to an optimized BLAS library
    (ATLAS, MKL, or Microsoft R Open on Windows) because computation becomes expensive
    on large datasets
- name: SC3
  role: Widely used benchmark clustering method against which SSCC's accuracy (20%
    improvement), speed (50-fold acceleration), and memory usage (66%) were compared
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:7fb0bb27197e3b329136df19789625e3ca9d4f6a3ffdd620e37af425b4133fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SSCC
  - sscClust
  - sscVis
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spearman-correlation-dimension-reduction

## Summary

Projects single-cell RNA-seq data into a feature space defined by Spearman correlations ('iCor'), as implemented in the SSCC framework and the sscClust R package, to improve clustering accuracy and robustness under aggressive subsampling. Spearman-correlation projection makes visualization and clustering more efficient than original or PCA-projected data when only a fraction of cells is used.

## When to use

Use when clustering large-scale scRNA-seq data with a subsampling-clustering-classification strategy (e.g. k-means on 20-80% subsampled cells) and you need a dimension-reduced feature space that retains clustering accuracy (NMI/ARI) better than PCA or raw expression.

## When NOT to use

- The dataset is small enough that direct clustering on all cells without subsampling is computationally feasible and PCA projection already suffices.
- The input is already a reduced/feature table (e.g. pre-computed PCA embeddings) rather than a raw gene-by-cell expression matrix.
- The task requires de novo cell-type discovery where Spearman correlation structure may not capture the relevant biology.

## Inputs

- Single-cell RNA-seq expression matrix as a SingleCellExperiment object (e.g. sce.Pollen bundled with sscClust)
- Known cell-type labels for NMI/ARI evaluation
- Subsampling fraction (sub.frac) and number of clusters (k.batch, e.g. k=11)

## Outputs

- Spearman-correlation ('iCor') projected feature space of cells
- Predicted cluster assignments from k-means on projected data
- NMI and ARI clustering metrics (e.g. written to pollen_clustering_metrics.csv)

## How to apply

Load a single-cell expression matrix as a SingleCellExperiment object (e.g. the Pollen dataset bundled with sscClust) plus known cell-type labels. Select variable genes using a standard-deviation (sd) based filter to retain highly variable genes for clustering. Construct the Spearman correlation ('iCor') projection of the data, in parallel preparing original (unprojected) and PCA-projected versions for comparison. Run subsampling-clustering (e.g. k-means with k=11 at sub.frac such as 0.2 or 0.8 via ssc.run) on each data version. Compute NMI and ARI against true labels with the sscClust evaluation utilities and compare across the three data versions: iCor-projected data should retain high NMI comparable to full-data clustering, while original and PCA-projected data degrade under subsampling. The rationale is that Spearman correlations serve as both feature engineering and projecting methods, reducing computational complexity from O(n^2) to O(n) while improving clustering accuracy and robustness.

## Related tools

- **sscClust** (R package implementing the SSCC pipeline, including variable gene identification, Spearman-correlation ('iCor') dimension reduction, subsampling-clustering-classification via ssc.run, and NMI/ARI evaluation utilities) — https://github.com/Japrin/sscClust
- **sscVis** (Required dependency package for sscClust installation and visualization) — https://github.com/Japrin/sscVis
- **R** (Implementation language; README recommends R linked to an optimized BLAS library (ATLAS, MKL, or Microsoft R Open on Windows) because computation becomes expensive on large datasets)
- **SC3** (Widely used benchmark clustering method against which SSCC's accuracy (20% improvement), speed (50-fold acceleration), and memory usage (66%) were compared)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Spearman ('iCor')-projected results at aggressive subsampling (e.g. 20%) should retain NMI comparable to full-data clustering on the Pollen dataset.
- Original and PCA-projected versions at the same subsampling fraction should show clearly degraded NMI/ARI relative to the iCor projection — if they match iCor, the projection step was likely not applied correctly.
- k-means with k.batch=11 on subsampled cells should be run consistently across all three data versions (original, iCor, PCA) for a fair comparison.
- Evaluation must use the sscClust NMI/ARI utilities against true cell-type labels, and metrics should be written to the expected output file (e.g. pollen_clustering_metrics.csv).
- On large benchmark data (68,578 human blood cells), SSCC should achieve ~20% higher accuracy, ~50-fold speedup, and ~66% memory usage relative to SC3, per the paper's benchmark.

## Limitations

- The SCC (subsampling-clustering-classification) strategy suffers from biases introduced by subsampling which may greatly decrease clustering accuracy and robustness; the Spearman-correlation feature construction is what mitigates this, so omitting iCor projection degrades results.
- The paper section does not state explicit parameter settings or implementation details for how Spearman correlations are computed in SSCC.
- Computation is expensive on large datasets unless R is linked to an optimized BLAS library (ATLAS, MKL) or Microsoft R Open is used on Windows.
- sscClust requires the sscVis package to be installed first.

## Evidence

- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [abstract] SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n): "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [readme] projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient, clustering with subsampling and classfification which make it feasible to process thousands cells' data.: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [readme] we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc. For windows users, Microsoft R open is recommanded: "we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
