---
name: random-cell-subsampling
description: Use when clustering large-scale scRNA-seq count matrices (e.g., tens
  of thousands of cells such as the 68,578-cell human blood dataset) where running
  a base clustering algorithm on all pairwise cell relationships is computationally
  infeasible.
when_to_use_negative:
- The dataset is small enough to cluster all cells directly without the O(n^2) memory/time
  burden (sscClust's ssc.run supports subsampling=F)
- Clustering accuracy on known labels is paramount and no feature-construction step
  will be added — the SCC strategy alone 'suffers from biases introduced by subsampling
  which may greatly decrease the clustering accuracy and robustness'
- Input is not a cell-level expression matrix (e.g., already aggregated pseudobulk
  or a feature table)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SSCC
  role: The subsampling-clustering-classification framework being instantiated; sscClust
    is its R implementation
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package providing ssc.run with subsampling=T and sub.frac parameters to
    execute subsample-then-classify clustering
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Runtime environment; optimized BLAS (ATLAS, MKL, Microsoft R Open) is recommended
    because subsampled clustering remains expensive
- name: SC3
  role: Benchmark base clustering algorithm compared against the subsampling framework
    (20% accuracy improvement, 50-fold acceleration with SSCC)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:789a7a32fa3d0752f16fb15f453c78660ef54ccc3cd8075abc6fc94e1a724eb7
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# random-cell-subsampling

## Summary

A clustering strategy for large-scale scRNA-seq in which cells are randomly subsampled, the subset is clustered, and remaining cells are classified into those clusters, reducing computational complexity from O(n^2) to O(n). As implemented in the SSCC framework (sscClust), it is combined with Spearman-correlation feature construction to mitigate subsampling bias.

## When to use

Use when clustering large-scale scRNA-seq count matrices (e.g., tens of thousands of cells such as the 68,578-cell human blood dataset) where running a base clustering algorithm on all pairwise cell relationships is computationally infeasible.

## When NOT to use

- The dataset is small enough to cluster all cells directly without the O(n^2) memory/time burden (sscClust's ssc.run supports subsampling=F)
- Clustering accuracy on known labels is paramount and no feature-construction step will be added — the SCC strategy alone 'suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness'
- Input is not a cell-level expression matrix (e.g., already aggregated pseudobulk or a feature table)

## Inputs

- Single-cell RNA-seq count matrix (gene x cell expression matrix) loadable in R
- Choice of base clustering algorithm (e.g., SC3, k-means, SIMLR) to run on the subsample
- Subsample fraction (sscClust sub.frac parameter, e.g. 0.8)

## Outputs

- Cluster assignment table for all cells (subsampled plus classified), e.g. sscc_clusters.csv
- Subsampled-cell clustering in the Spearman-correlation feature space
- Nearest-neighbor classifications of unsampled cells into existing clusters

## How to apply

Load the raw scRNA-seq count data into R using the sscClust package's data-preparation utilities, then randomly subsample cells (in sscClust, controlled by the sub.frac parameter, e.g. 0.8 of cells) from the gene expression matrix so clustering runs on a manageable subset. Cluster the subsampled cells with a base clustering algorithm in the SSCC framework's constructed feature space — Spearman correlations serve as the feature engineering and projecting methods, which is what addresses the subsampling bias that plain SCC strategies suffer from. Classify each unsampled cell by projecting it into the same Spearman-correlation feature space and assigning it to the cluster of its nearest subsampled cells, then combine all cluster assignments. Judge success by cluster accuracy (ARI against known labels) and runtime/memory relative to running the base algorithm on the full dataset — SSCC reports 20% higher accuracy and 50-fold acceleration with 66% memory use versus SC3 on 68,578 cells.

## Related tools

- **SSCC** (The subsampling-clustering-classification framework being instantiated; sscClust is its R implementation) — https://github.com/Japrin/sscClust
- **sscClust** (R package providing ssc.run with subsampling=T and sub.frac parameters to execute subsample-then-classify clustering) — https://github.com/Japrin/sscClust
- **R** (Runtime environment; optimized BLAS (ATLAS, MKL, Microsoft R Open) is recommended because subsampled clustering remains expensive)
- **SC3** (Benchmark base clustering algorithm compared against the subsampling framework (20% accuracy improvement, 50-fold acceleration with SSCC))

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- All cells receive exactly one cluster assignment: subsampled cells from direct clustering plus unsampled cells from nearest-neighbor classification in the Spearman feature space
- Runtime scales linearly (O(n)) with cell count rather than quadratically; verify by benchmarking on increasing subsets
- Clustering accuracy (e.g., ARI against known labels) of the subsample-and-classify result is comparable to or better than clustering the full dataset with the base algorithm
- Memory consumption stays well below that of full-dataset clustering (SSCC used 66% of SC3's memory)
- Changing the random subsample should not drastically change cluster structure — Spearman feature construction guards against subsampling-bias-driven instability

## Limitations

- Subsampling alone introduces biases that can greatly decrease clustering accuracy and robustness; the Spearman-correlation feature engineering step is what mitigates this in SSCC
- Calculation is still expensive on large datasets; the sscClust README strongly recommends R linked to an optimized BLAS library (ATLAS, MKL) or Microsoft R Open
- sscClust depends on the additional package sscVis, which must be installed first

## Evidence

- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n): "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [abstract] SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [readme] clustering with subsampling and classfification which make it feasible to process thousands cells' data.: "clustering with subsampling and classfification which make it feasible to process thousands cells' data."
