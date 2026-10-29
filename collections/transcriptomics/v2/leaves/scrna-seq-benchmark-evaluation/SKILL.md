---
name: scrna-seq-benchmark-evaluation
description: Use when testing whether engineered clustering features (e.g., Spearman-correlation
  features from SSCC) stay consistent with true cell-type labels across fluctuating
  subsampling rates of a benchmark scRNA-seq dataset.
when_to_use_negative:
- No true cell-type labels are available, since silhouette values must be computed
  against ground-truth labels
- The analysis goal is speed or accuracy benchmarking against SC3/k-means rather than
  robustness of features to subsampling rate
- Input is a small dataset where clustering on all cells is feasible without any subsampling,
  making subsampling-robustness moot
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: sscClust
  role: R implementation of SSCC; constructs Spearman-correlation features, runs subsampling-clustering-classification,
    and supports configurable sub.frac rates
  repo: https://github.com/Japrin/sscClust
- name: SSCC
  role: The clustering framework (Spearman subsampling-clustering-classification)
    whose feature robustness to subsampling rate is being evaluated
  repo: https://github.com/Japrin/sscClust
- name: SIMLR
  role: Alternative scRNA-seq clustering method mentioned as a comparison point for
    large-scale single-cell clustering
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SC3
  role: Widely used clustering package used as the accuracy/memory/runtime baseline
    for SSCC
- name: R
  role: Implementation language for the sscClust package and the silhouette/subsampling
    benchmark scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:eab02d23f6db9452973f96c524a1bea9f2ee87c9b44e055a64a7365d841d3aa3
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-seq-benchmark-evaluation

## Summary

Benchmark-evaluate scRNA-seq clustering features by measuring whether per-cell silhouette values against true cell-type labels are robust to cell-subsampling rate, using Spearman-correlation features built with sscClust (SSCC).

## When to use

Use when testing whether engineered clustering features (e.g., Spearman-correlation features from SSCC) stay consistent with true cell-type labels across fluctuating subsampling rates of a benchmark scRNA-seq dataset.

## When NOT to use

- No true cell-type labels are available, since silhouette values must be computed against ground-truth labels
- The analysis goal is speed or accuracy benchmarking against SC3/k-means rather than robustness of features to subsampling rate
- Input is a small dataset where clustering on all cells is feasible without any subsampling, making subsampling-robustness moot

## Inputs

- scRNA-seq count matrices for benchmark datasets (e.g., sce.Pollen SingleCellExperiment objects)
- true cell-type labels for each dataset
- list of cell-subsampling rates (10%, 20%, 30%, 40%, 50%)

## Outputs

- Per-cell silhouette values computed against true labels for each subsample and rate
- Scatter plots of silhouette value (X) versus subsampling rate (Y), one per dataset
- Pearson correlation coefficient between silhouette values and subsampling rates per dataset

## How to apply

Load the benchmark scRNA-seq count matrices together with their true cell-type labels, then generate subsamples at five cell-subsampling rates (10%, 20%, 30%, 40%, 50%). For each subsample, construct Spearman-correlation features with the sscClust R package (which projects data into a feature space using Spearman correlation), cluster the subsample, and compute per-cell silhouette values against the true labels. Plot silhouette value versus subsampling rate as scatter plots, one per benchmark dataset (e.g., sce.Pollen), and compute the Pearson correlation coefficient for each. Accept the result when the Pearson correlation between silhouette value and subsampling rate is near zero, indicating the engineered features remain consistent with the true cluster labels regardless of the fraction of cells subsampled. Rationale: the SCC (subsampling-clustering-classification) strategy can introduce subsampling biases, and this test directly quantifies that robustness.

## Related tools

- **sscClust** (R implementation of SSCC; constructs Spearman-correlation features, runs subsampling-clustering-classification, and supports configurable sub.frac rates) — https://github.com/Japrin/sscClust
- **SSCC** (The clustering framework (Spearman subsampling-clustering-classification) whose feature robustness to subsampling rate is being evaluated) — https://github.com/Japrin/sscClust
- **SIMLR** (Alternative scRNA-seq clustering method mentioned as a comparison point for large-scale single-cell clustering) — https://github.com/BatzoglouLabSU/SIMLR
- **SC3** (Widely used clustering package used as the accuracy/memory/runtime baseline for SSCC)
- **R** (Implementation language for the sscClust package and the silhouette/subsampling benchmark scripts)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Pearson correlation coefficient between silhouette values and subsampling rate is near zero for every benchmark dataset
- Per-cell silhouette values of Spearman-correlation features are almost unchanged across the 10%–50% subsampling rates
- The sscClust pipeline runs at the specified sub.frac values (0.1–0.5) without error on each benchmark SingleCellExperiment object
- Clusters from each subsample remain consistent with the true cell-type labels (silhouette against true labels stays high)

## Limitations

- The SCC (subsampling-clustering-classification) strategy suffers from biases introduced by subsampling, which may decrease clustering accuracy and robustness if features are not verified as in this workflow
- Computation becomes expensive as datasets grow; the README recommends R linked to optimized BLAS (ATLAS, MKL) or Microsoft R Open
- Requires the sscVis package alongside sscClust, and no dataset accessions or version numbers for compared tools are stated in the article

## Evidence

- [abstract] Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [intro] SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering: "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
- [readme] clustering with subsampling and classfification which make it feasible to process thousands cells' data.: "clustering with subsampling and classfification which make it feasible to process thousands cells' data."
- [abstract] An R implementation of SSCC is available at https://github.com/Japrin/sscClust: "An R implementation of SSCC is available at https://github.com/Japrin/sscClust"
