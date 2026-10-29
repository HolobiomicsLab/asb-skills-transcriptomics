---
name: single-cell-rna-seq-dataset-mapping
description: Use when two library-depth-normalized scRNA-seq expression matrices (e.g.
  TPM/CPM gene-by-cell data frames) exist, one with known reference clusters, and
  target cells must be assigned to equivalent reference cell types, especially with
  batch effect or imbalanced cell numbers/library sizes.
when_to_use_negative:
- When neither dataset is well clustered and contains good gene signatures — the article
  states scID 'is not a recommended approach for identification of equivalent cell
  types' in this case
- When the input expression matrices are log-transformed — scID is compatible only
  with library-depth normalization (TPM/CPM), not log-transformed data
- 'When the goal is explicit batch correction/integration of the datasets themselves
  rather than label transfer: scID identifies equivalent cells without assumptions
  about technical and biological variation and does not impute or over-correct expression'
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: 'Core method: Fisher''s Linear Discriminant Analysis-like framework for identifying
    transcriptionally equivalent cell types between scRNA-seq datasets; provides scid_multiclass,
    find_markers, scID_weight, make_heatmap'
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Runtime environment for the scID package and vignette (rmarkdown::html_vignette)
- name: Seurat
  role: Imported dependency of the scID R package
- name: mclust
  role: Provides adjustedRandIndex for evaluating mapping accuracy against published
    cell labels in self-mapping tests
- name: devtools
  role: Used to install scID from GitHub (devtools::install_github("BatadaLab/scID"))
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:8352fec37048ff61929756f2d242f35b591860d360a0a4d4a4baaa44816dfe11
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
  - devtools
  - mclust
  - scID
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rna-seq-dataset-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map cells from a target scRNA-seq dataset to reference clusters using scID's Fisher's Linear Discriminant Analysis-like framework, which extracts cluster-specific gene signatures, weights them by discriminative power, and assigns equivalent target cells via a finite mixture of Gaussians — robust to batch effect without explicit data integration.

## When to use

Use when two library-depth-normalized scRNA-seq expression matrices (e.g. TPM/CPM gene-by-cell data frames) exist, one with known reference clusters, and target cells must be assigned to equivalent reference cell types, especially with batch effect or imbalanced cell numbers/library sizes.

## When NOT to use

- When neither dataset is well clustered and contains good gene signatures — the article states scID 'is not a recommended approach for identification of equivalent cell types' in this case
- When the input expression matrices are log-transformed — scID is compatible only with library-depth normalization (TPM/CPM), not log-transformed data
- When the goal is explicit batch correction/integration of the datasets themselves rather than label transfer: scID identifies equivalent cells without assumptions about technical and biological variation and does not impute or over-correct expression

## Inputs

- target_gem: n genes x m cells data frame of library-depth (column) normalized expression with unique cell names
- reference_gem: N genes x M cells data frame of normalized expression with known grouping
- reference_clusters: list of cluster labels for the reference cells
- Pre-computed TPM-normalized example files (target_gem.rds, reference_gem.rds, reference_clusters.rds) from github.com/BatadaLab/scID/tree/master/ExampleData
- Optionally a .gmt gene set file via gmt_to_markers()
- Raw count matrices normalized with counts_to_cpm()/normalize_gene()

## Outputs

- scID_output$labels: named list of cluster labels for all target cells (exportable as scid_labels.csv)
- scID_output$markers: data frame of signature genes extracted from reference clusters
- scID_output$weights: list of estimated weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores for each reference cluster-specific geneset
- make_heatmap visualizations of cluster-specific geneset expression in reference and target clusters
- plot_score_2D projection of selected target cells onto positive/negative marker scores
- Adjusted Rand Index values (via mclust::adjustedRandIndex) when mapping accuracy is evaluated against published labels (e.g. ari_self_mapping_table.csv)

## How to apply

Load scID in R and read target_gem, reference_gem, and reference_clusters (gene-by-cell data frames; TPM/CPM library-depth normalized, NOT log-transformed). Run scid_multiclass with logFC = 0.6 as the minimum log-fold change for cluster-specific genes and only_pos = FALSE so downregulated cluster-specific genes are retained (scID v2.0.0 uses negative markers with positive to improve classification when very similar cell types are present). Set normalize_reference = FALSE if the reference is already normalized, and decide the key parameter estimate_weights_from_target: weights estimated from the target appear more accurate under strong batch effect, whereas weights from the reference suffice when datasets have similar library depths (as in the 10X E18 mouse brain neuron_9k vs nuclei_900 vignette). Internally, stage 1 extracts cluster-specific gene signatures, stage 2 estimates Fisher-LDA-like gene weights via choose_training_set and scID_weight using differential precision-recall, and stage 3 scores target cells and assigns clusters with final_populations (finite mixture of Gaussians). Validate with make_heatmap: the heatmap of cluster-specific geneset expression in target clusters should resemble the reference heatmap pattern; also inspect scID_output$labels, $markers, $weights, and $scores.

## Related tools

- **scID** (Core method: Fisher's Linear Discriminant Analysis-like framework for identifying transcriptionally equivalent cell types between scRNA-seq datasets; provides scid_multiclass, find_markers, scID_weight, make_heatmap) — https://github.com/BatadaLab/scID
- **R** (Runtime environment for the scID package and vignette (rmarkdown::html_vignette))
- **Seurat** (Imported dependency of the scID R package)
- **mclust** (Provides adjustedRandIndex for evaluating mapping accuracy against published cell labels in self-mapping tests)
- **devtools** (Used to install scID from GitHub (devtools::install_github("BatadaLab/scID")))

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, only_pos = FALSE, normalize_reference = FALSE, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- make_heatmap of average cluster-specific geneset expression in target clusters should resemble the reference heatmap pattern; e.g. in the vignette no target cells should be assigned to reference clusters 5 and 13 (gray columns)
- All target cells receive a label in scID_output$labels with names matching target_gem cell names
- In self-mapping (same dataset as reference and target), Adjusted Rand Index between scID_output$labels and published labels should be relatively high and close to 1 (Figure 1D)
- scID_output$scores must contain one row per reference cluster-specific geneset and one column per target cell
- Input matrices must be library-depth normalized (TPM/CPM), not log-transformed — verify before running

## Limitations

- If neither dataset is well clustered and contains good gene signatures, scID is not recommended
- When reference and target have strong batch effect, estimation of weights from the target appears more accurate — the choice of estimate_weights_from_target matters and defaults can mislead
- Imputation of gene expression for shallow target data is deferred to future work and is not implemented or evaluated
- Biomarker-based identification alone fails owing to the dropout problem in scRNA-seq data; scID's gene weighting mitigates but dropout still affects shallow data
- scID v2.0.0+ behavior differs from earlier releases: negative markers are used together with positive markers since May 17, 2019

## Evidence

- [readme] v2.0.0 negative markers: [evidence span withheld — anchor status: external_doc]
