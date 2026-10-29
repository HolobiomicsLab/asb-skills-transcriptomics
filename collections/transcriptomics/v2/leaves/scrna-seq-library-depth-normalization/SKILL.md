---
name: scrna-seq-library-depth-normalization
description: Use when preparing raw count matrices from scRNA-seq/snRNA-seq datasets
  (e.g. 10X neuron_9k and nuclei_900) as input to scID or similar methods that require
  library-depth-normalized, non-log-transformed data.
when_to_use_negative:
- Data are already log-transformed — scID requires non-log-transformed values, so
  log-normalized matrices must not be fed in directly
- The goal is cross-dataset integration by batch correction rather than reference-based
  cell assignment — the article notes batch-correction methods risk over-correction
  and losing biological variance
- Neither the reference nor the target is well clustered or contains good gene signatures
  — the article states scID is not a recommended approach in that case
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: Provides counts_to_cpm for library-depth (CPM) normalization and requires
    the normalized, non-log-transformed matrices as input to scid_multiclass
  repo: github.com/BatadaLab/scID
- name: R
  role: Language in which the normalization and the scID pipeline are executed
- name: Seurat
  role: Listed as an import of the scID R package in the methods section
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:e0c8882ff3e007e725a3c27bcc700c5da1c2c1ec1c1014021951f113cccea666
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  tools:
  - CaSTLe
  - R
  - Seurat
  - data.table
  - ggplot2
  - scID
  - scVI
  - scmap
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scRNA-seq library-depth normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies library-depth (TPM or CPM) column normalization to scRNA-seq count matrices so they meet scID's input requirement of depth-normalized, non-log-transformed expression, using scID's counts_to_cpm helper.

## When to use

Use when preparing raw count matrices from scRNA-seq/snRNA-seq datasets (e.g. 10X neuron_9k and nuclei_900) as input to scID or similar methods that require library-depth-normalized, non-log-transformed data.

## When NOT to use

- Data are already log-transformed — scID requires non-log-transformed values, so log-normalized matrices must not be fed in directly
- The goal is cross-dataset integration by batch correction rather than reference-based cell assignment — the article notes batch-correction methods risk over-correction and losing biological variance
- Neither the reference nor the target is well clustered or contains good gene signatures — the article states scID is not a recommended approach in that case

## Inputs

- Raw count gene expression matrix (n genes x m cells) for the target dataset
- Raw count gene expression matrix (N genes x M cells) for the reference dataset
- Reference cluster labels for reference cells (e.g. reference_clusters.rds)
- Optionally, pre-computed TPM-normalized .rds files from the scID GitHub ExampleData folder

## Outputs

- Library-depth (TPM/CPM) normalized expression matrices in linear (non-log) scale, ready for scid_multiclass
- A normalized reference matrix compatible with find_markers and scID_weight stages

## How to apply

Take the raw gene-by-cell count matrices (genes as rows, cells as columns) and apply a library-depth or column normalization such as TPM or CPM, scaling each cell's counts by its total and a size factor; scID's counts_to_cpm function performs CPM normalization directly. Critically, do NOT log-transform the values, because scID requires linear-scale normalized expression for its Fisher-LDA-like scoring. For the scID mouse brain example, download the pre-computed TPM-normalized target_gem.rds, reference_gem.rds and reference_clusters.rds from the BatadaLab/scID ExampleData folder, or reproduce normalization on the 10X E18 neuron_9k and nuclei_900 count matrices. If reference and target differ strongly in library depth or protocol (e.g. Smart-Seq2 reference vs CEL-Seq1/2 target), normalize both consistently but set estimate_weights_from_target = TRUE downstream, since estimating weights from the target is more accurate under strong batch effect.

## Related tools

- **scID** (Provides counts_to_cpm for library-depth (CPM) normalization and requires the normalized, non-log-transformed matrices as input to scid_multiclass) — github.com/BatadaLab/scID
- **R** (Language in which the normalization and the scID pipeline are executed)
- **Seurat** (Listed as an import of the scID R package in the methods section)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, normalize_reference = FALSE, estimate_weights_from_target = FALSE)
```

## Evaluation signals

- Check that every cell column has been scaled to a common library depth (TPM/CPM totals are constant across cells)
- Verify values remain on the linear scale — confirm no log2/log10 transformation was applied, since 'Any library-depth normalization (e.g. TPM, CPM) is compatible with scID, but not log-transformed data'
- Downstream check: the make_heatmap of cluster-specific geneset expression in the target clusters should resemble the reference heatmap pattern
- If using the pre-computed example data, confirm target_gem.rds, reference_gem.rds and reference_clusters.rds load correctly from the ExampleData folder before normalization decisions

## Limitations

- Log-transformed data are incompatible with scID — feeding log-normalized matrices will invalidate the Fisher-LDA-like scoring
- Imbalanced library sizes per cell between reference and target affect accuracy; when they differ strongly, gene-signature weights should be estimated from the target rather than the reference
- Imputation of gene expression for highly shallow target data is not implemented and is deferred to future work, limiting power on very sparse targets

## Evidence
