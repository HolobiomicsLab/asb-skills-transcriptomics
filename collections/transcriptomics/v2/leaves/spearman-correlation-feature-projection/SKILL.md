---
name: spearman-correlation-feature-projection
description: Use when clustering large-scale scRNA-seq data (e.g., tens of thousands
  of cells) with subsampling, and raw-expression clustering accuracy or 10%-vs-50%
  subsample consistency is poor; the goal is to cluster in a Spearman-correlation
  feature space instead.
when_to_use_negative:
- The dataset is small enough that full-data clustering without subsampling is tractable
  (ssc.run with subsampling=F) and feature projection is unnecessary
- The input is already a feature/reduced-dimension table rather than a raw scRNA-seq
  count matrix needing Spearman feature construction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: SSCC
  role: The framework that performs Spearman-correlation feature engineering and random
    projection before clustering and classification
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package implementing the ssc.run pipeline, Spearman-correlation feature
    projection (sscClust.iCor vignette), and subsampling-classification
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Execution environment for the sscClust implementation of SSCC
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:b769d46f3a3948e858ae48d3c9fc2aa988f2b4a5ea73b3389b6da9e386758deb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - CIDR
  - R
  - SC3
  - SIMLR
  - SSCC
  - Seurat
  - ZIFA
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spearman-correlation-feature-projection

## Summary

Projects single-cell RNA-seq data into a feature space built from Spearman correlations (feature engineering) before clustering and classification, the core step of the SSCC framework that reduces subsampling bias and computational cost versus clustering directly in raw gene expression space.

## When to use

Use when clustering large-scale scRNA-seq data (e.g., tens of thousands of cells) with subsampling, and raw-expression clustering accuracy or 10%-vs-50% subsample consistency is poor; the goal is to cluster in a Spearman-correlation feature space instead.

## When NOT to use

- The dataset is small enough that full-data clustering without subsampling is tractable (ssc.run with subsampling=F) and feature projection is unnecessary
- The input is already a feature/reduced-dimension table rather than a raw scRNA-seq count matrix needing Spearman feature construction

## Inputs

- Single-cell RNA-seq gene expression count matrix (e.g., R SingleCellExperiment such as sce.Pollen) with optional true cell-type labels
- Subsampling fraction (e.g., sub.frac = 0.8; benchmark rates 5%, 10%, 20%, 30%, 40%, 50%)

## Outputs

- Cluster assignment table for all cells (subsampled + classified), e.g., sscc_clusters.csv
- NMI values versus true labels and NMI consistency between subsampling rates (e.g., nmi_benchmark_results.csv, nmi_consistency_table.csv)
- Per-cell silhouette values of the Spearman-correlation features against true labels

## How to apply

After random cell subsampling (reducing complexity from O(n^2) to O(n)), compute Spearman correlations among the subsampled cells to construct engineered features rather than clustering raw gene expression. Cluster the subsampled cells in this Spearman feature space with the chosen base algorithm (e.g., k-means, SC3, SIMLR), then project/classify unsampled cells into the same feature space by assigning each to the cluster of its nearest subsampled cells. This is implemented in the sscClust R package (ssc.run with subsampling=T, sub.frac, e.g., 0.8, and Spearman-based feature construction in the sscClust.iCor workflow). Decision point: compare NMI and NMI consistency between subsampling rates against plain SCC; adopt the feature projection when SSCC improves ΔNMI and subsample-consistency (e.g., NMI consistency >0.5 on Kolodziejczyk, 0.60→0.69 on Macosko).

## Related tools

- **SSCC** (The framework that performs Spearman-correlation feature engineering and random projection before clustering and classification) — https://github.com/Japrin/sscClust
- **sscClust** (R package implementing the ssc.run pipeline, Spearman-correlation feature projection (sscClust.iCor vignette), and subsampling-classification) — https://github.com/Japrin/sscClust
- **R** (Execution environment for the sscClust implementation of SSCC)

## Examples

```
devtools::install_github("Japrin/sscClust"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- NMI of SSCC clustering versus true labels exceeds plain SCC, with ΔNMI within reported ranges (e.g., 0.12–0.60 on Kolodziejczyk, 0.10–0.28 on Zheng)
- NMI consistency between 10% and 50% subsamples is >0.5 for all five algorithms under SSCC (e.g., on Kolodziejczyk)
- Per-cell silhouette values of Spearman features have Pearson correlation near zero with the subsampling rate (10%–50%)
- On the 49,300-cell Macosko dataset with k-means + k-NN, average silhouette difference between 5% and 10% schemes drops to ~0.01 (from ~0.29) and NMI rises from ~0.60 to ~0.69
- All NMI values lie in [0, 1] across ten random repeats per configuration

## Limitations

- The SCC/SSCC subsampling strategy introduces subsampling biases; without feature projection accuracy and robustness decrease, so results depend on subsample representativeness
- Optimized BLAS (e.g., MKL, ATLAS; Microsoft R Open on Windows) is strongly recommended since computation becomes expensive for large datasets
- The article does not give explicit parameter/implementation details for how Spearman correlations are computed (missing from the discussion section)

## Evidence

- [abstract] Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [readme] sscClust README on Spearman feature space: "such as projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [intro] SSCC reduces computational complexity: "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [discussion] Spearman correlations as feature engineering: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] sscClust subsampling example invocation: [evidence span withheld — anchor status: external_doc]
