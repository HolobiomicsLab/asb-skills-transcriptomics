---
name: scrna-seq-data-clustering
description: Use when clustering a large scRNA-seq expression matrix (e.g. tens of
  thousands of cells, such as the Pollen dataset or 68,578-cell blood cell dataset)
  and standard k-means or PCA-based clustering is too slow or inaccurate.
when_to_use_negative:
- Input is already a reduced feature space or embedding (e.g. a precomputed PCA or
  SIMLR similarity) rather than a raw expression matrix
- Dataset is small enough that full-data clustering with SC3 or k-means is already
  fast and accurate, so the subsampling-classification machinery adds unnecessary
  complexity
- The goal is de novo estimation of the number of clusters rather than clustering
  with a user-specified k (ssc.run takes k.batch as input)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sscClust
  role: 'R package implementing the SSCC pipeline: ssc.run with subsampling, Spearman
    correlation (''iCor'') feature projection, variable-gene identification, clustering,
    and NMI/ARI evaluation'
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Runtime environment; README recommends R linked to optimized BLAS (ATLAS,
    MKL) or Microsoft R open for performance
- name: SC3
  role: Baseline clustering method against which SSCC's accuracy (20% improvement)
    and speed (50-fold acceleration) were benchmarked
- name: SIMLR
  role: Alternative scRNA-seq clustering/dimensionality-reduction framework benchmarked
    in the same space; included in the enriched index
  repo: BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:abe05f62f2c9cf54e3d48bc92154afee5e0e096400ee0ffa8b94ff9b18a4e5b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SIMLR
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-seq-data-clustering

## Summary

Clustering of single-cell RNA-seq expression matrices using the SSCC (Spearman subsampling-clustering-classification) framework implemented in the sscClust R package, which combines subsampling, Spearman-correlation ('iCor') feature projection, and classification to improve clustering accuracy, robustness, and scalability.

## When to use

Use when clustering a large scRNA-seq expression matrix (e.g. tens of thousands of cells, such as the Pollen dataset or 68,578-cell blood cell dataset) and standard k-means or PCA-based clustering is too slow or inaccurate.

## When NOT to use

- Input is already a reduced feature space or embedding (e.g. a precomputed PCA or SIMLR similarity) rather than a raw expression matrix
- Dataset is small enough that full-data clustering with SC3 or k-means is already fast and accurate, so the subsampling-classification machinery adds unnecessary complexity
- The goal is de novo estimation of the number of clusters rather than clustering with a user-specified k (ssc.run takes k.batch as input)

## Inputs

- Single-cell RNA-seq expression matrix or SingleCellExperiment object (e.g. data('sce.Pollen') from sscClust)
- Known cell-type labels for validation (when available)
- Expected number of clusters (e.g. k.batch = 11 for Pollen)
- Subsampling fraction (e.g. sub.frac = 0.8, or 0.2 for aggressive subsampling)

## Outputs

- Cluster assignments per cell from the SSCC subsampling-clustering-classification pipeline
- iCor-projected (Spearman correlation) feature space representation for clustering/visualization
- Clustering accuracy metrics (NMI, ARI) against true labels, e.g. pollen_clustering_metrics.csv

## How to apply

Load the scRNA-seq data (e.g. the sce.Pollen SingleCellExperiment object bundled with sscClust) and select variable genes using a standard-deviation (sd) based filter. Run ssc.run, optionally with subsampling (sub.frac, e.g. 0.8 or an aggressive 0.2) and k.batch set to the expected number of clusters (e.g. 11 for Pollen). Project data into the Spearman correlation ('iCor') feature space, since the paper shows this feature construction preserves clustering accuracy under aggressive subsampling where original or PCA-projected data degrade dramatically. Compare predicted cluster assignments to known cell-type labels using NMI and ARI (sscClust evaluation utilities) and write metrics such as pollen_clustering_metrics.csv. For large datasets, prefer R linked to an optimized BLAS library (ATLAS, MKL, or Microsoft R open) because the correlation computation becomes expensive.

## Related tools

- **sscClust** (R package implementing the SSCC pipeline: ssc.run with subsampling, Spearman correlation ('iCor') feature projection, variable-gene identification, clustering, and NMI/ARI evaluation) — https://github.com/Japrin/sscClust
- **R** (Runtime environment; README recommends R linked to optimized BLAS (ATLAS, MKL) or Microsoft R open for performance)
- **SC3** (Baseline clustering method against which SSCC's accuracy (20% improvement) and speed (50-fold acceleration) were benchmarked)
- **SIMLR** (Alternative scRNA-seq clustering/dimensionality-reduction framework benchmarked in the same space; included in the enriched index) — BatzoglouLabSU/SIMLR

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- NMI/ARI between predicted clusters and true cell-type labels: Spearman (iCor)-projected results should retain accuracy comparable to full-data clustering even at 20% subsampling
- Invariant: clustering accuracy on original or PCA-projected data degrades dramatically under aggressive (20%) subsampling, while iCor-projected data stays high — the reverse pattern indicates a pipeline error
- Cluster count: with k.batch = 11 on Pollen, the pipeline should return 11 clusters matching the known cell types
- SC3 benchmark: SSCC should show roughly 20% higher clustering accuracy and ~50-fold acceleration with ~66% memory usage relative to SC3 on large datasets
- Metrics file schema: pollen_clustering_metrics.csv contains NMI and ARI rows for all three data versions (original, iCor, PCA) at the tested subsample fraction

## Limitations

- The SCC (subsampling-clustering) strategy suffers from biases introduced by subsampling, which can decrease clustering accuracy and robustness unless Spearman-correlation feature construction is used
- Computation becomes expensive as dataset size grows; the README recommends optimized BLAS libraries (ATLAS, MKL) or Microsoft R open, otherwise runs may be slow
- No explicit parameter settings or implementation details (e.g. how Spearman correlations are computed) are given in parts of the article; users must rely on the package vignettes

## Evidence

- [abstract] Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [abstract] SSCC achieved 20% improvement in clustering accuracy and 50-fold acceleration with 66% memory usage compared to SC3.: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [readme] projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient: "such as projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient, clustering with subsampling and classfification"
- [readme] sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11): "data("sce.Pollen")
sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)"
- [readme] we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc.: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
