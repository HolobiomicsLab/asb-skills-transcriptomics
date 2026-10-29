---
name: cluster-marker-signature-extraction
description: Use when you have a TPM/CPM-normalized reference gene expression matrix
  with known cluster labels and need cluster-specific marker gene sets (e.g., via
  find_markers with logFC 0.6) to score or map target cells.
when_to_use_negative:
- Input gene expression is log-transformed rather than TPM/CPM library-depth normalized
  — the article states this is not compatible with scID marker/weight estimation.
- The reference dataset is not well clustered and lacks good gene signatures — the
  paper warns scID is not a recommended approach for identification of equivalent
  cell types in that case.
- You only need final target-cell labels via a full one-call mapping; then scid_multiclass(target_gem,
  reference_gem, reference_clusters, ...) internally handles signature extraction
  and marker output.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scID
  role: Provides find_markers to extract cluster-specific gene signatures from the
    reference data, and scid_multiclass whose $markers output returns the signature
    genes
  repo: github.com/BatadaLab/scID
- name: R
  role: Runtime environment for loading reference_gem.rds / markers.rds and executing
    scID functions
- name: Seurat
  role: Listed as an Imports dependency of the scID package used for reference-data
    handling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:3d77728d1b4070aa3253d61f61db788b0f310440e046740ba8a1bcd368abc74f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  tools:
  - R
  - Seurat
  - scID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cluster-marker-signature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts cluster-specific gene signatures (differentially expressed positive and negative markers) from a clustered reference scRNA-seq gene expression matrix, as the first stage of the scID Fisher's Linear Discriminant Analysis-like cell-mapping pipeline. Signatures and their logFC thresholds feed downstream discriminative weight estimation (choose_training_set / scID_weight).

## When to use

Use when you have a TPM/CPM-normalized reference gene expression matrix with known cluster labels and need cluster-specific marker gene sets (e.g., via find_markers with logFC 0.6) to score or map target cells.

## When NOT to use

- Input gene expression is log-transformed rather than TPM/CPM library-depth normalized — the article states this is not compatible with scID marker/weight estimation.
- The reference dataset is not well clustered and lacks good gene signatures — the paper warns scID is not a recommended approach for identification of equivalent cell types in that case.
- You only need final target-cell labels via a full one-call mapping; then scid_multiclass(target_gem, reference_gem, reference_clusters, ...) internally handles signature extraction and marker output.

## Inputs

- reference_gem: nxm data frame of n genes (rows) by m cells (columns), TPM- or CPM-normalized (not log-transformed), e.g. reference_gem.rds from the scID ExampleData repository
- reference_clusters: list of cluster labels for the reference cells, e.g. reference_clusters.rds
- markers.rds (optional): precomputed data frame of cluster-specific signature genes from the BatadaLab/scID ExampleData repository
- logFC threshold parameter (e.g. 0.6) and only.pos flag controlling marker stringency

## Outputs

- markers: a data frame of signature genes extracted from the reference clusters (also returned as scID_output$markers by scid_multiclass)
- per-cluster positive/negative marker lists consumable by choose_training_set and scID_weight

## How to apply

Start from a reference gene expression matrix (reference_gem) normalized by library depth (TPM or CPM — log-transformed data are not compatible) and a list of reference cluster labels (reference_clusters). Call find_markers(reference_gem, reference_clusters, logFC, only.pos, normalize_reference), where logFC sets the minimum log-fold change for a gene to be selected as cluster-specific; the reproduced DPR evaluation workflow uses logFC 0.6. To include negative markers (the scID v2.0.0 behavior), generate cluster-specific marker lists containing both positive and negative markers, or load a precomputed markers.rds file such as the one in the BatadaLab/scID ExampleData repository. The resulting data frame of signature genes is used both for training-cell selection via choose_training_set(gem, positive_markers, negative_markers) and for weight estimation with scID_weight(gem, true_cells, false_cells). Decision point: extract markers yourself with find_markers (controlling logFC and only.pos) versus using precomputed markers when available.

## Related tools

- **scID** (Provides find_markers to extract cluster-specific gene signatures from the reference data, and scid_multiclass whose $markers output returns the signature genes) — github.com/BatadaLab/scID
- **R** (Runtime environment for loading reference_gem.rds / markers.rds and executing scID functions)
- **Seurat** (Listed as an Imports dependency of the scID package used for reference-data handling)

## Examples

```
markers <- find_markers(reference_gem, reference_clusters, logFC = 0.6, only.pos = FALSE, normalize_reference = TRUE)
```

## Evaluation signals

- The markers object is a data frame whose signature genes are those meeting the chosen logFC minimum (e.g. 0.6) within each reference cluster.
- Marker lists contain both positive and negative markers when using the scID v2.0.0 (post May 17, 2019) behavior.
- Downstream DPR training-cell selection on these markers yields a median true-positive rate in the 45%–80% range and a median false-positive rate in the 0%–5% range across published reference datasets.
- scid_multiclass output includes scID_output$markers, confirming signatures were extracted from the reference clusters.
- Normalization check: input matrix values are library-depth/column normalized (TPM/CPM), not log-transformed.

## Limitations

- Marker-based identification across datasets is affected by the scRNA-seq dropout problem; scID compensates via weighted signatures but dropout remains a challenge.
- If the reference data are not well clustered with good gene signatures, scID is not a recommended approach for identification of equivalent cell types.
- No software version for scID is stated in the article sections; only the README notes the v2.0.0 release (May 17, 2019) that added negative markers.

## Evidence

- [readme] README: negative markers added in v2.0.0: [evidence span withheld — anchor status: external_doc]
