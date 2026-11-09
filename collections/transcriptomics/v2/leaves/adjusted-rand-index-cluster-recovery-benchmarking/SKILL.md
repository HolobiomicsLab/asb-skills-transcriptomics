---
name: adjusted-rand-index-cluster-recovery-benchmarking
description: Use when comparing dimension-reduction or clustering methods on scRNAseq
  data that have published ground-truth cell-type annotations, and you need a single
  reproducible metric (ARI) to rank cluster recovery across datasets.
when_to_use_negative:
- The dataset lacks ground-truth cell-type annotations or the annotations were not
  independently validated — ARI then measures agreement with a possibly narrow 'reference'
  rather than true recovery
- The method's advantages lie in resolving sub-populations or rare cell types that
  the reference labels do not distinguish; the article notes ARI 'would actually receive
  a small penalty for differences from "reference"' in such cases
- The benchmarking goal is complex-tissue cell-type discovery — the article notes
  high-quality ground-truth datasets are limited to simple pseudo-cell mixtures or
  pools of distinct cell types and 'do not reflect real-world tissue complexity'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: R/Bioconductor package used to compute CA embeddings on raw counts (SVD of
    Pearson chi-squared residuals) as one arm of the ARI benchmark
  repo: github.com/laurenhsu1/corral
- name: R
  role: Analysis language in which the benchmark pipeline (embedding, NNGraph clustering,
    ARI computation) is executed
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral interfaces with
    for loading datasets and computing embeddings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:357713ee3147dc0ae54b0395847fd220272cee9c583c7c8b5d25aa5b9040c835
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# adjusted-rand-index-cluster-recovery-benchmarking

## Summary

Benchmark dimension-reduction methods (e.g., CA vs PCA) for single-cell RNA-seq by clustering cells in the reduced embeddings and scoring cluster assignments against published ground-truth cell-type labels with Adjusted Rand Index (ARI), as done across the SCMixology and Zhengmix benchmark datasets.

## When to use

Use when comparing dimension-reduction or clustering methods on scRNAseq data that have published ground-truth cell-type annotations, and you need a single reproducible metric (ARI) to rank cluster recovery across datasets.

## When NOT to use

- The dataset lacks ground-truth cell-type annotations or the annotations were not independently validated — ARI then measures agreement with a possibly narrow 'reference' rather than true recovery
- The method's advantages lie in resolving sub-populations or rare cell types that the reference labels do not distinguish; the article notes ARI 'would actually receive a small penalty for differences from "reference"' in such cases
- The benchmarking goal is complex-tissue cell-type discovery — the article notes high-quality ground-truth datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types and 'do not reflect real-world tissue complexity'

## Inputs

- scRNAseq count matrix with published ground-truth cell-type labels per cell (SingleCellExperiment or equivalent), e.g. SCMixology and Zhengmix benchmark datasets
- Cell embeddings from each dimension-reduction method under comparison (e.g., CA embeddings from corral on counts; PCA embeddings on log(x + 1) logcounts)

## Outputs

- Per-dataset ARI values comparing cluster assignments to ground-truth labels for each method
- Compiled ARI comparison table across datasets (e.g., ari_comparison.csv) showing whether the candidate method matches or exceeds the baseline

## How to apply

Load benchmark scRNAseq datasets with published ground-truth labels (in the Hsu & Culhane 2023 study: three SCMixology cell-mixture datasets from CellBench and three Zhengmix datasets from DuoClustering2018). For each method under test, compute cell embeddings (e.g., CA on raw counts via corral, which does SVD of the Pearson chi-squared residuals, or PCA on log(x + 1) logcounts as the typical baseline). Cluster cells in the reduced-dimension embeddings using NNGraph clustering, keeping the clustering step fixed so only the dimension-reduction method varies. Compute the Adjusted Rand Index of each method's cluster assignments against the annotated cell-type labels and compile a comparison table (e.g., ari_comparison.csv) across all datasets. Judge success by whether the candidate method matches or exceeds the baseline ARI in a majority of paired comparisons; in the source study CA matched or outperformed PCA in every counts and logcounts comparison.

## Related tools

- **corral** (R/Bioconductor package used to compute CA embeddings on raw counts (SVD of Pearson chi-squared residuals) as one arm of the ARI benchmark) — github.com/laurenhsu1/corral
- **R** (Analysis language in which the benchmark pipeline (embedding, NNGraph clustering, ARI computation) is executed)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral interfaces with for loading datasets and computing embeddings)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- corral(sce, whichmat = "counts"); mbluster::clusterCells(sce, ...); mclust::adjustedRandIndex(clusters, sce$cell_type)
```

## Evaluation signals

- ARI is computed for every dataset x method pair (CA on counts, CA on logcounts, PCA on logcounts) against the annotated cell-type labels — no missing cells in the comparison table
- Candidate method matches or exceeds the baseline PCA ARI in paired comparisons; in the source study CA outperformed or matched PCA in every counts and logcounts NNGraph comparison
- Clustering method (NNGraph) and its parameters are held constant across methods so ARI differences are attributable to the dimension-reduction embeddings
- Ground-truth labels match the original published cell-type annotations for SCMixology (CellBench) and Zhengmix (DuoClustering2018) datasets
- Comparison table (ari_comparison.csv) compiles ARI values for all six benchmark datasets in a single reproducible artifact

## Limitations

- Ground-truth labels come from original dataset publications and were not independently validated, so performance is limited by the original cell type assignments
- ARI may penalize methods that better distinguish sub-populations or rare cell types relative to the 'reference' labels
- High-quality ground-truth benchmarking datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types (SCMixology, Zhengmix) and do not reflect real-world tissue complexity
- Complex-tissue benchmarks assign cell identities with a single analytical method and no independent validation, giving an overly narrow benchmark standard; the number of true clusters is typically unknown a priori
- No tool or package versions for corral or dependencies are reported; no accession IDs or links for the benchmark datasets are provided

## Evidence

- [results] CA outperformed or matched PCA in every comparison of the counts and logcounts pipelines, as assessed by Adjusted Rand Index against annotated cell types: [evidence span withheld — anchor status: not_found]
- [intro] CA via SVD of Pearson residuals vs PCA on logcounts: "most single cell workflows begin with a log(x + 1) transformation of the counts matrix, and then use PCA to decompose the resulting “logcounts” data"
- [discussion] ARI penalizes deviations from reference labels: "these advantages may not be reflected in the ARI, and the method would actually receive a small penalty for differences from “reference.”"
- [discussion] Benchmark datasets are simple mixtures, not complex tissues: "most high-quality, “ground-truth” benchmarking datasets are derived from simple “pseudo”-cell mixtures, or from pools of distinct cell types"
- [readme] corral is a Bioconductor package for CA on single-cell count data: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
