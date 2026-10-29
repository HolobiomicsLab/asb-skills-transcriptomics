---
name: feature-space-clustering
description: Use when clustering a large single-cell RNA-seq expression matrix where
  direct clustering on raw gene expression is computationally prohibitive, and reduced
  O(n) complexity via subsampling plus Spearman-correlation feature projection is
  desired.
when_to_use_negative:
- The dataset is small enough that clustering directly on raw gene expression (or
  a plain SCC subsampling pipeline) is already tractable — the feature-projection
  layer is designed for large-scale data where O(n^2) complexity impedes scaling.
- The analysis requires clustering in raw gene-expression space rather than a correlation-based
  feature space, e.g. when per-gene expression structure must be preserved.
- Cells lack enough expressed genes for stable Spearman rank-correlation estimates
  (heavy dropout), since the projection relies on correlation as the feature.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SSCC
  role: The subsampling-clustering-classification framework whose Step 3 feature projection
    and Steps 4–5 (cluster subsampled cells, classify unsampled cells) define this
    skill
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R implementation of the SSCC pipeline, including Spearman-correlation projection,
    subsampling and classification utilities
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Runtime environment; an optimized BLAS-linked R build is recommended for large
    datasets
- name: SC3
  role: Benchmark baseline clustering algorithm that can serve as the base clusterer
    within the feature space
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:c37f3b350a453ba27099934baba56b12b31d47d12f9b007a9e45d5164b186c16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# feature-space-clustering

## Summary

Clusters large-scale scRNA-seq data by first projecting cells into a Spearman-correlation feature space and then clustering the subsampled cells there, using classification to assign all unsampled cells — the feature-projection step (Figure 1B Step 3) that SSCC adds on top of the SCC subsampling-clustering-classification strategy.

## When to use

Use when clustering a large single-cell RNA-seq expression matrix where direct clustering on raw gene expression is computationally prohibitive, and reduced O(n) complexity via subsampling plus Spearman-correlation feature projection is desired.

## When NOT to use

- The dataset is small enough that clustering directly on raw gene expression (or a plain SCC subsampling pipeline) is already tractable — the feature-projection layer is designed for large-scale data where O(n^2) complexity impedes scaling.
- The analysis requires clustering in raw gene-expression space rather than a correlation-based feature space, e.g. when per-gene expression structure must be preserved.
- Cells lack enough expressed genes for stable Spearman rank-correlation estimates (heavy dropout), since the projection relies on correlation as the feature.

## Inputs

- Single-cell RNA-seq gene expression/count matrix (R object, e.g. SingleCellExperiment such as sce.Pollen from sscClust)
- Choice of base clustering algorithm (e.g., SC3, k-means)
- Subsampling fraction (sub.frac, e.g. 0.8) and expected number of clusters (k.batch)

## Outputs

- Cluster assignment for all cells (subsampled + classified unsampled cells), e.g. written to sscc_clusters.csv
- Spearman-correlation feature-space representation usable for visualization and clustering

## How to apply

Load the scRNA-seq count matrix in R (sscClust / sscVis) and randomly subsample cells so clustering complexity drops from O(n^2) to O(n). Compute Spearman correlations among the subsampled cells to construct a feature space — this is the feature-engineering/projection step that replaces clustering directly on raw expression as plain SCC would. Run the chosen base clustering algorithm (e.g., SC3, k-means) on the subsampled cells inside this Spearman feature space; the article reports SSCC achieves 20% higher accuracy and 50-fold acceleration vs SC3, and up to 3-fold improvement over k-means. Project/classify every unsampled cell into the same Spearman feature space and assign it to the cluster of its nearest subsampled cells. Combine subsampled and classified assignments into the final cluster table; use ssc.run with subsampling=T and a sub.frac parameter (e.g., 0.8 in the README demo) to execute the pipeline. Rationale: Spearman correlations act as both feature engineering and projecting methods, improving accuracy and robustness over SCC's subsampling biases.

## Related tools

- **SSCC** (The subsampling-clustering-classification framework whose Step 3 feature projection and Steps 4–5 (cluster subsampled cells, classify unsampled cells) define this skill) — https://github.com/Japrin/sscClust
- **sscClust** (R implementation of the SSCC pipeline, including Spearman-correlation projection, subsampling and classification utilities) — https://github.com/Japrin/sscClust
- **R** (Runtime environment; an optimized BLAS-linked R build is recommended for large datasets)
- **SC3** (Benchmark baseline clustering algorithm that can serve as the base clusterer within the feature space)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- The clustering output covers every cell: each unsampled cell is assigned via classification to the cluster of its nearest subsampled cells in the Spearman feature space.
- Complexity behaves as O(n), not O(n^2): runtime should scale roughly linearly with total cell count (SSCC showed 50-fold acceleration over SC3 on 68,578 human blood cells).
- Clustering accuracy in the Spearman feature space exceeds the same base algorithm applied directly (e.g., ~20% improvement over SC3, up to 3-fold over k-means in the paper's benchmarks).
- Reproducibility: ssc.run with subsampling=T and a fixed sub.frac (e.g., 0.8) yields a complete cluster assignment table comparable to the full-data run (subsampling=F).
- Memory footprint should stay well below full pairwise approaches (SSCC used only 66% of SC3's memory in the benchmark).

## Limitations

- Subsampling introduces biases that can decrease clustering accuracy and robustness; the Spearman feature projection is designed to mitigate, not eliminate, this SCC weakness.
- Implementation details such as how Spearman correlations are computed are not fully specified in the article sections provided; consult the sscClust vignettes.
- Performance is demonstrated on real scRNA-seq datasets covering a wide range of technologies, sequencing depths, and organisms, but no dataset accessions or tool version numbers are given, complicating exact replication.

## Evidence

- [abstract] SSCC proposal with feature construction: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [discussion] Spearman correlations as feature engineering: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [intro] Complexity reduction O(n2) to O(n): "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [abstract] Benchmark performance vs SC3: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [readme] README on spearman feature projection: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
