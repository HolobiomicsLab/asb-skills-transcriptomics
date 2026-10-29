---
name: subsampling-clustering-classification-pipeline
description: Use when clustering large scRNA-seq datasets (e.g., tens of thousands
  of cells like the 49,300-cell Macosko retina matrix) where full O(n^2) clustering
  is infeasible, or when clustering consistency across subsampling ratios (e.g., 5%
  vs 10%) must be assessed.
when_to_use_negative:
- The dataset is small enough for full (non-subsampled) clustering, where ssc.run(subsampling=FALSE)
  applies and subsampling bias need not be managed
- The task requires supervised cell-type annotation from known labels rather than
  unsupervised clustering of a subsample with k-NN label propagation
- A clustering algorithm incompatible with the subsampling-classification decomposition
  is required (the paper benchmarks general algorithms, but SSCC's Spearman feature
  engineering is designed for its own framework)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SSCC
  role: 'The core Spearman subsampling-clustering-classification framework being executed:
    Spearman-correlation feature engineering plus random projection added to the SCC
    pipeline'
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language; sscClust is an R package and R should be linked to
    optimized BLAS (ATLAS, MKL; Microsoft R Open on Windows) for large datasets
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: Baseline SCC-strategy clustering method against which SSCC accuracy/speed/memory
    (20% accuracy improvement, 50-fold acceleration, 66% memory usage) is benchmarked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:34d8b3e87108b0db27e0811dc1c782dd9e17703bfec90cfc1ff45c591877a227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SSCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# subsampling-clustering-classification-pipeline

## Summary

A two-stage (SCC/SSCC) clustering pipeline for large-scale scRNA-seq data in which a random cell subsample is clustered (e.g., k-means) and remaining cells are assigned labels by k-NN classification; the SSCC variant adds Spearman-correlation-based feature engineering and random projection to reduce subsampling bias and improve accuracy, robustness, and consistency.

## When to use

Use when clustering large scRNA-seq datasets (e.g., tens of thousands of cells like the 49,300-cell Macosko retina matrix) where full O(n^2) clustering is infeasible, or when clustering consistency across subsampling ratios (e.g., 5% vs 10%) must be assessed.

## When NOT to use

- The dataset is small enough for full (non-subsampled) clustering, where ssc.run(subsampling=FALSE) applies and subsampling bias need not be managed
- The task requires supervised cell-type annotation from known labels rather than unsupervised clustering of a subsample with k-NN label propagation
- A clustering algorithm incompatible with the subsampling-classification decomposition is required (the paper benchmarks general algorithms, but SSCC's Spearman feature engineering is designed for its own framework)

## Inputs

- Large-scale scRNA-seq expression matrix (e.g., 49,300-cell Macosko retina dataset; 68,578 human blood cells in the paper's benchmark)
- Subsampling ratio(s) to compare (e.g., 5% and 10% of cells)
- Number of clusters parameter (e.g., k.batch passed to ssc.run)
- SingleCellExperiment-style input object (e.g., sce.Pollen demo data)

## Outputs

- Cluster labels for the subsample (k-means) and classified labels for remaining cells (k-NN)
- Consistency metrics between subsampling schemes: silhouette difference, NMI (e.g., 0.60 SCC vs 0.69 SSCC), Pearson correlation of per-cell silhouette values (e.g., 0.47 SCC vs 0.58 SSCC)
- Benchmark results table (e.g., scc_sscc_macosko_benchmark.csv)
- Reduced-dimension feature-space representation via Spearman correlation projection and random projection

## How to apply

Load the scRNA-seq expression matrix (e.g., the 49,300-cell Macosko retina dataset) and preprocess it within the sscClust R framework. For each subsampling fraction (e.g., 5% and 10%), run SCC: k-means clustering on the random cell subsample, then k-NN classification to propagate labels to the remaining cells. Optionally run the full SSCC variant, which formulates the framework using Spearman correlations as the feature engineering and projecting methods (projecting data to a Spearman-correlation feature space) together with random projection before clustering and classification, reducing complexity from O(n^2) to O(n) while mitigating subsampling bias. Compare subsampling schemes with silhouette difference, NMI between scheme labelings, and Pearson correlation of per-cell silhouette values; in the Macosko benchmark SSCC reduced the average silhouette difference from ~0.29 to ~0.01 and raised NMI from 0.60 to 0.69 (silhouette correlation ~0.47 to ~0.58). Use ssc.run() with subsampling=TRUE and sub.frac set to the desired fraction, and link R to an optimized BLAS (ATLAS/MKL) since computation becomes expensive on large datasets.

## Related tools

- **SSCC** (The core Spearman subsampling-clustering-classification framework being executed: Spearman-correlation feature engineering plus random projection added to the SCC pipeline) — https://github.com/Japrin/sscClust
- **R** (Implementation language; sscClust is an R package and R should be linked to optimized BLAS (ATLAS, MKL; Microsoft R Open on Windows) for large datasets) — https://github.com/Japrin/sscClust
- **SC3** (Baseline SCC-strategy clustering method against which SSCC accuracy/speed/memory (20% accuracy improvement, 50-fold acceleration, 66% memory usage) is benchmarked)

## Examples

```
devtools::install_github("Japrin/sscVis"); devtools::install_github("Japrin/sscClust"); sce.sub <- ssc.run(sce, subsampling=T, sub.frac=0.05, k.batch=11)
```

## Evaluation signals

- Average silhouette difference between the 5% and 10% subsampling schemes drops to ~0.01 with SSCC versus ~0.29 with plain SCC on the Macosko dataset
- NMI between the two subsampling schemes rises from ~0.60 (SCC) to ~0.69 (SSCC)
- Pearson correlation of per-cell silhouette values between schemes increases from ~0.47 (SCC) to ~0.58 (SSCC)
- Clustering accuracy matches or exceeds the SC3 baseline (~20% improvement reported) while runtime scales approximately linearly (O(n) rather than O(n^2))
- All metrics are written to a results file (e.g., scc_sscc_macosko_benchmark.csv) with the expected metric columns populated

## Limitations

- The SCC strategy suffers from biases introduced by subsampling which may greatly decrease clustering accuracy and robustness, although it reduces complexity from O(n^2) to O(n)
- Computation becomes expensive as datasets grow; the README strongly recommends R linked to an optimized BLAS library (ATLAS, MKL; Microsoft R Open on Windows)
- The sscClust package depends on the companion sscVis package, which must be installed first
- No parameter settings or implementation details for how Spearman correlations are computed are given in some article sections, and no dataset accessions are provided for the evaluation datasets

## Evidence

- [discussion] SSCC framework definition via Spearman features: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [discussion] SCC subsampling bias limitation: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
- [intro] Complexity reduction claim: "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [readme] README subsampling invocation with sub.frac parameter: [evidence span withheld — anchor status: external_doc]
- [readme] README BLAS recommendation for large datasets: "we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc"
