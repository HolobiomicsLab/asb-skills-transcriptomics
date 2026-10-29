---
name: scrna-pseudobulk-cluster-aggregation
description: Use when scRNA-seq data stored as a Seurat object must be collapsed into
  pooled, non-sparse per-cluster transcriptomic profiles before applying a predictor
  (e.g., miRNA inference) to each cell type.
when_to_use_negative:
- When per-cell resolution of miRNA activity is required — the article notes miReact's
  authors 'interpreted the predicted miRNA at the level of cell clusters', and pooling
  deliberately collapses single cells.
- When the sample context does not match the training cohort — the article warns accuracy
  is reduced 'when the cellular/tissue context of the application sample does not
  perfectly match those of the training cohort'.
- When genes are largely undetected due to dropout — the article states 'a pre-trained
  model may not work and miRSCAPE needs to be retrained using only the genes that
  are detected at a reasonable level'.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: miRSCAPE
  role: Downstream XGBoost regression tool applied to the pooled transcriptomic profiles
    to infer miRNA activity per cluster
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Data container and preparation layer; modifySeuratObject pools cells within
    specified clusters from a Seurat object
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying regression engine (default booster, objective, max depth, eta,
    rounds) used by miRSCAPE on pooled profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Required runtime for running modifySeuratObject, bulkTransform, and miRSCAPE
  repo: https://github.com/hannenhalli-lab/miRSCAPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:9406a6a8b52c1fa6782d1f4f3279af417121520ecd4392eb7605ddec6e334e15
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hannenhalli-lab/miRSCAPE
  repo_url: https://github.com/hannenhalli-lab/miRSCAPE
  tools:
  - R
  - R 4.1.0
  - Seurat
  - miRSCAPE
  - xgboost
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-pseudobulk-cluster-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates cells within each single-cell cluster into a non-sparse pseudo-bulk mRNA profile (via Seurat-object manipulation and bootstrapped cell pooling) so that downstream regression models such as miRSCAPE can be applied at the cell-type level rather than on sparse individual cells.

## When to use

Use when scRNA-seq data stored as a Seurat object must be collapsed into pooled, non-sparse per-cluster transcriptomic profiles before applying a predictor (e.g., miRNA inference) to each cell type.

## When NOT to use

- When per-cell resolution of miRNA activity is required — the article notes miReact's authors 'interpreted the predicted miRNA at the level of cell clusters', and pooling deliberately collapses single cells.
- When the sample context does not match the training cohort — the article warns accuracy is reduced 'when the cellular/tissue context of the application sample does not perfectly match those of the training cohort'.
- When genes are largely undetected due to dropout — the article states 'a pre-trained model may not work and miRSCAPE needs to be retrained using only the genes that are detected at a reasonable level'.

## Inputs

- Seurat object of scRNA-seq data with gene names as row names (e.g., HCL brain and kidney profiles)
- Cell cluster/cell-type identifiers (clusterId parameter, e.g., brain and kidney cell types)
- Paired bulk mRNA matrix (rows = genes, columns = samples) for model training context
- Paired bulk miRNA matrix (rows = miRNAs, columns = samples)

## Outputs

- Non-sparse pooled (pseudo-bulk) mRNA profiles per cell cluster
- Predicted miRNA expression per cluster, available in the pred output element of miRSCAPE

## How to apply

Load the scRNA-seq data (e.g., HCL brain/kidney profiles) as a Seurat object with gene names as row names, then run modifySeuratObject with the clusterId parameter set to the cell types of interest (or omit clusterId to pool all clusters). Pooling — including bootstrapped sampling of cells within each cluster — converts the sparse per-cell matrix into a non-sparse pseudo-bulk mRNA profile, which is required because regression models like miRSCAPE were trained on non-sparse bulk cohorts. If the pooled profiles will be compared against bulk-trained predictions, prepare the paired bulk mRNA data with bulkTransform (log-transform, scaled) and bulk miRNA data with justNorm = TRUE so the miRNA matrix is only log-transformed and not scaled. Feed the pooled scRNA matrix into miRSCAPE together with the transformed bulk mRNA/miRNA training matrices and retrieve predicted miRNA expression from the pred output element.

## Related tools

- **miRSCAPE** (Downstream XGBoost regression tool applied to the pooled transcriptomic profiles to infer miRNA activity per cluster) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Data container and preparation layer; modifySeuratObject pools cells within specified clusters from a Seurat object) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying regression engine (default booster, objective, max depth, eta, rounds) used by miRSCAPE on pooled profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Required runtime for running modifySeuratObject, bulkTransform, and miRSCAPE) — https://github.com/hannenhalli-lab/miRSCAPE

## Examples

```
denem = modifySeuratObject(pbmc = example, clusterId = c('Ductal cell type 1'))
```

## Evaluation signals

- The pooled per-cluster mRNA matrix is non-sparse (cells within each cluster have been pooled/bootstrapped before prediction).
- For a brain-vs-kidney application, Spearman rho between predicted and observed inter-cell-type miRNA fold-changes across 262 miRNAs should reach ~0.8 with tissue-matched models (paper benchmark ~0.81).
- Using a single pooled cross-tissue model instead of cell-type-specific models should yield a lower but comparable rho (~0.73 in the HCL brain/kidney validation).
- Predicted values are retrievable from the pred element of the miRSCAPE output for every specified cluster.
- Bulk mRNA was transformed with bulkTransform (scaled) and bulk miRNA with justNorm = TRUE (log-transform only, not scaled), matching the README's data-preparation rules.

## Limitations

- miRSCAPE accuracy is expected to be slightly reduced when the cellular/tissue context of the application sample does not match the training cohort, and a substantive loss occurs when training context differs substantially (quantified in Figure S9).
- Due to the large dropout rate of scRNA-seq, a pre-trained model may not work and retraining restricted to reasonably detected genes may be required.
- miRSCAPE must be trained on a large cohort of paired miRNA-mRNA data, unlike target-based approaches that need no training.

## Evidence
