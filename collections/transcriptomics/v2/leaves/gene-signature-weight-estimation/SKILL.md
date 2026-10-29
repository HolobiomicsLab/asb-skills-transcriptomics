---
name: gene-signature-weight-estimation
description: Use when mapping target scRNA-seq cells to reference clusters and you
  need discriminative weights for cluster-specific gene signatures — e.g. before scid_multiclass
  scoring or when batch effect/strong asymmetry in cell number and library size means
  weights should come from the target data.
when_to_use_negative:
- Input data are log-transformed — scID requires library-depth normalized (TPM/CPM)
  values, not log-transformed data
- Neither reference nor target data are well clustered or contain good gene signatures
  — the article states scID is not a recommended approach in that case
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: Provides choose_training_set (DPR-based c+/c- cell selection) and scID_weight
    (Fisher's LDA-like weight fitting), plus scid_multiclass with estimate_weights_from_target
    parameter
  repo: github.com/BatadaLab/scID
- name: R
  role: Runtime environment for the scID package and its vignette workflows
  repo: github.com/BatadaLab/scID
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:1702a86ed95b6dfc5003e3dc05b85adf6fdecaec939379ce5019ea3614b4cdba
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
  - scID
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-signature-weight-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates discriminative gene weights for reference-cluster signatures using scID's Fisher's Linear Discriminant Analysis-like framework: differentially expressed marker genes (logFC >= 0.6) are weighted by their ability to separate putative c+ and c- cells selected via differential precision-recall (DPR), enabling scoring of target cells in cross-dataset mapping.

## When to use

Use when mapping target scRNA-seq cells to reference clusters and you need discriminative weights for cluster-specific gene signatures — e.g. before scid_multiclass scoring or when batch effect/strong asymmetry in cell number and library size means weights should come from the target data.

## When NOT to use

- Input data are log-transformed — scID requires library-depth normalized (TPM/CPM) values, not log-transformed data
- Neither reference nor target data are well clustered or contain good gene signatures — the article states scID is not a recommended approach in that case

## Inputs

- reference_gem.rds: TPM/CPM-normalized reference gene expression matrix (genes x cells)
- reference_clusters.rds: named list of cluster labels for reference cells
- markers.rds or find_markers output: cluster-specific positive/negative marker lists (logFC 0.6)
- target_gem.rds: library-depth-normalized target expression matrix (if estimating weights from target, estimate_weights_from_target = TRUE)

## Outputs

- scID_output$weights: list of estimated discriminative weights for all cluster-specific genes
- scID_output$markers: data frame of signature genes extracted from reference clusters
- Per-cluster training-cell selections from DPR (true_cells / false_cells)

## How to apply

First extract cluster-specific gene signatures from the reference data with find_markers using a minimum logFC of 0.6 on TPM/CPM-normalized (not log-transformed) expression. For each cluster, run choose_training_set(gem, positive_markers, negative_markers) to select putative c+ and c- training cells via the DPR procedure, then fit weights with scID_weight(gem, true_cells, false_cells). Alternatively, pass estimate_weights_from_target = TRUE to scid_multiclass so weights are estimated from the target dataset itself; the article reports this is more accurate when reference and target have strong batch effect. Weights can be estimated from either the reference or the target (estimate_weights_from_target parameter), and scID v2.0.0 uses both positive and negative markers, which improves classification in the presence of very similar cell types.

## Related tools

- **scID** (Provides choose_training_set (DPR-based c+/c- cell selection) and scID_weight (Fisher's LDA-like weight fitting), plus scid_multiclass with estimate_weights_from_target parameter) — github.com/BatadaLab/scID
- **R** (Runtime environment for the scID package and its vignette workflows) — github.com/BatadaLab/scID

## Examples

```
true_cells <- scID::choose_training_set(gem, positive_markers, negative_markers); weights <- scID::scID_weight(gem, true_cells, false_cells)
```

## Evaluation signals

- Median per-cluster true-positive rate of DPR-selected training cells falls in the reported 45%-80% range across published reference datasets
- Median per-cluster false-positive rate falls in the reported 0%-5% range
- Weights object (scID_output$weights) is populated for every cluster-specific gene in the signature list
- Downstream scid_multiclass mapping using these weights reproduces reported coverage, e.g. 99.5% of mouse brain snRNA-seq cells (n=949) assigned into 14 reference clusters
- Reference expression matrices are TPM/CPM-normalized, not log-transformed, before weight estimation

## Limitations

- When reference and target have strong batch effect, estimating weights from the target appears more accurate; weights from the reference may underperform
- scID is not recommended if neither dataset is well clustered or contains good gene signatures
- Biomarker/signature-based approaches (which weight estimation feeds) can suffer from the dropout problem in scRNA-seq data

## Evidence
