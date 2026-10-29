---
name: multiregional-gbm-data-analysis
description: Use when analyzing multiregional GBM scRNA-seq count matrices (e.g.,
  GSE131928 MGH samples) to detect clonal copy number substructure, classify malignant
  cells, or reproduce runtime benchmarking of SCEVAN vs inferCNV/CopyKAT.
when_to_use_negative:
- Tumors with minimal genomic alterations such as leukemia, pediatric cancers, and
  Ependymomas, where the aneuploidy-based assumption underlying SCEVAN is unsuitable
- Bulk WGS/WGS data rather than scRNA-seq count matrices, since the skill's segmentation
  operates on single-cell smoothed expression profiles
- Analyses requiring manual identification of reference normal cell clusters as a
  method choice, which inferCNV-like workflows require and SCEVAN is designed to avoid
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_2229
tools:
- name: SCEVAN
  role: 'Primary tool: pipelineCNA performs automatic malignant/non-malignant classification
    and greedy multichannel Mumford-Shah clonal CN segmentation of GBM scRNA-seq data'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Benchmark comparator for the classification stage runtime on the same count
    matrices
- name: inferCNV v1.4.0
  role: Benchmark comparator for classification runtime and segmentation speed reference
    on Multiple Myeloma data
- name: DNAcopy v1.62.0
  role: Segmentation reference used for GBM samples in the runtime speedup computation
- name: CNVkit v0.9.9
  role: Segmentation reference used for the Multiple Myeloma dataset in the runtime
    speedup computation
- name: yaGST
  role: Mann-Whitney-Wilcoxon single-sample gene set test used to identify confident
    normal cells (P < 1e-10, NES > 1.0)
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:f616f22ba73c24d425fa3a67cdac65c18a116dc3c0a14652b96c8eb4082cb41f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit v0.9.9
  - CopyKAT v1.0.5
  - DNAcopy
  - DNAcopy v1.62.0
  - SCEVAN
  - inferCNV v1.4.0
  - yaGST
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multiregional-gbm-data-analysis

## Summary

A method for analyzing multiregional glioblastoma scRNA-seq data (GSE131928 samples MGH102, MGH104, MGH105, MGH106) with SCEVAN to classify malignant vs non-malignant cells and reconstruct clonal copy number substructure, and to benchmark runtimes against CopyKAT v1.0.5 and inferCNV v1.4.0 (segmentation vs DNAcopy v1.62.0 on GBM and CNVkit v0.9.9 on Multiple Myeloma).

## When to use

Use when analyzing multiregional GBM scRNA-seq count matrices (e.g., GSE131928 MGH samples) to detect clonal copy number substructure, classify malignant cells, or reproduce runtime benchmarking of SCEVAN vs inferCNV/CopyKAT.

## When NOT to use

- Tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas, where the aneuploidy-based assumption underlying SCEVAN is unsuitable
- Bulk WGS/WGS data rather than scRNA-seq count matrices, since the skill's segmentation operates on single-cell smoothed expression profiles
- Analyses requiring manual identification of reference normal cell clusters as a method choice, which inferCNV-like workflows require and SCEVAN is designed to avoid

## Inputs

- Raw scRNA-seq count matrix with genes on rows (Gene Symbol or Ensembl ID) and cells on columns
- GSE131928 GBM multiregional samples MGH102, MGH104, MGH105, MGH106 (pre-processed MGH106.RData and listCountMtx.RData available from the SCEVAN README)
- HNSCC26.RData (GSE103322 head & neck primary and lymph node samples) as additional sample data
- Optional known normal cell names vector (norm_cells) and AdditionalGeneSets signatures

## Outputs

- Per-cell malignant/non-malignant classification and subclone assignment metadata (Seurat-compatible, integrable via AddMetaData)
- Per-clone segmented CNA matrix and .seg files (e.g., MGH106_CNAmtx.RData, MGH106_count_mtx_annot.RData)
- Shared and clone-specific alterations and phylogenetic tree of subclones
- runtime_benchmark.csv with per-tool, per-stage runtimes and SCEVAN speedup factors (classification 2-7x vs CopyKAT; segmentation 2x/5x on GBM, 11x/19x on MM)

## How to apply

Load the GBM multiregional count matrices from GSE131928 (MGH102, MGH104, MGH105, MGH106) into R and run SCEVAN::pipelineCNA with par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE. SCEVAN first filters cells with fewer than 200 detected genes and genes expressed in less than 1% of cells, annotates genes with Ensembl genomic locations sorted by coordinates, and removes REACTOME cell cycle pathway genes to reduce artificial segments. Confident normal cells are identified via yaGST gene signature tests (P < 1e-10, NES > 1.0, max 30 cells), their median expression baseline is subtracted, and the relative matrix is smoothed with edge-preserving TV (logcosh prior) before greedy Mumford-Shah joint segmentation (beta_vega, default 0.5). For runtime benchmarking, time classifyTumorCells and getBreaksVegaMC separately, run CopyKAT v1.0.5 and inferCNV v1.4.0 classification identically, segment with DNAcopy v1.62.0 (GBM) and CNVkit v0.9.9 (Multiple Myeloma), and compute speedup factors into runtime_benchmark.csv. On the Multiple Myeloma 10X data, SCEVAN segmentation is 11x faster than inferCNV and 19x faster than CopyKAT, with CopyKAT slow due to the large number of cells.

## Related tools

- **SCEVAN** (Primary tool: pipelineCNA performs automatic malignant/non-malignant classification and greedy multichannel Mumford-Shah clonal CN segmentation of GBM scRNA-seq data) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Benchmark comparator for the classification stage runtime on the same count matrices)
- **inferCNV v1.4.0** (Benchmark comparator for classification runtime and segmentation speed reference on Multiple Myeloma data)
- **DNAcopy v1.62.0** (Segmentation reference used for GBM samples in the runtime speedup computation)
- **CNVkit v0.9.9** (Segmentation reference used for the Multiple Myeloma dataset in the runtime speedup computation)
- **yaGST** (Mann-Whitney-Wilcoxon single-sample gene set test used to identify confident normal cells (P < 1e-10, NES > 1.0)) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Reproduced classification speedup vs CopyKAT falls in the reported 2-7x range
- Reproduced segmentation speedup on GBM (vs DNAcopy) is ~2x/5x and on Multiple Myeloma (vs CNVkit/inferCNV) is ~11x/19x
- F1 score of malignant/non-malignant classification across samples reaches ~0.90, versus 0.63 for state-of-the-art comparators
- Cell and gene filters applied correctly: cells with <200 detected genes and genes expressed in <1% of cells removed; REACTOME cell cycle genes filtered
- Confident normal cells pass yaGST thresholds P < 1e-10 and NES > 1.0 (top 30 cells); if none found, sample treated as pure tumor with synthetic gaussian baseline removal

## Limitations

- The aneuploidy-based approach may not suit tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)
- CopyKAT is particularly slow on datasets with large cell numbers (e.g., Multiple Myeloma 10X data), limiting comparability at scale
- Ground truth CNAs from WES/WGS have different resolution than single-cell data and require resampling before comparison
- Version of SCEVAN itself is not stated in the article, unlike compared tool versions

## Evidence

- [abstract] SCEVAN automatically classifies GBM cells and reconstructs clonal substructure: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment"
- [discussion] Filtering thresholds before segmentation: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] Confident normal cell identification thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] Benchmark comparator versions: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [methods] pipelineCNA invocation on GBM sample: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [readme] README: F1 comparison and speed rationale: [evidence span withheld — anchor status: external_doc]
- [readme] README: GSE131928 sample data availability: [evidence span withheld — anchor status: not_found]
