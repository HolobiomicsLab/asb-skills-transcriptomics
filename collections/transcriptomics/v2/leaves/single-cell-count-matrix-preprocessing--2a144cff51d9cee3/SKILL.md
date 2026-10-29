---
name: single-cell-count-matrix-preprocessing
description: Use when raw gene-by-cell scRNA-seq count matrices (cells in rows, genes
  in columns, CSV or similar) must be normalized and rescaled before scGATE inference,
  or when base GRN .parquet files need read_base_GRN() summarization.
when_to_use_negative:
- Data are already library-size normalized and rescaled — re-running normalization
  on pre-normalized deposits can distort values; set library_size_normalization =
  'False'
- The goal is genome-wide large-scale GRN inference — scGATE preprocessing is designed
  for small (<200) gene subsets with a refined candidate TF list
- Input is pseudotime-sorted single-cell data along differentiation trajectories —
  scGATE does not yet handle temporal ordering or delays
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGATE
  role: Provides scRNA_seq_preprocessing() for normalization/rescaling and read_base_GRN()
    for base GRN summarization
  repo: https://github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Language environment in which the scGATE package runs (Windows and Linux)
- name: arrow
  role: Used to read base GRN .parquet files before read_base_GRN() summarization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:ea284723000716eb260abb94a38fb95ac27c0a3267bd4d1cb8ae4d3177862b7f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  repo_url: https://github.com/CompBioIPM/scGATE
  tools:
  - Monocle
  - R
  - R 4.1.3
  - Rtsne
  - TCM
  - TSCAN
  - VGAM
  - Wishbone
  - arrow
  - devtools
  - diffusionMap
  - doParallel
  - doSNOW
  - foreach
  - scDatasets
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# single-cell-count-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess single-cell RNA-seq count matrices (e.g. mouse haematopoiesis or Tabula Muris CSV counts) with scGATE's scRNA_seq_preprocessing(), performing library size normalization and rescaling into the (0,1) interval so downstream logic-gate inference receives suitable continuous input.

## When to use

Use when raw gene-by-cell scRNA-seq count matrices (cells in rows, genes in columns, CSV or similar) must be normalized and rescaled before scGATE inference, or when base GRN .parquet files need read_base_GRN() summarization.

## When NOT to use

- Data are already library-size normalized and rescaled — re-running normalization on pre-normalized deposits can distort values; set library_size_normalization = 'False'
- The goal is genome-wide large-scale GRN inference — scGATE preprocessing is designed for small (<200) gene subsets with a refined candidate TF list
- Input is pseudotime-sorted single-cell data along differentiation trajectories — scGATE does not yet handle temporal ordering or delays

## Inputs

- Raw scRNA-seq count matrix (cells in rows, genes in columns; e.g. .csv such as subset_counts_cluster_11.csv)
- Base GRN candidate TF-target dataframe in .parquet format (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- Optional TF list (tf_list) to restrict preprocessing to transcription factors

## Outputs

- Normalized and rescaled gene expression matrix with values within the (0,1) interval, ready for scGATE_logic()/scGATE_edge()
- Summarized base GRN dataframe from read_base_GRN()

## How to apply

Load the raw count matrix (e.g. subset_counts_cluster_11.csv from the mouse haematopoiesis Zenodo deposit 10.5281/zenodo.8353409, with cells in rows and genes in columns) and call scRNA_seq_preprocessing() with the library_size_normalization flag set appropriately: 'True' to divide each cell's raw UMI counts by total reads, or 'False' for deposits that are already normalized. Rescale the expression values into the (0,1) interval, since scGATE models continuous expression via a Hill activation function rather than binarizing; optionally restrict to a supplied TF list via tf_list. Separately, summarize base GRN data (e.g. Buenrostro2018_base_GRN_dataframe.parquet) with read_base_GRN() to obtain the candidate TF-target list. Validate that the output matrix contains only values in (0,1) before passing it to scGATE_logic() or scGATE_edge().

## Related tools

- **scGATE** (Provides scRNA_seq_preprocessing() for normalization/rescaling and read_base_GRN() for base GRN summarization) — https://github.com/CompBioIPM/scGATE
- **R 4.1.3** (Language environment in which the scGATE package runs (Windows and Linux))
- **arrow** (Used to read base GRN .parquet files before read_base_GRN() summarization)

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "True", tf_list = NA)
```

## Evaluation signals

- Output matrix values all lie within the (0,1) interval as required by scGATE_logic()/scGATE_edge() input specification
- Each cell's normalized counts sum consistently when library_size_normalization = 'True' (UMI counts divided by total reads)
- Base GRN dataframe is successfully read via arrow/read_parquet and summarized by read_base_GRN() without NA-collapsed candidate edges
- Downstream scGATE_logic() runs without errors on the preprocessed matrix, indicating compatibility (e.g. AUROC > 0.9 achievable with ≥1000 cells and 15 regulatory TFs)

## Limitations

- library_size_normalization must be disabled ('False') for deposits that are already pre-normalized, e.g. Zenodo 10.5281/zenodo.8353409 files
- scGATE operates on highly variable genes per context; genes with low variability should be removed
- scGATE is applicable only to a small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints
- No software version numbers beyond scGATE 0.1.0 and R 4.1.3 are stated for dependency packages

## Evidence
