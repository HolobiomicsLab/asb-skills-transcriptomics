---
name: fisher-lda-gene-weight-estimation
description: Use when a reference scRNA-seq dataset has cluster-specific gene signatures
  and you need to weight those genes by discriminability before scoring target cells,
  especially with batch effect or imbalanced library sizes.
when_to_use_negative:
- Neither the reference nor the target data is well clustered or contains good gene
  signatures — scID is not a recommended approach for identification of equivalent
  cell types
- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM)
  normalization, not log-transformed data
- You need imputation for highly shallow target data — this is deferred to future
  work and not implemented in scID
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scID
  role: Provides choose_training_set, scID_weight, find_markers and scid_multiclass
    functions implementing the Fisher-LDA-like weight estimation (stage 2) within
    the three-stage mapping pipeline
  repo: github.com/BatadaLab/scID
- name: R
  role: Execution environment for the scID R package; installed via devtools::install_github('BatadaLab/scID')
  repo: github.com/BatadaLab/scID
- name: Seurat
  role: Listed as an Imports dependency of the scID package
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:44609c2ed8d190f5c1c4bfe55db039781e623aa23c61dfd7a129da07043dd48f
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
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fisher-lda-gene-weight-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage 2 of the scID pipeline: estimate discriminative weights for reference-cluster gene signatures using a Fisher's Linear Discriminant Analysis-like framework, by first selecting putative positive/negative training populations via differential precision-recall (DPR) and then fitting gene weights with scID_weight.

## When to use

Use when a reference scRNA-seq dataset has cluster-specific gene signatures and you need to weight those genes by discriminability before scoring target cells, especially with batch effect or imbalanced library sizes.

## When NOT to use

- Neither the reference nor the target data is well clustered or contains good gene signatures — scID is not a recommended approach for identification of equivalent cell types
- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM) normalization, not log-transformed data
- You need imputation for highly shallow target data — this is deferred to future work and not implemented in scID

## Inputs

- TPM- or CPM-normalized (library-depth normalized, not log-transformed) gene expression matrix (gem) as genes-rows x cells-columns data frame
- Cluster-specific positive and negative marker lists extracted from reference clusters (e.g. via find_markers with logFC = 0.6)
- Putative true_cells and false_cells training populations produced by choose_training_set

## Outputs

- A list of estimated weights for all cluster-specific genes (scID_output$weights) representing each gene's discriminative power
- Putative positive (true_cells) and negative (false_cells) training populations identified by DPR in choose_training_set

## How to apply

After extracting cluster-specific signatures in stage 1 (e.g. find_markers with logFC = 0.6 on TPM/CPM-normalized, non-log-transformed data), run choose_training_set(gem, positive_markers, negative_markers) to identify putative positive and negative training populations by differential precision-recall, then scID_weight(gem, true_cells, false_cells) to fit the weights. Decide whether weights are estimated from the reference or the target: use estimate_weights_from_target = FALSE when the datasets' library depths are similar, and TRUE under strong batch effect, since estimating weights from the target appears more accurate in that case. In scid_multiclass, pass these weights (or let the function estimate them) and optionally include downregulated cluster-specific genes via only_pos = FALSE. The resulting weights scale each gene's contribution to the stage-3 target-cell scores, which are then modeled with a finite mixture of Gaussians in final_populations.

## Related tools

- **scID** (Provides choose_training_set, scID_weight, find_markers and scid_multiclass functions implementing the Fisher-LDA-like weight estimation (stage 2) within the three-stage mapping pipeline) — github.com/BatadaLab/scID
- **R** (Execution environment for the scID R package; installed via devtools::install_github('BatadaLab/scID')) — github.com/BatadaLab/scID
- **Seurat** (Listed as an Imports dependency of the scID package)

## Examples

```
true_cells <- scID::choose_training_set(gem, positive_markers, negative_markers); weights <- scID::scID_weight(gem, true_cells, false_cells)
```

## Evaluation signals

- choose_training_set must return distinguishable putative positive and negative training populations from the positive_markers and negative_markers of a single reference cluster
- scID_output$weights must be a list containing a weight value for every gene in each cluster-specific signature (the scID_output$markers genes)
- Downstream stage-3 classification should achieve a higher true-positive rate than the stage-2 DPR-selected populations with no increase in false-positive rate (as reported for mouse retinal bipolar Drop-seq data, Figure 1C pattern)
- make_heatmap of cluster-specific geneset expression in target clusters should resemble the reference heatmap pattern, confirming weights produced discriminative features
- With estimate_weights_from_target = FALSE, weights must be derived from the already-normalized reference; verify the reference was not re-normalized (normalize_reference = FALSE)

## Limitations

- When the reference and the target data have strong batch effect, weights estimated from the target appear more accurate; estimating from the reference can mislead under strong batch effect
- If neither dataset is well clustered or contains good gene signatures, scID (and hence weight estimation) is not recommended
- Biomarker/dropout sparsity in scRNA-seq data limits naive unweighted signature use; imputation for shallow target data is not implemented and deferred to future work

## Evidence
