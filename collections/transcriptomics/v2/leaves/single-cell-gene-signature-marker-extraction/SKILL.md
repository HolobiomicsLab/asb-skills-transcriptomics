---
name: single-cell-gene-signature-marker-extraction
description: Use when a labeled reference scRNA-seq dataset (e.g. Drop-seq clusters)
  must be converted into per-cluster marker gene sets before scoring or classifying
  cells of an unlabeled target dataset (e.g. Smart-seq2).
when_to_use_negative:
- Input data are log-transformed — scID requires library-depth (TPM/CPM) normalized,
  non-log data
- The reference data are not well clustered and lack good gene signatures — scID is
  not a recommended approach in that case
- The goal is batch correction/integration of balanced datasets rather than marker-based
  mapping — batch-correction methods may be preferable
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: Provides find_markers to extract cluster-specific gene signatures and estimate
    discriminative (Fisher's LDA-like, DPR-based) weights from reference or target
    data
  repo: github.com/BatadaLab/scID
- name: R
  role: Implementation environment; scID is an R package installed via devtools and
    used in Rmarkdown vignettes
  repo: github.com/BatadaLab/scID
- name: Seurat
  role: Listed as an Imports dependency of the scID R package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:04acd70fdb9de3f7099f3ce54ea48eb0c204bf1681e70757eaf0e76c70184182
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-gene-signature-marker-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts cluster-specific gene signatures from a reference scRNA-seq dataset with scID::find_markers (logFC threshold, positive and negative markers) so they can weight and score target cells during cross-dataset mapping. This is Stage 1 of the scID Fisher's Linear Discriminant Analysis-like pipeline.

## When to use

Use when a labeled reference scRNA-seq dataset (e.g. Drop-seq clusters) must be converted into per-cluster marker gene sets before scoring or classifying cells of an unlabeled target dataset (e.g. Smart-seq2).

## When NOT to use

- Input data are log-transformed — scID requires library-depth (TPM/CPM) normalized, non-log data
- The reference data are not well clustered and lack good gene signatures — scID is not a recommended approach in that case
- The goal is batch correction/integration of balanced datasets rather than marker-based mapping — batch-correction methods may be preferable

## Inputs

- reference_gem: NxM data frame of N genes (rows) x M cells (columns), library-depth (TPM/CPM) normalized, non-log-transformed
- reference_clusters: list of cluster labels for the reference cells (e.g. 15 Drop-seq clusters)
- logFC threshold parameter (minimum log-fold-change for cluster-specific gene selection)

## Outputs

- markers: data frame of signature genes extracted from each reference cluster (e.g. as an .rds markers file)
- weights: list of estimated discriminative weights for all cluster-specific genes
- Per-cluster gene signatures usable by scid_multiclass for target-cell assignment

## How to apply

Load the reference expression matrix with cluster labels using library-depth-normalized (TPM/CPM) data, NOT log-transformed data, since scID requires non-log data. Call scID::find_markers(reference_gem, reference_clusters, logFC, only.pos, normalize_reference), where logFC sets the minimum log-fold-change for a gene to be selected as cluster-specific; use only.pos=FALSE in scID v2.0.0+ because combining negative markers with positive ones improves classification among very similar cell types. The returned marker data frame feeds scid_multiclass, and signature weights for these genes can be estimated either from the reference or from the target data (estimate_weights_from_target), the latter being more accurate under strong batch effect. Internally the discriminative weights are derived by identifying putative cluster-positive and cluster-negative cells (choose_training_set) and applying differential precision-recall (DPR)-based weighting via scID_weight.

## Related tools

- **scID** (Provides find_markers to extract cluster-specific gene signatures and estimate discriminative (Fisher's LDA-like, DPR-based) weights from reference or target data) — github.com/BatadaLab/scID
- **R** (Implementation environment; scID is an R package installed via devtools and used in Rmarkdown vignettes) — github.com/BatadaLab/scID
- **Seurat** (Listed as an Imports dependency of the scID R package)

## Examples

```
markers <- scID::find_markers(reference_gem, reference_clusters, logFC, only.pos, normalize_reference)
```

## Evaluation signals

- Marker extraction runs on non-log, library-depth-normalized input (TPM/CPM), verified by checking data scale before find_markers
- Each reference cluster yields a gene signature meeting the chosen logFC minimum log-fold-change threshold, including negative markers (only.pos=FALSE in v2.0.0+)
- scid_multiclass on the 288-cell Smart-seq2 target assigns 100% of cells (288/288) into the 15 Drop-seq reference clusters
- make_heatmap of average expression of each cluster-specific geneset shows enrichment in the corresponding reference/target clusters
- Downstream mapping accuracy (ARI/VI against reference-derived ground truth) places the method among the most accurate, as in Figures 2D-2E

## Limitations

- Biomarker-based identification of equivalent cells can be hampered by the dropout problem in scRNA-seq data
- If the reference is not well clustered and contains no good gene signatures, scID is not recommended
- Imputation of gene expression for shallow target data is deferred to future work, not implemented or evaluated
- When reference and target have strong batch effect, estimating weights from the target (estimate_weights_from_target=TRUE) appears more accurate than from the reference

## Evidence

- [readme] v2.0.0 uses negative markers with positive: [evidence span withheld — anchor status: external_doc]
