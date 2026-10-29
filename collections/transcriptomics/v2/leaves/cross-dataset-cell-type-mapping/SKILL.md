---
name: cross-dataset-cell-type-mapping
description: Use when a labeled reference scRNA-seq dataset and an unlabeled target
  dataset (e.g., Drop-seq reference to Smart-seq2 target, or Smart-Seq2 to CEL-Seq
  pancreas) must be mapped across technologies or batch conditions.
when_to_use_negative:
- Neither dataset is well clustered or contains good gene signatures — the article
  states scID 'is not a recommended approach' in that case
- Only top biomarker genes with simple per-gene thresholds are available — dropout
  makes such marker-threshold baselines assign most cells as Orphans
- The goal is batch-corrected integration of similarly sized, well-matched datasets
  where batch-correction methods (e.g., scVI) may suffice
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: 'core method: find_markers signature extraction, choose_training_set/scID_weight
    DPR weighting, scid_multiclass assignment'
  repo: github.com/BatadaLab/scID
- name: scmap
  role: competing reference-mapping method; extracts features from each reference
    cluster and leaves a fraction of cells unassigned
- name: CaSTLe
  role: competing method using genes with high expression in both target and reference
    or high mutual information with cluster identities
- name: scVI
  role: competing batch-integration method used in cross-platform pancreas benchmarking
- name: R
  role: implementation environment; scID installed via devtools::install_github('BatadaLab/scID')
  repo: github.com/BatadaLab/scID
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:6768d3240bc276d93aec907fe47a920463060ee3643543886f3b46833403b9b8
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
  - data.table
  - ggplot2
  - scID
  - scVI
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-dataset-cell-type-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign cell-type labels to a target scRNA-seq dataset using a labeled reference dataset via scID, a Fisher's Linear Discriminant Analysis-like framework that extracts cluster-specific gene signatures, estimates discriminative weights, and assigns target cells with a finite mixture of Gaussians—robust to batch effect and strong imbalance in cell number and library size.

## When to use

Use when a labeled reference scRNA-seq dataset and an unlabeled target dataset (e.g., Drop-seq reference to Smart-seq2 target, or Smart-Seq2 to CEL-Seq pancreas) must be mapped across technologies or batch conditions.

## When NOT to use

- Neither dataset is well clustered or contains good gene signatures — the article states scID 'is not a recommended approach' in that case
- Only top biomarker genes with simple per-gene thresholds are available — dropout makes such marker-threshold baselines assign most cells as Orphans
- The goal is batch-corrected integration of similarly sized, well-matched datasets where batch-correction methods (e.g., scVI) may suffice

## Inputs

- target_gem: n genes x m cells library-depth (TPM/CPM) normalized expression matrix, non-log-transformed
- reference_gem: N genes x M cells normalized expression matrix from the labeled reference dataset
- reference_clusters: list of cluster labels for the reference cells
- cluster-specific marker lists (from find_markers with logFC 0.6 or precomputed markers.rds)

## Outputs

- scID_output$labels: named list of cluster labels assigned to target cells
- scID_output$markers: data frame of signature genes extracted from reference clusters
- scID_output$weights: list of estimated weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores for each reference cluster geneset
- per-method coverage/ARI/VI summary tables and heatmaps of average geneset expression per cluster (make_heatmap)

## How to apply

First library-depth normalize both datasets (TPM or CPM, NOT log-transformed) and supply reference cluster labels. Extract cluster-specific gene signatures with scID::find_markers using a minimum log-fold-change threshold of logFC = 0.6. Then run scid_multiclass; when reference and target differ strongly in library depth or protocol (e.g., Smart-Seq2 reference vs CEL-Seq1/CEL-Seq2 target), set estimate_weights_from_target = TRUE so discriminative gene weights are estimated from the target by selecting putative c+ and c- training cells via the differential precision-recall (DPR) procedure (choose_training_set) and scID_weight. Target-cell scores are modeled with a finite mixture of Gaussians and cells with high likelihood under the highest-average-score distribution are assigned, with conflicts resolved by a relative normalized score strategy. Compare against competing methods scmap and CaSTLe (and optionally scVI) using coverage, Adjusted Rand Index, and Variation of Information.

## Related tools

- **scID** (core method: find_markers signature extraction, choose_training_set/scID_weight DPR weighting, scid_multiclass assignment) — github.com/BatadaLab/scID
- **scmap** (competing reference-mapping method; extracts features from each reference cluster and leaves a fraction of cells unassigned)
- **CaSTLe** (competing method using genes with high expression in both target and reference or high mutual information with cluster identities)
- **scVI** (competing batch-integration method used in cross-platform pancreas benchmarking)
- **R** (implementation environment; scID installed via devtools::install_github('BatadaLab/scID')) — github.com/BatadaLab/scID

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- Median true-positive rate across reference clusters falls in the reported 45%-80% range and median false-positive rate in the 0%-5% range for DPR training-cell selection
- On Drop-seq (reference, ~26,800 cells, 15 clusters) to Smart-seq2 (target, 288 cells): scID coverage must be 100% (288/288 cells assigned into 15 clusters) vs scmap 63.2% (182/288 into 10 clusters)
- ARI and VI on target labels must reproduce the relative accuracy ordering of Figures 2D-2E
- In cross-platform pancreas mapping (Smart-Seq2 reference to Baron CEL-Seq1 / Muraro CEL-Seq2 targets), scID must rank within the top two methods in accuracy and consistency
- make_heatmap of average expression of each cluster-specific geneset per cluster should show concordant reference-target cluster pairs

## Limitations

- If neither dataset is well clustered and contains good gene signatures, scID is not recommended for identification of equivalent cell types
- Distance metrics in high-dimensional space may not work as expected for reference-based cell mapping
- Imputation of gene expression for highly shallow target data is deferred to future work and not implemented or evaluated
- When reference and target have strong batch effect, estimating weights from the target is more accurate; using reference weights in that setting underperforms
