---
name: method-runtime-profiling
description: Use when you need to quantify how a DEG prediction method's detection
  accuracy and computational cost scale with dataset size, using artificial single-cell
  datasets with known planted DEGs as ground truth.
when_to_use_negative:
- The analysis goal is DEG discovery on a real biological dataset rather than method
  benchmarking against known ground-truth labels.
- The input data lacks planted or otherwise known true DEG labels, so ROC AUC of DEG
  detection cannot be computed.
- Only one method and one dataset size are of interest — runtime/accuracy scaling
  comparisons across methods are the point of this skill.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Method under benchmark; DEG prediction without clustering using Kullback-Leibler
    divergence, run via haystack() in R
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Comparison DEG method via FindAllMarkers, run both with and without its default
    filtering; also used to prepare embeddings
  repo: https://github.com/satijalab/seurat
- name: monocle 2
  role: Comparison DEG prediction method in the runtime/AUC benchmark
- name: DEsingle
  role: Cluster-based comparison DEG method with far longer runtimes on artificial
    datasets
- name: scDD
  role: Comparison DEG method; fails on the largest artificial datasets
- name: edgeR
  role: Bulk RNA-seq-derived comparison DEG method included in the benchmark
- name: EMDomics
  role: Comparison DEG prediction method in the benchmark
- name: ROCR
  role: R package used to compute AUC of the DEG detection ROC curve from method-specific
    gene rankings
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:70a3b36435d7ac607814d9a821d7894c72f89deb0fcd077b6843bb98b09b609e
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# method-runtime-profiling

## Summary

Benchmark the accuracy (AUC of DEG detection ROC) and elapsed runtime of singleCellHaystack against competing DEG prediction methods (DEsingle, EMDomics, scDD, edgeR, monocle 2, Seurat FindAllMarkers) on artificial scRNA-seq datasets of increasing size (1,000–10,000 cells), aggregating medians per method and dataset size.

## When to use

Use when you need to quantify how a DEG prediction method's detection accuracy and computational cost scale with dataset size, using artificial single-cell datasets with known planted DEGs as ground truth.

## When NOT to use

- The analysis goal is DEG discovery on a real biological dataset rather than method benchmarking against known ground-truth labels.
- The input data lacks planted or otherwise known true DEG labels, so ROC AUC of DEG detection cannot be computed.
- Only one method and one dataset size are of interest — runtime/accuracy scaling comparisons across methods are the point of this skill.

## Inputs

- Artificial scRNA-seq datasets with known planted DEGs in non-randomly positioned cell subsets, at multiple cell numbers (1,000–10,000 cells)
- Cell coordinates (2D embedding such as t-SNE, or PCA coordinates) for haystack() input
- Binary detection / expression matrix of genes by cells

## Outputs

- Per-method, per-dataset-size ROC AUC values computed with ROCR against planted DEG labels
- Per-method, per-dataset-size elapsed runtimes (with failure notes, e.g., scDD on the largest sets)
- Aggregated summary table benchmark_median_auc_runtime.csv with columns method, dataset size, median AUC, median runtime

## How to apply

Generate artificial scRNA-seq datasets containing known DEGs in non-randomly positioned cell subsets at several cell numbers (e.g., 1,000 up to 10,000 cells). Run singleCellHaystack via its R interface, calling haystack() on cell coordinates (e.g., t-SNE or PCA space) with binary detection/expression input, and record elapsed runtime; run each comparison method (DEsingle, EMDomics, scDD, edgeR, monocle 2, Seurat FindAllMarkers with and without default filtering) on the same inputs, also recording runtime and failures (scDD fails on the largest sets). For every method–dataset combination, rank genes by the method's significance scores and compute the AUC of the DEG-detection ROC curve with ROCR, treating the planted DEGs as true labels. Aggregate per-method results by dataset size into median AUC and median runtime across replicates, write the summary table (method, dataset size, median AUC, median runtime) to benchmark_median_auc_runtime.csv, and check that expected trends hold: haystack achieves the highest AUC except at 1,000 cells and completes in roughly 5 minutes on 10,000-cell datasets.

## Related tools

- **singleCellHaystack** (Method under benchmark; DEG prediction without clustering using Kullback-Leibler divergence, run via haystack() in R) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Comparison DEG method via FindAllMarkers, run both with and without its default filtering; also used to prepare embeddings) — https://github.com/satijalab/seurat
- **monocle 2** (Comparison DEG prediction method in the runtime/AUC benchmark)
- **DEsingle** (Cluster-based comparison DEG method with far longer runtimes on artificial datasets)
- **scDD** (Comparison DEG method; fails on the largest artificial datasets)
- **edgeR** (Bulk RNA-seq-derived comparison DEG method included in the benchmark)
- **EMDomics** (Comparison DEG prediction method in the benchmark)
- **ROCR** (R package used to compute AUC of the DEG detection ROC curve from method-specific gene rankings)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- benchmark_median_auc_runtime.csv contains exactly one row per method per dataset size, with finite median AUC values in [0,1] and finite median runtimes.
- AUC ordering reproduces the reported finding: singleCellHaystack attains the highest accuracy except at the smallest size (1,000 cells).
- Runtime column reproduces the reported trend: haystack finishes in roughly 5 minutes on 10,000-cell datasets, while cluster-based methods like DEsingle and scDD take far longer.
- Failure cases are explicitly recorded (e.g., scDD failing on the largest artificial datasets) rather than silently dropped.
- Per-method AUCs are computed against the same planted-DEG ground truth using ROCR, ensuring cross-method comparability.

## Limitations

- singleCellHaystack requires binary detection data as input; no continuous expression support is implemented, and a hard detection threshold can give identical DKL and p-values to genes with different expression levels but the same detection pattern.
- singleCellHaystack suffers from p-value inflation due to double use of gene expression data (coordinates such as PCs, t-SNE, or UMAP derive from the same data); this issue was deferred to future updates.
- The definition of the detection threshold is left to the user, introducing potential arbitrariness; zero counts may also reflect dropout, and imputation was not applied.
- Benchmark conclusions are specific to artificial datasets; scDD failures on the largest sets limit complete runtime comparison at 10,000 cells.

## Evidence

- [abstract] singleCellHaystack has higher accuracy than existing DEG prediction approaches on artificial datasets: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [methods] haystack called with cell coordinates and expression input: "res <- haystack(dat.tsne, dat.expression)"
- [methods] R implementation of singleCellHaystack: "library(singleCellHaystack)"
- [discussion] binary detection input requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [abstract] singleCellHaystack is fast and accurate: "We demonstrate that our method is a fast and accurate approach for DEG prediction in single-cell data"
- [readme] README notes on updated vs original version: [evidence span withheld — anchor status: external_doc]
