---
name: discriminant-analysis-cell-classification
description: Use when a target scRNA-seq dataset lacks cell-type labels while a reference
  dataset with known clusters exists, especially when the two differ in protocol,
  batch effect, cell number, or library depth (e.g., Smart-Seq2 reference mapped to
  CEL-Seq1/CEL-Seq2 targets).
when_to_use_negative:
- Neither dataset is well clustered or contains good gene signatures — the article
  states scID 'is not a recommended approach for identification of equivalent cell
  types' in that case
- Input expression matrices are log-transformed — scID requires library-depth-normalized
  (TPM/CPM), non-log data
- The target dataset is highly shallow and requires imputation — the article defers
  imputation of gene expression to future work and does not implement or evaluate
  it
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: 'Primary method: extracts reference-cluster gene signatures, estimates FLD-based
    weights, and assigns target cells via scid_multiclass'
  repo: github.com/BatadaLab/scID
- name: scmap
  role: Competing reference-mapping method used as a benchmark for classification
    accuracy
- name: CaSTLe
  role: Competing classification method used as a benchmark for classification accuracy
- name: scVI
  role: Competing deep-learning integration method used as a benchmark for classification
    accuracy
- name: R
  role: Implementation environment; scID is installed from GitHub via devtools and
    run in R/Rmarkdown
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:555d233568dedfe409d2eeccf1a5536aca830a3c433f64236d616aba7d42069b
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
  - scID
  - scVI
  - scmap
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# discriminant-analysis-cell-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reference-based cell-type classification across scRNA-seq datasets using scID, a Fisher's Linear Discriminant Analysis-like framework that scores target cells on weighted cluster-specific gene signatures and assigns them via a finite mixture of Gaussians, robust to batch effect and imbalanced library sizes.

## When to use

Use when a target scRNA-seq dataset lacks cell-type labels while a reference dataset with known clusters exists, especially when the two differ in protocol, batch effect, cell number, or library depth (e.g., Smart-Seq2 reference mapped to CEL-Seq1/CEL-Seq2 targets).

## When NOT to use

- Neither dataset is well clustered or contains good gene signatures — the article states scID 'is not a recommended approach for identification of equivalent cell types' in that case
- Input expression matrices are log-transformed — scID requires library-depth-normalized (TPM/CPM), non-log data
- The target dataset is highly shallow and requires imputation — the article defers imputation of gene expression to future work and does not implement or evaluate it

## Inputs

- target_gem: n genes x m cells data frame of library-depth-normalized (non-log) expression for the unlabeled target dataset with unique cell names
- reference_gem: N genes x M cells data frame of library-depth-normalized (non-log) expression for the labeled reference dataset
- reference_clusters: list of cluster labels for the reference cells (e.g., published pancreas cell-type labels from Segerstolpe Smart-Seq2)
- markers: optional data frame of cluster-specific signature genes (e.g., a .rds markers file or gmt file parsed with gmt_to_markers)

## Outputs

- scID_output$labels: named list of predicted cluster labels for target cells
- scID_output$markers: data frame of signature genes extracted from reference clusters
- scID_output$weights: list of estimated weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores per reference cluster-specific geneset
- Per-method accuracy table against published labels when benchmarked against scmap, CaSTLe, and scVI

## How to apply

Normalize both count matrices with library-depth normalization (TPM or CPM), keeping data non-log-transformed since scID does not accept log-transformed data. Extract cluster-specific gene signatures from the reference clusters with find_markers, using a logFC threshold to define cluster-specific genes. Estimate discriminative gene-signature weights (via differential precision-recall, identifying putative positive and negative cells with choose_training_set and scID_weight); under strong batch effect, set estimate_weights_from_target=TRUE so weights are estimated from the target data, which the article reports as more accurate when reference and target differ strongly in library depth and protocol. Run scid_multiclass to score target cells and assign each to the reference cluster whose Gaussian mixture component has the highest average score. Optionally visualize agreement with make_heatmap and inspect assignments with plot_score_2D.

## Related tools

- **scID** (Primary method: extracts reference-cluster gene signatures, estimates FLD-based weights, and assigns target cells via scid_multiclass) — github.com/BatadaLab/scID
- **scmap** (Competing reference-mapping method used as a benchmark for classification accuracy)
- **CaSTLe** (Competing classification method used as a benchmark for classification accuracy)
- **scVI** (Competing deep-learning integration method used as a benchmark for classification accuracy)
- **R** (Implementation environment; scID is installed from GitHub via devtools and run in R/Rmarkdown)

## Examples

```
devtools::install_github("BatadaLab/scID"); scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- Mapping accuracy against published cell-type labels ranks scID within the top two methods for both accuracy and consistency across target datasets (as in Figure 1E for Baron CEL-Seq1 and Muraro CEL-Seq2 targets)
- scid_multiclass returns a list with all four objects (labels, markers, weights, scores); labels must cover the target cells with unique cell names
- Input matrices are confirmed column/library-depth normalized and not log-transformed
- Heatmap of average expression of cluster-specific genesets across reference and target clusters shows matching clusters with aligned expression patterns
- Under strong batch effect, weights were estimated from the target data (estimate_weights_from_target=TRUE), consistent with the article's guidance

## Limitations

- If neither dataset is well clustered and contains good gene signatures, scID is not a recommended approach
- Imputation of gene expression for highly shallow target data may increase power but is not implemented or evaluated (deferred to future work)
- Log-transformed data are incompatible; only library-depth normalization (TPM, CPM) is supported
- Biomarker-based identification alone is ineffective due to dropout; scID mitigates but relies on quality cluster-specific signatures

## Evidence

- [other] Pancreas benchmark outcome: [evidence span withheld — anchor status: not_found]
