---
name: kmeans-clustering-with-subsampling
description: Use when clustering large-scale scRNA-seq data (e.g., tens of thousands
  of cells) where full-data k-means or SC3-style clustering is too slow or memory-heavy,
  and robustness to subsampling bias matters.
when_to_use_negative:
- Dataset is small enough that full-data clustering (subsampling=F) is tractable —
  subsampling adds unnecessary bias risk
- The analysis goal is de novo estimation of the number of clusters rather than clustering
  at a fixed k
- Input is already a low-dimensional embedding rather than a gene-by-cell expression
  matrix requiring variable-gene selection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SSCC
  role: 'The subsampling-clustering-classification framework being applied: k-means
    on a random cell subsample with classification of remaining cells'
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package implementing the pipeline (ssc.run), iCor projection, variable gene
    identification, and NMI/ARI evaluation utilities
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: 'Benchmarked comparator: SSCC achieved 20% higher accuracy and 50-fold acceleration
    vs SC3 on 68,578 PBMCs'
- name: SIMLR
  role: Alternative single-cell clustering tool referenced as prior art; not part
    of this skill's execution path
  repo: BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:30cd90ecc892a97bf5afd85a9422484c96407ce128690ddc1a87a49a5eb4409a
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# kmeans-clustering-with-subsampling

## Summary

A subsampling-clustering-classification scheme (SSCC) in which k-means is run on a random cell subsample and the resulting clusters are used to classify the remaining cells, reducing complexity from O(n^2) to O(n). Accuracy and robustness under aggressive subsampling are preserved by clustering in a Spearman correlation ('iCor') feature space.

## When to use

Use when clustering large-scale scRNA-seq data (e.g., tens of thousands of cells) where full-data k-means or SC3-style clustering is too slow or memory-heavy, and robustness to subsampling bias matters.

## When NOT to use

- Dataset is small enough that full-data clustering (subsampling=F) is tractable — subsampling adds unnecessary bias risk
- The analysis goal is de novo estimation of the number of clusters rather than clustering at a fixed k
- Input is already a low-dimensional embedding rather than a gene-by-cell expression matrix requiring variable-gene selection

## Inputs

- Single-cell RNA-seq expression matrix with cell-type labels (e.g., sce.Pollen SingleCellExperiment object bundled with sscClust)
- Number of clusters (k.batch, e.g., 11 for the Pollen dataset)
- Subsampling fraction (sub.frac, e.g., 0.8 in README; 20% subsample in the vignette task)

## Outputs

- Cluster assignment labels for all cells (subsampled-cluster plus classified remainder)
- NMI and ARI clustering accuracy metrics vs. true cell-type labels (e.g., pollen_clustering_metrics.csv)
- Projected feature spaces: original, PCA-projected, and Spearman correlation (iCor) projected data

## How to apply

Load the scRNA-seq expression matrix with known labels (e.g., sce.Pollen bundled with sscClust) and select highly variable genes using a standard-deviation (sd) based filter. Build three feature versions of the data: the original expression matrix, a PCA projection, and a Spearman correlation ('iCor') feature space projection, since the paper shows the iCor space confers robustness to subsampling bias. Randomly subsample a fraction of cells (the vignette card uses 20% subsampling; the README example uses sub.frac=0.8), run k-means with the chosen cluster number (k.batch=11 for Pollen's 11 cell types), then classify the held-out cells. Evaluate predicted labels against true cell types with NMI and ARI using sscClust evaluation utilities and expect the iCor-projected subsampled run to retain accuracy comparable to full-data clustering while original/PCA versions degrade.

## Related tools

- **SSCC** (The subsampling-clustering-classification framework being applied: k-means on a random cell subsample with classification of remaining cells) — https://github.com/Japrin/sscClust
- **sscClust** (R package implementing the pipeline (ssc.run), iCor projection, variable gene identification, and NMI/ARI evaluation utilities) — https://github.com/Japrin/sscClust
- **SC3** (Benchmarked comparator: SSCC achieved 20% higher accuracy and 50-fold acceleration vs SC3 on 68,578 PBMCs)
- **SIMLR** (Alternative single-cell clustering tool referenced as prior art; not part of this skill's execution path) — BatzoglouLabSU/SIMLR

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- NMI/ARI of the iCor-projected subsampled clustering should be comparable to full-data (subsampling=F) clustering on the same dataset
- Original and PCA-projected versions at the same subsampling rate (e.g., 20% subsample, k=11 on Pollen) should show markedly degraded NMI/ARI, serving as a negative control
- Predicted cluster count matches k.batch (e.g., 11 for Pollen's known cell types)
- Metrics written to pollen_clustering_metrics.csv cover all three data versions (original, iCor, PCA)
- Runtime/memory should be reduced relative to full-data clustering, consistent with SSCC reducing O(n^2) to O(n)

## Limitations

- Plain subsampling-clustering (SCC) without Spearman feature construction suffers subsampling biases that decrease clustering accuracy and robustness, especially at aggressive subsampling rates (e.g., 20%)
- Computation is expensive on large datasets unless R is linked to an optimized BLAS library (ATLAS, MKL); Microsoft R Open is recommended on Windows
- sscClust requires the sscVis package to be installed first

## Evidence

- [abstract] SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage"
- [discussion] with Spearman correlations as the feature engineering and projecting methods: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
- [readme] data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11): "data("sce.Pollen")
sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)"
