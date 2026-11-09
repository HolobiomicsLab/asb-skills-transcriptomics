---
name: subsampling-based-clustering-strategy
description: Use when clustering large-scale scRNA-seq data (e.g., PBMC 68k) where
  naive full-data clustering is too slow or memory-heavy, and when subsampling bias
  must be mitigated to preserve or improve clustering accuracy.
when_to_use_negative:
- The dataset is small enough that full-data clustering (subsampling=F) is computationally
  feasible without accuracy loss
- A supervised classification or label-transfer task is needed rather than unsupervised
  de novo clustering
- The analysis requires reproducible clustering without randomness, since results
  are averaged over random subsample repeats
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: sscClust
  role: R implementation of the SSCC subsampling-clustering-classification framework,
    including ssc.run and Spearman-correlation feature-space projection
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: Baseline clustering algorithm wrapped in the subsampling framework and compared
    against SC3+SSCC on PBMC 68k
- name: SIMLR
  role: Clustering algorithm benchmarked under both SCC and SSCC strategies on the
    five scRNA-seq datasets
  repo: BatzoglouLabSU/SIMLR
- name: dropClust
  role: Third-party subsampling-based clustering method compared against SC3 and SC3+SSCC
    on PBMC 68k
- name: R
  role: Execution environment; sscClust requires R linked to an optimized BLAS library
    (ATLAS, MKL, or Microsoft R Open) for scalability
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:0003403f690c618944668bdec72b0a399efee307d59cc42f3427e1f49c2dcd59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SIMLR
  - SSCC
  - dropClust
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# subsampling-based-clustering-strategy

## Summary

A subsampling-clustering-classification strategy for scRNA-seq clustering in which clustering is run on a cell subsample and remaining cells are classified, reducing complexity from O(n2) to O(n); the SSCC variant performs subsampling-clustering in a Spearman-correlation feature space to remove subsampling bias and improve NMI accuracy.

## When to use

Use when clustering large-scale scRNA-seq data (e.g., PBMC 68k) where naive full-data clustering is too slow or memory-heavy, and when subsampling bias must be mitigated to preserve or improve clustering accuracy.

## When NOT to use

- The dataset is small enough that full-data clustering (subsampling=F) is computationally feasible without accuracy loss
- A supervised classification or label-transfer task is needed rather than unsupervised de novo clustering
- The analysis requires reproducible clustering without randomness, since results are averaged over random subsample repeats

## Inputs

- scRNA-seq gene expression count matrix with cell-type reference labels (e.g., PBMC 68k, Pollen, Kolodziejczyk, Usoskin, Zeisel datasets)
- Choice of base clustering algorithm (AP, k-means, k-medoids, SC3, SIMLR)
- Subsampling parameters: subsampling rate (sub.frac) or fixed subsample size (e.g., 5000 or 200 cells)
- Number of clusters (e.g., k.batch=11 for the Pollen dataset)

## Outputs

- Cluster assignments for all cells (subsampled clusters plus classified remaining cells)
- NMI values of inferred clusters vs. true labels averaged over ten random repeats
- Benchmark tables of per-dataset ΔNMI (SSCC vs SCC), runtime, and peak memory (e.g., nmi_benchmark_results.csv, pbmc68k_benchmark_results.csv)

## How to apply

In R with the sscClust package, load the scRNA-seq count matrix (e.g., as a SingleCellExperiment such as sce.Pollen) and run the subsampling-clustering-classification pipeline: cluster a random cell subsample at a chosen subsampling rate (e.g., sub.frac=0.8, or a fixed subsample size such as 5000 cells for SC3 default or 200 cells for SSCC), then classify the remaining cells. The SCC variant clusters in the original gene expression space; the SSCC variant instead builds a Spearman-correlation feature space from the subsample and clusters there, using random projection and feature engineering, which counters the subsampling biases that otherwise decrease accuracy and robustness. Wrap existing algorithms (AP, k-means, k-medoids, SC3, SIMLR) inside this framework and repeat over random subsamples (e.g., ten repeats). Evaluate with NMI against true cell-type labels, and record wall-clock runtime and peak memory for large datasets.

## Related tools

- **sscClust** (R implementation of the SSCC subsampling-clustering-classification framework, including ssc.run and Spearman-correlation feature-space projection) — https://github.com/Japrin/sscClust
- **SC3** (Baseline clustering algorithm wrapped in the subsampling framework and compared against SC3+SSCC on PBMC 68k)
- **SIMLR** (Clustering algorithm benchmarked under both SCC and SSCC strategies on the five scRNA-seq datasets) — BatzoglouLabSU/SIMLR
- **dropClust** (Third-party subsampling-based clustering method compared against SC3 and SC3+SSCC on PBMC 68k)
- **R** (Execution environment; sscClust requires R linked to an optimized BLAS library (ATLAS, MKL, or Microsoft R Open) for scalability)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- NMI values must lie in [0, 1], with ten random repeats recorded per dataset-algorithm-strategy-subsampling configuration
- Reproduced per-dataset ΔNMI ranges (SSCC vs SCC) must fall within reported ranges: 0.12–0.60 (Kolodziejczyk), 0.04–0.19 (Pollen), 0.14–0.37 (Usoskin), 0.02–0.28 (Zeisel), 0.10–0.28 (Zheng)
- On PBMC 68k, SC3+SSCC should reach ~0.59 NMI vs ~0.48 for default SC3 (~21% increase), with runtime near 2.2 min and peak memory near 3.7 GB
- Accuracy improvement over plain k-means can reach up to 3-fold; substantially smaller gains suggest the Spearman feature-space step was not applied
- Computational complexity should scale as O(n) rather than O(n2), observable via runtime and memory scaling on large datasets

## Limitations

- The plain SCC strategy introduces subsampling biases that may greatly decrease clustering accuracy and robustness even though it reduces complexity from O(n2) to O(n)
- Computation becomes expensive as datasets grow; R should be linked to an optimized BLAS library (ATLAS, MKL) and Microsoft R Open is recommended on Windows
- No version numbers for compared tools (SC3, Seurat, dropClust, bigScale, SCANPY) and no dataset accessions are stated in the extracted article sections

## Evidence

- [discussion] SSCC formulates a framework using Spearman correlations as feature engineering and projecting methods: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [discussion] SCC strategy reduces complexity to O(n) but subsampling biases hurt accuracy: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [readme] sscClust README describes Spearman-correlation feature space plus subsampling-classification: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient, clustering with subsampling and classfification which make it feasible to process"
- [readme] sscClust README shows concrete subsampling invocation: [evidence span withheld — anchor status: external_doc]
- [abstract] PBMC 68k benchmark result for SSCC vs SC3: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
