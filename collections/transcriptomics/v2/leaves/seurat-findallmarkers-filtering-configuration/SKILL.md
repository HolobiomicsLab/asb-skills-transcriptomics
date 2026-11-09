---
name: seurat-findallmarkers-filtering-configuration
description: Use when benchmarking DEG-prediction methods on artificial single-cell
  datasets with known/planted DEGs, and you need Seurat FindAllMarkers results — both
  with default filtering and unfiltered — as a cluster-based comparison baseline.
when_to_use_negative:
- When the goal is DEG prediction without relying on cell clustering — the source
  study showed singleCellHaystack achieves higher accuracy on artificial datasets
  and FindAllMarkers depends on cluster definitions that are 'often not straightforward'
- When input is not a scRNA-seq expression/detection matrix with cell-level metadata,
  since FindAllMarkers requires clustered Seurat objects rather than a pre-ranked
  feature table or generic feature matrix
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Clustering-based DEG-prediction baseline run via FindAllMarkers with and without
    default filtering
  repo: https://github.com/satijalab/seurat
- name: ROCR
  role: Computes AUC of the DEG-detection ROC curve using planted DEGs as true labels
- name: singleCellHaystack
  role: Reference method against which the FindAllMarkers configurations are compared
    for accuracy and runtime
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:bafe1b99b8ed1271a0281b0b50529f96dcca8c19ee93650a8d6ec5a41bc6c66a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
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
yes_rate: 0.9333333333333333
n_claims: 15
---

# seurat-findallmarkers-filtering-configuration

## Summary

Configure and run Seurat's FindAllMarkers, with and without its default filtering, as a clustering-based DEG-prediction baseline whose gene rankings are scored by AUC against planted DEGs when benchmarking methods like singleCellHaystack on artificial scRNA-seq datasets.

## When to use

Use when benchmarking DEG-prediction methods on artificial single-cell datasets with known/planted DEGs, and you need Seurat FindAllMarkers results — both with default filtering and unfiltered — as a cluster-based comparison baseline.

## When NOT to use

- When the goal is DEG prediction without relying on cell clustering — the source study showed singleCellHaystack achieves higher accuracy on artificial datasets and FindAllMarkers depends on cluster definitions that are 'often not straightforward'
- When input is not a scRNA-seq expression/detection matrix with cell-level metadata, since FindAllMarkers requires clustered Seurat objects rather than a pre-ranked feature table or generic feature matrix

## Inputs

- Artificial (or real) scRNA-seq count/detection matrix with planted or known DEGs used as true labels
- Seurat object with cell cluster identities assigned by prior clustering
- Dataset-size strata (e.g., 1,000 up to 10,000 cells)

## Outputs

- Per-gene significance-score rankings from FindAllMarkers (filtered and unfiltered configurations)
- Per-run elapsed runtimes
- AUC values of the DEG-detection ROC curve computed with ROCR
- Aggregated rows (method, dataset size, median AUC, median runtime) written to benchmark_median_auc_runtime.csv

## How to apply

First cluster the artificial scRNA-seq dataset within Seurat, since FindAllMarkers is a cluster-based approach that requires cell cluster identities. Then run FindAllMarkers twice per dataset — once with default filtering parameters and once with the filtering disabled — recording elapsed runtime for each configuration on datasets spanning roughly 1,000 to 10,000 cells. Rank genes by the method-specific significance scores returned by each configuration. Treat the planted DEGs in the artificial data as true labels and compute the AUC of the DEG-detection ROC curve with the ROCR package. Finally, aggregate per-method results by dataset size (median AUC and median runtime) so the unfiltered and filtered configurations can be compared against singleCellHaystack, which the source study found achieved higher accuracy on artificial datasets except at the smallest 1,000-cell size.

## Related tools

- **Seurat** (Clustering-based DEG-prediction baseline run via FindAllMarkers with and without default filtering) — https://github.com/satijalab/seurat
- **ROCR** (Computes AUC of the DEG-detection ROC curve using planted DEGs as true labels)
- **singleCellHaystack** (Reference method against which the FindAllMarkers configurations are compared for accuracy and runtime) — https://github.com/alexisvdb/singleCellHaystack

## Evaluation signals

- The output CSV contains one row per method per dataset size with finite median AUC in [0,1] and finite median runtime
- Both FindAllMarkers configurations (with and without default filtering) produce complete runtime measurements on every dataset size including 10,000 cells
- Median-AUC values for filtered and unfiltered FindAllMarkers are distinguishable, reflecting the effect of the default filtering on DEG detection
- The tabulated AUC ordering reproduces the reported finding that singleCellHaystack attains higher accuracy than Seurat FindAllMarkers on artificial datasets
- Planted DEGs used as true labels are consistently defined across all method-dataset combinations

## Limitations

- Cluster-based DEG prediction suffers from p-value inflation because gene expression data are used twice — once for defining clusters and once for DEG prediction
- The definition of cell clusters is often not straightforward, and the number of biologically relevant clusters is often not obvious, affecting FindAllMarkers reproducibility
- Comparison-method runtimes grow steeply with dataset size; cluster-based methods like scDD had far longer runtimes and failed on the largest sets, so plan for failure at high cell counts

## Evidence

- [intro] Seurat included as a comparison DEG tool: "Hundreds of bioinformatics tools have been developed to process, analyze, and interpret the results from single-cell genomics data6, such as monocle 2 and Seurat7,8."
- [abstract] singleCellHaystack outperforms comparison methods on artificial data: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [methods] Seurat loaded in the methods environment: "library(Seurat)
library(SeuratData)"
- [readme] Seurat is an R toolkit for single-cell analysis: [evidence span withheld — anchor status: external_doc]
- [intro] Cluster definition is a prerequisite problem for cluster-based DEG methods: "A major problem with clustering-based approaches for DEG prediction is that the definition of cell clusters is often not straightforward."
