---
name: single-cell-label-mapping-across-datasets
description: Use when a target scRNA-seq dataset lacks cell labels but a reference
  dataset with known clusters exists, especially when batch effect or strong imbalance
  in cell numbers/library sizes makes integration-based or biomarker-based mapping
  unreliable.
when_to_use_negative:
- Neither the reference nor the target data is well clustered or contains good gene
  signatures — the article states scID 'is not a recommended approach for identification
  of equivalent cell types' in that case
- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM)
  normalized, not log-transformed, data
- There is no labeled reference dataset with known clusters to map against
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: 'Core method: extracts cluster-specific signatures, estimates Fisher''s-LDA-like
    gene weights, and assigns target cells to reference clusters'
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Runtime environment for the scID package, installed via devtools::install_github
  repo: https://github.com/BatadaLab/scID
- name: Seurat
  role: Listed as an Imports dependency of the scID package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:adc96494a7f0c8e21051a2f36092a8e52c4c385ff47cd220063bff5ed63d0235
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-label-mapping-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map cells in an unannotated target scRNA-seq dataset to clusters of a labeled reference dataset using scID, a Fisher's Linear Discriminant Analysis-like framework that scores target cells on weighted cluster-specific gene signatures and assigns them via a finite mixture of Gaussians, robust to batch effect.

## When to use

Use when a target scRNA-seq dataset lacks cell labels but a reference dataset with known clusters exists, especially when batch effect or strong imbalance in cell numbers/library sizes makes integration-based or biomarker-based mapping unreliable.

## When NOT to use

- Neither the reference nor the target data is well clustered or contains good gene signatures — the article states scID 'is not a recommended approach for identification of equivalent cell types' in that case
- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM) normalized, not log-transformed, data
- There is no labeled reference dataset with known clusters to map against

## Inputs

- target_gem: nxm data frame (n genes x m cells), library-depth or column-normalized (TPM/CPM), unique cell names
- reference_gem: NxM data frame (N genes x M cells), library-depth or column-normalized
- reference_clusters: list of cluster labels for reference cells
- Optional: markers data frame of cluster-specific genes (e.g. markers.rds) and/or pre-computed weights

## Outputs

- scID_output$labels: named list of cluster labels assigned to target cells
- scID_output$markers: data frame of signature genes extracted from reference clusters
- scID_output$weights: list of estimated weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores for each reference cluster-specific geneset

## How to apply

In R, normalize both datasets by library depth (TPM or CPM, e.g. via counts_to_cpm); scID does not accept log-transformed data. Stage 1: run scID::find_markers on the reference gene-expression matrix and cluster labels, using logFC=0.6 as the minimum log-fold-change for selecting cluster-specific signature genes (positive and, since scID v2.0.0, negative markers). Stage 2: estimate signature weights either from the reference or from the target — choose_training_set identifies putative true/false cells by differential precision-recall (DPR) and scID_weight estimates discriminative weights; when reference and target have strong batch effect, estimating weights from the target (estimate_weights_from_target=TRUE) is more accurate. Stage 3: score all target cells with the weighted signatures and assign them with final_populations using a finite mixture of Gaussians, resolving conflicts by relative normalized score. A single call to scid_multiclass(target_gem, reference_gem, reference_clusters, markers, logFC, ...) chains these stages.

## Related tools

- **scID** (Core method: extracts cluster-specific signatures, estimates Fisher's-LDA-like gene weights, and assigns target cells to reference clusters) — https://github.com/BatadaLab/scID
- **R** (Runtime environment for the scID package, installed via devtools::install_github) — https://github.com/BatadaLab/scID
- **Seurat** (Listed as an Imports dependency of the scID package)

## Examples

```
scID_output <- scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- scid_multiclass returns a list with exactly four components: labels, markers, weights, scores — verify all are present and non-empty
- make_heatmap of average expression of each cluster-specific geneset in reference vs target clusters should show concordant patterns across datasets
- plot_score_2D should project selected target cells for a given reference cluster onto positive- vs negative-marker score space with assigned cells separated
- Against a labeled target, stage-3 weight-based classification should show higher true-positive rate than stage-2 DPR-selected populations without increased false-positive rate (Figure 1C pattern)
- logFC threshold applied in find_markers must be respected: only genes meeting the minimum log-fold-change are retained as cluster-specific

## Limitations

- If neither dataset is well clustered or has good gene signatures, scID is not recommended
- Imputation of gene expression for shallow target data is deferred to future work and not implemented or evaluated
- Accuracy depends on normalization: log-transformed data are incompatible; any library-depth normalization (TPM, CPM) is acceptable
- Since v2.0.0 (May 17, 2019) negative markers are used alongside positive ones, which matters when very similar cell types are present; older versions may misclassify such clusters

## Evidence

- [readme] v2.0.0 uses negative markers to improve classification with very similar cell types: [evidence span withheld — anchor status: external_doc]
- [readme] scID_output is a list of four objects (labels, markers, weights, scores): [evidence span withheld — anchor status: external_doc]
