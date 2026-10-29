---
name: dpr-training-cell-selection
description: Use when classifying target scRNA-seq cells against reference clusters
  with scID and you need to select c+/c- training cells and estimate gene signature
  weights, especially when data have batch effect.
when_to_use_negative:
- The reference and target data lack good clustering and well-defined gene signatures
  — the article states scID 'is not a recommended approach' in that case
- Input gene expression matrices are log-transformed — scID requires library-depth
  (TPM/CPM) normalized data, 'but not log-transformed data'
- The goal is cell-type discovery without a labeled reference; scID/DPR training-cell
  selection presupposes reference clusters with known labels
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: R package providing choose_training_set (DPR-based c+/c- training cell selection)
    and scID_weight (Fisher's Linear Discriminant Analysis-like gene weight estimation)
  repo: github.com/BatadaLab/scID
- name: R
  role: Runtime environment for the scID package workflow and evaluation of per-cluster
    TPR/FPR
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:3c87bd637787c3a95ea8b01737f85c1c40a77ec16f09354ba33c27f29433fab9
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
  - scID
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# dpr-training-cell-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage-2 of the scID pipeline: identify putative positive (c+) and negative (c-) training cells via differential precision-recall (DPR) and fit discriminative Fisher's Linear Discriminant Analysis-like gene signature weights from them. This bridges reference-cluster gene signatures and target-cell scoring in scID.

## When to use

Use when classifying target scRNA-seq cells against reference clusters with scID and you need to select c+/c- training cells and estimate gene signature weights, especially when data have batch effect.

## When NOT to use

- The reference and target data lack good clustering and well-defined gene signatures — the article states scID 'is not a recommended approach' in that case
- Input gene expression matrices are log-transformed — scID requires library-depth (TPM/CPM) normalized data, 'but not log-transformed data'
- The goal is cell-type discovery without a labeled reference; scID/DPR training-cell selection presupposes reference clusters with known labels

## Inputs

- TPM- or CPM-normalized (non-log) gene expression matrix (gem) with genes as rows and cells as columns (reference_gem.rds / target GEM)
- Cluster labels for reference cells (reference_clusters.rds)
- Cluster-specific positive and negative marker lists from find_markers or precomputed markers.rds (logFC=0.6)

## Outputs

- Putative c+ (true) and c- (false) training cell sets per reference cluster from choose_training_set
- List of discriminative gene signature weights from scID_weight (scID_output$weights)
- Per-cluster TPR/FPR summary table (e.g. dpr_weight_summary.csv, tpr_fpr_comparison.csv)

## How to apply

After extracting cluster-specific signatures with find_markers (logFC=0.6 minimum log-fold-change), call scID::choose_training_set(gem, positive_markers, negative_markers) to select putative c+ and c- training cells using the differential precision-recall (DPR) procedure on the gene expression matrix. Then fit discriminative weights with scID::scID_weight(gem, true_cells, false_cells); weights may be estimated from the reference or from the target data (estimate_weights_from_target=TRUE), with target-derived weights appearing more accurate under strong batch effect. The weighted signatures feed stage 3 (final_populations / scid_multiclass), where a finite mixture of Gaussians assigns target cells. Expected DPR accuracy: median true-positive rate 45%-80% and median false-positive rate 0%-5% per reference dataset; the weight-based (stage 3) classification should raise TPR without increasing FPR versus DPR selection alone.

## Related tools

- **scID** (R package providing choose_training_set (DPR-based c+/c- training cell selection) and scID_weight (Fisher's Linear Discriminant Analysis-like gene weight estimation)) — github.com/BatadaLab/scID
- **R** (Runtime environment for the scID package workflow and evaluation of per-cluster TPR/FPR)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, markers = markers, logFC = 0.6, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- Per reference dataset, the median true-positive rate of DPR-selected training cells falls in the reported 45%-80% range
- Per reference dataset, the median false-positive rate falls in the reported 0%-5% range
- On the mouse retinal bipolar Drop-seq data, stage-3 weight-based classification shows a higher TPR than stage-2 DPR selection with no FPR increase (matches Figure 1C)
- scID_weight output weights cover all cluster-specific genes and are carried through into scID_output$weights from scid_multiclass

## Limitations

- Weights estimated from the target data are preferred only when reference and target share strong batch effect; otherwise estimation source must be chosen deliberately
- The DPR approach alone achieves only moderate TPR (45%-80% median); relying on stage-2 populations without stage-3 weighted scoring under-classifies true cluster members
- scID cannot rescue datasets where neither reference nor target is well clustered with good gene signatures

## Evidence

- [readme] v2.0 uses negative markers: [evidence span withheld — anchor status: external_doc]
