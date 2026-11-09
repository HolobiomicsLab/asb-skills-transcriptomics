---
name: roc-auc-computation-gene-ranking
description: Use when comparing accuracy and runtime of DEG prediction methods on
  artificial scRNA-seq datasets with known ground-truth DEGs, where genes must be
  ranked by method-specific significance scores and evaluated via ROC AUC.
when_to_use_negative:
- Input is real experimental data without known ground-truth DEGs — ROC AUC against
  planted labels is only meaningful on artificial datasets with known true DEGs
- 'The method under test requires continuous expression values: singleCellHaystack''s
  implementation at this time requires binary detection data as input, so raw continuous
  counts are not directly supported'
- Goal is differential expression on one dataset rather than method comparison — running
  the full multi-method benchmark is unnecessary
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ROCR
  role: Computes ROC curve AUC from method significance-score gene rankings using
    planted DEGs as true labels
- name: singleCellHaystack
  role: Benchmarked clustering-independent DEG prediction method; run via haystack()
    on cell coordinates and binary detection data
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Comparison method run as FindAllMarkers both with and without default filtering
  repo: https://github.com/satijalab/seurat
- name: DEsingle
  role: Cluster-based comparison method included in the runtime/accuracy benchmark
- name: scDD
  role: Comparison method; noted to fail on the largest artificial datasets
- name: edgeR
  role: Bulk RNA-seq method included as comparison in the benchmark
- name: monocle 2
  role: Comparison method included in the benchmark
- name: EMDomics
  role: Comparison method included in the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:21ca2e24760f4609dc0bc453e4c2da6bc31087c45efa164430f22373dbbbbb10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - DEsingle
  - EMDomics
  - R
  - ROCR
  - Seurat
  - edgeR
  - monocle 2
  - scDD
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# roc-auc-computation-gene-ranking

## Summary

Benchmark DEG prediction methods (singleCellHaystack vs DEsingle, EMDomics, scDD, edgeR, monocle 2, Seurat FindAllMarkers) on artificial single-cell datasets with planted differentially expressed genes, ranking genes by each method's significance score and computing the AUC of the DEG detection ROC curve with ROCR.

## When to use

Use when comparing accuracy and runtime of DEG prediction methods on artificial scRNA-seq datasets with known ground-truth DEGs, where genes must be ranked by method-specific significance scores and evaluated via ROC AUC.

## When NOT to use

- Input is real experimental data without known ground-truth DEGs — ROC AUC against planted labels is only meaningful on artificial datasets with known true DEGs
- The method under test requires continuous expression values: singleCellHaystack's implementation at this time requires binary detection data as input, so raw continuous counts are not directly supported
- Goal is differential expression on one dataset rather than method comparison — running the full multi-method benchmark is unnecessary

## Inputs

- Artificial scRNA-seq count matrices with planted DEGs in non-randomly positioned cell subsets, at multiple cell sizes (1000–10,000 cells)
- Cell coordinates in 2D or multidimensional space (e.g., t-SNE, PCA, UMAP embeddings)
- Binary gene detection matrices (gene detected/not detected per cell)
- Known ground-truth DEG labels for the planted genes

## Outputs

- Per-method gene rankings by method-specific significance scores
- AUC of the DEG detection ROC curve per method per dataset size (values in [0,1], computed with ROCR)
- Median runtime per method per dataset size
- Aggregated summary table benchmark_median_auc_runtime.csv with columns: method, dataset size, median AUC, median runtime

## How to apply

Generate artificial scRNA-seq datasets containing known DEGs in non-randomly positioned cell subsets at multiple cell numbers (e.g., 1000 to 10,000 cells). Run singleCellHaystack via its R interface using haystack() with cell coordinates and binary detection data (the method requires binary input, i.e., a gene is detected or not in each cell), and run each comparison method (DEsingle, EMDomics, scDD, edgeR, monocle 2, Seurat FindAllMarkers with and without default filtering) on the same datasets, recording elapsed runtime and noting scDD failures on the largest sets. For each method-dataset combination, rank genes by method-specific significance scores and compute the AUC of the DEG detection ROC curve with ROCR, treating the planted DEGs as true labels. Aggregate per-method results by dataset size, computing median AUC and median runtime across replicates, and write to a summary table (method, dataset size, median AUC, median runtime) such as benchmark_median_auc_runtime.csv. Verify that singleCellHaystack attains the highest AUC except at the smallest size (1000 cells) and completes in roughly 5 minutes on 10,000-cell datasets.

## Related tools

- **ROCR** (Computes ROC curve AUC from method significance-score gene rankings using planted DEGs as true labels)
- **singleCellHaystack** (Benchmarked clustering-independent DEG prediction method; run via haystack() on cell coordinates and binary detection data) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Comparison method run as FindAllMarkers both with and without default filtering) — https://github.com/satijalab/seurat
- **DEsingle** (Cluster-based comparison method included in the runtime/accuracy benchmark)
- **scDD** (Comparison method; noted to fail on the largest artificial datasets)
- **edgeR** (Bulk RNA-seq method included as comparison in the benchmark)
- **monocle 2** (Comparison method included in the benchmark)
- **EMDomics** (Comparison method included in the benchmark)

## Examples

```
library(ROCR); pred <- prediction(scores, labels); auc <- performance(pred, "auc")@y.values[[1]]
```

## Evaluation signals

- The output CSV contains exactly one row per method per dataset size, with finite median AUC in [0,1] and a finite median runtime
- The tabulated AUC ordering reproduces the finding that singleCellHaystack attains the highest accuracy except at the smallest dataset size (1000 cells)
- singleCellHaystack median runtime is roughly 5 minutes on 10,000-cell datasets, and cluster-based methods (DEsingle, scDD) show far longer runtimes
- scDD failures on the largest datasets are recorded rather than silently dropped
- Per-method rankings and AUCs are reproducible when re-running the same artificial datasets with the same planted DEG labels

## Limitations

- singleCellHaystack requires binary detection data as input; continuous expression measures such as normalized read counts are not supported yet, and a hard detection threshold is a weak point — genes with different expression levels but the same detection pattern get identical D_KL and p-value
- P-values can be inflated by double use of gene expression data (for defining clusters/embeddings and for DEG prediction); singleCellHaystack suffers from the same issue because input coordinates capture a large proportion of the variability
- Many zero counts reflect dropout events due to lack of sensitivity; imputation could address this but was not applied

## Evidence

- [abstract] singleCellHaystack has higher accuracy than existing DEG prediction approaches on artificial datasets: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [readme] KL-divergence basis of the benchmarked method: "singleCellHaystack uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space"
- [methods] R interface invocation of haystack: "res <- haystack(dat.tsne, dat.expression)"
- [methods] gene filtering threshold used before analysis: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] binary detection requirement for the method: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
