---
name: gaussian-mixture-cell-classification
description: Use when a labeled reference scRNA-seq dataset with cluster-specific
  gene signatures and weights (e.g., from scID_weight) is available and unlabeled
  target cells must be assigned to reference clusters from per-cell signature scores.
when_to_use_negative:
- Neither the reference nor the target data are well clustered or contain good gene
  signatures — scID is explicitly not recommended for identifying equivalent cell
  types in that case
- The expression matrices are log-transformed rather than library-depth (TPM/CPM)
  normalized
- The goal is batch correction/integration of two datasets rather than label transfer
  via reference cluster signatures
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: Provides final_populations (Gaussian-mixture assignment of scored target cells),
    scID_weight/choose_training_set for weight estimation, and scid_multiclass to
    run the full three-stage mapping
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Execution environment for the scID package functions (R, with Seurat as an
    import)
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:18e515e829892a0293ed974cfee5544df2e80d994fd6e17b14e3b70ee748d226
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  repo_url: https://github.com/BatadaLab/scID
  tools:
  - R
  - Seurat
  - scID
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gaussian-mixture-cell-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage 3 of the scID pipeline: score target cells with weighted cluster-specific gene signatures and assign them to reference clusters by fitting a finite mixture of Gaussian distributions to the score distribution. This converts discriminant scores into hard cell-type labels with conflict resolution.

## When to use

Use when a labeled reference scRNA-seq dataset with cluster-specific gene signatures and weights (e.g., from scID_weight) is available and unlabeled target cells must be assigned to reference clusters from per-cell signature scores.

## When NOT to use

- Neither the reference nor the target data are well clustered or contain good gene signatures — scID is explicitly not recommended for identifying equivalent cell types in that case
- The expression matrices are log-transformed rather than library-depth (TPM/CPM) normalized
- The goal is batch correction/integration of two datasets rather than label transfer via reference cluster signatures

## Inputs

- Data frame of target-cell scores for each reference cluster-specific geneset (rows = cluster signatures, columns = cells), as produced by scID_weight/scid_multiclass
- Weighted cluster-specific gene signatures extracted with find_markers (logFC threshold, e.g. 0.6, only_pos = FALSE to include downregulated genes)
- TPM/CPM-normalized target and reference gene expression matrices (genes x cells, non-log-transformed)

## Outputs

- Named list of cluster labels for all target cells (scID_output$labels)
- Cluster assignment populations obtained from final_populations(score), usable for TPR/FPR evaluation against known target clusters

## How to apply

First compute, for every target cell, a normalized score against each reference cluster's signature using the Fisher-LDA-like gene weights estimated in stage 2 (scID_weight on putative positive/negative training cells from choose_training_set). Then apply scID::final_populations to the score matrix: model the scores of target cells with a finite mixture of Gaussians and assign cells with high likelihood of falling under the distribution with the highest average score to that cluster. Resolve conflicts when a cell scores high for multiple clusters using the relative normalized score strategy. Inputs must be library-depth normalized (TPM/CPM), not log-transformed, and weights may come from the reference or the target (target-side weights are preferred under strong batch effect). Validate assignments by comparing make_heatmap patterns of cluster-specific genesets between reference and target clusters.

## Related tools

- **scID** (Provides final_populations (Gaussian-mixture assignment of scored target cells), scID_weight/choose_training_set for weight estimation, and scid_multiclass to run the full three-stage mapping) — https://github.com/BatadaLab/scID
- **R** (Execution environment for the scID package functions (R, with Seurat as an import))

## Examples

```
scID_output <- scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, only_pos = FALSE, normalize_reference = FALSE, estimate_weights_from_target = FALSE); labels <- scID_output$labels
```

## Evaluation signals

- scid_multiclass returns a list with $labels, $markers, $weights, and $scores; $labels must cover all target cells with unique cell names
- On the Shekhar et al. mouse retinal bipolar Drop-seq benchmark, stage-3 (weight-based, final_populations) assignments must show a higher true-positive rate than stage-2 DPR-selected populations (choose_training_set) with no increase in false-positive rate, matching Figure 1C
- make_heatmap of cluster-specific geneset average expression in target clusters should resemble the reference heatmap pattern
- In the 10X neuron_9k vs nuclei_900 (E18 mouse brain) vignette, no target cells should be assigned to reference clusters 5 and 13 (gray columns)

## Limitations

- If neither dataset is well clustered and contains good gene signatures, scID (and hence this Gaussian-mixture assignment step) is not a recommended approach
- Weights estimated from the target data appear more accurate when reference and target have strong batch effect; reference-side weights are used when library depths are similar
- Imputation of gene expression for highly shallow target data is deferred to future work and is not implemented or evaluated
- Distance metrics / biomarker-only matching in high-dimensional space can fail due to dropout, which is why weighted signatures plus mixture modeling are used instead

## Evidence
