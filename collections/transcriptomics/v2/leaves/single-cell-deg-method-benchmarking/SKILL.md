---
name: single-cell-deg-method-benchmarking
description: Use when you need to quantitatively compare DEG prediction methods on
  single-cell RNA-seq data with known ground truth — i.e., artificial datasets with
  planted DEGs where AUC and elapsed runtime per method per dataset size can be tabulated.
when_to_use_negative:
- The dataset is real data without known ground-truth DEGs — AUC against planted labels
  cannot be computed, so this benchmarking design does not apply.
- The goal is spatial DEG discovery on actual tissue coordinates (e.g., 10x Genomics
  Visium) rather than method comparison on artificial benchmarks.
- Gene expression is only available as continuous normalized counts and cannot be
  binarized into detection data, since singleCellHaystack requires binary detection
  input.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Method under benchmark; run via R haystack() on cell coordinates and binary
    expression/detection input; use the 'binary' branch for the version described
    in the paper
  repo: github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Comparison DEG method run as FindAllMarkers, both with and without its default
    filtering
  repo: github.com/satijalab/seurat
- name: monocle 2
  role: Comparison DEG prediction method run on the same artificial datasets
- name: DEsingle
  role: Cluster-based comparison method with far longer runtime on large datasets
- name: EMDomics
  role: Comparison DEG prediction method scored and AUC-ranked on identical datasets
- name: scDD
  role: Comparison method that fails on the largest artificial datasets; failures
    must be recorded in the benchmark
- name: edgeR
  role: Bulk RNA-seq comparison method showing bulk approaches need not perform worse
    than scRNA-seq-specific ones
- name: ROCR
  role: Computes the AUC of the DEG detection ROC curve using planted DEGs as true
    labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:93e56925c70ad6fda9a1f7feb6c64833c96cc74e1cf1465aa67567267829b428
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-deg-method-benchmarking

## Summary

Benchmark the accuracy (AUC of DEG detection ROC via ROCR) and runtime of singleCellHaystack against DEsingle, EMDomics, scDD, edgeR, monocle 2, and Seurat FindAllMarkers on artificial scRNA-seq datasets with planted DEGs, reproducing the Vandenbon & Diez benchmark table.

## When to use

Use when you need to quantitatively compare DEG prediction methods on single-cell RNA-seq data with known ground truth — i.e., artificial datasets with planted DEGs where AUC and elapsed runtime per method per dataset size can be tabulated.

## When NOT to use

- The dataset is real data without known ground-truth DEGs — AUC against planted labels cannot be computed, so this benchmarking design does not apply.
- The goal is spatial DEG discovery on actual tissue coordinates (e.g., 10x Genomics Visium) rather than method comparison on artificial benchmarks.
- Gene expression is only available as continuous normalized counts and cannot be binarized into detection data, since singleCellHaystack requires binary detection input.

## Inputs

- Artificial scRNA-seq count matrices with known planted DEGs in non-randomly positioned cell subsets (multiple cell numbers, e.g., 1000–10000 cells)
- Cell coordinates in a multidimensional space (t-SNE, PCA, or other embedding) for haystack()
- Binary detection matrices (gene detected/not detected per cell)
- R script files implementing each method's DEG call and runtime measurement
- ROCR prediction/label objects for AUC computation

## Outputs

- benchmark_median_auc_runtime.csv with one row per method per dataset size (median AUC, median runtime)
- Per-method ranked gene lists with method-specific significance scores per dataset
- Runtime logs including recorded failures (e.g., scDD on the largest sets)

## How to apply

Generate artificial scRNA-seq datasets of increasing size (e.g., 1000 up to 10,000 cells) containing known DEGs in non-randomly positioned cell subsets, so that the planted DEGs serve as true labels. Run singleCellHaystack via its R interface (haystack() on cell coordinates with binary detection input, after filtering genes detected in fewer than 10 cells) and each comparison method (DEsingle, EMDomics, scDD, edgeR, monocle 2, and Seurat FindAllMarkers with and without default filtering) on identical datasets, recording elapsed runtime and noting failures such as scDD on the largest sets. Rank genes by each method's significance score and compute the AUC of the ROC curve with ROCR, treating planted DEGs as positives. Aggregate per method and dataset size by taking median AUC and median runtime across replicates, write the summary table to benchmark_median_auc_runtime.csv, and verify the reported trends: singleCellHaystack attains the highest accuracy except at 1000 cells and completes in about 5 minutes on 10,000-cell datasets, while cluster-based methods like DEsingle and scDD have far longer runtimes.

## Related tools

- **singleCellHaystack** (Method under benchmark; run via R haystack() on cell coordinates and binary expression/detection input; use the 'binary' branch for the version described in the paper) — github.com/alexisvdb/singleCellHaystack
- **Seurat** (Comparison DEG method run as FindAllMarkers, both with and without its default filtering) — github.com/satijalab/seurat
- **monocle 2** (Comparison DEG prediction method run on the same artificial datasets)
- **DEsingle** (Cluster-based comparison method with far longer runtime on large datasets)
- **EMDomics** (Comparison DEG prediction method scored and AUC-ranked on identical datasets)
- **scDD** (Comparison method that fails on the largest artificial datasets; failures must be recorded in the benchmark)
- **edgeR** (Bulk RNA-seq comparison method showing bulk approaches need not perform worse than scRNA-seq-specific ones)
- **ROCR** (Computes the AUC of the DEG detection ROC curve using planted DEGs as true labels)

## Examples

```
library(singleCellHaystack); res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- benchmark_median_auc_runtime.csv contains exactly one row per method per dataset size, with finite median AUC values in [0,1] and finite median runtimes.
- The tabulated AUC ordering reproduces the reported finding: singleCellHaystack has the highest AUC across artificial datasets except at the smallest size (1000 cells).
- singleCellHaystack's median runtime on 10,000-cell datasets is on the order of 5 minutes, placing it among the fastest methods.
- Failures of scDD on the largest datasets are explicitly recorded rather than silently dropped.
- Each method's AUC is computed from its own significance-score gene ranking via ROCR with the planted DEGs as true labels, not from cluster assignments.

## Limitations

- singleCellHaystack (binary version) requires binary detection data as input; no continuous expression support was implemented, and the hard detection threshold is a weak point — genes with different expression levels but the same detection pattern receive identical DKL and p-value.
- P-values from singleCellHaystack are inflated by double use of gene expression data (coordinates derive from the same data); this issue was acknowledged and deferred to future updates.
- singleCellHaystack attains the highest accuracy only except at the smallest dataset size (1000 cells), and scDD fails on the largest artificial sets, so the benchmark table is incomplete for that method.

## Evidence

- [abstract] Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [methods] Gene filtering threshold: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] Binary input requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] Correct package version for the paper's benchmark: [evidence span withheld — anchor status: external_doc]
