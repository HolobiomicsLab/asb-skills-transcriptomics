---
name: pooled-versus-single-cell-prediction-strategy
description: Use when predicting miRNA activity from scRNA-seq data where single-cell
  profiles are too sparse for a pretrained bulk-trained model, and you want to contrast
  pooled-cluster predictions with per-cell predictions.
when_to_use_negative:
- The input is already bulk (non-sparse) RNA-seq data — pooling is unnecessary; apply
  the miRSCAPE model directly in bulk mode
- The application sample's cellular/tissue context differs substantially from the
  model's training cohort, where the article expects 'a more substantive loss in performance'
  — retraining on detected genes may be needed instead
- The analysis goal is miRNA target activity from putative-target lists alone without
  a trained model — use the target-only control approach (e.g. miReact) rather than
  pooled miRSCAPE
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: xgboost-based machine learning model applied in pooled-cluster and single-cell
    modes to predict miRNA expression from genome-wide mRNA profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: miReact
  role: cross-gene-correlation-based target-only method run as a control on the same
    induced/uninduced profiles
- name: Seurat
  role: reads and prepares the scRNA-seq Seurat object via modifySeuratObject before
    prediction
- name: xgboost
  role: R dependency providing the boosted-tree backend for the pretrained miRSCAPE
    models
- name: R 4.1.0
  role: runtime environment required to run the miRSCAPE pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:dfb9c45a43d00cb1b0d0b6d4cc8948ff34e99dd0b12996b509c68c95ee5e301b
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
  - miReact
  - xgboost
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pooled-versus-single-cell-prediction-strategy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A two-mode miRNA-inference strategy from the miRSCAPE paper: pool cells within a scRNA-seq cluster to obtain a non-sparse transcriptomic profile before applying pretrained miRSCAPE models, and separately apply the same models per single cell, then compare induced-vs-uninduced log-fold changes against a target-only control (miReact).

## When to use

Use when predicting miRNA activity from scRNA-seq data where single-cell profiles are too sparse for a pretrained bulk-trained model, and you want to contrast pooled-cluster predictions with per-cell predictions.

## When NOT to use

- The input is already bulk (non-sparse) RNA-seq data — pooling is unnecessary; apply the miRSCAPE model directly in bulk mode
- The application sample's cellular/tissue context differs substantially from the model's training cohort, where the article expects 'a more substantive loss in performance' — retraining on detected genes may be needed instead
- The analysis goal is miRNA target activity from putative-target lists alone without a trained model — use the target-only control approach (e.g. miReact) rather than pooled miRSCAPE

## Inputs

- Seurat object containing induced vs uninduced single-cell RNA-seq expression data (e.g. Rzepiela miR-199a dataset)
- Pretrained CCLE-trained miRSCAPE xgboost models for miR-199a-5p and miR-199a-3p
- Cell cluster/cell-type labels (clusterId) for pooled prediction
- Bulk mRNA and bulk miRNA training matrices (rows = genes/miRNAs, columns = samples, gene names as row names)

## Outputs

- Predicted miRNA expression values per pooled cluster and per single cell (miRSCAPE pred output)
- Induced-vs-uninduced log-fold changes and Wilcoxon p-values for pooled miRSCAPE, single-cell miRSCAPE, and miReact control
- mir199a_logfc_results.csv with one row per miRNA/cell-type/strategy comparison

## How to apply

Prepare the Seurat-object scRNA-seq data with modifySeuratObject (R 4.1.0, Seurat, xgboost), then run the CCLE-trained xgboost miRSCAPE models in two modes: (1) pool cells within each cluster so the transcriptomic profile is non-sparse, mimicking a bulk profile, and (2) apply the model per cell on genome-wide mRNA profiles. Run miReact, the cross-gene-correlation target-only method, on the same induced/uninduced profiles as a control. Compute induced-vs-uninduced log-fold changes and Wilcoxon p-values per strategy: pooled miRSCAPE (logFC 6.2, 6.0, 5.9, 4.9, p < E-72), single-cell miRSCAPE (logFC 2.6, 2.6, 1.9, 1.5, p < E-36), miReact control (logFC 0.33, 0.15, -1.00, -1.13), and write results to mir199a_logfc_results.csv. Pooling is rationalized by the large dropout rate in single-cell data, which makes pretrained bulk models unreliable on sparse individual profiles.

## Related tools

- **miRSCAPE** (xgboost-based machine learning model applied in pooled-cluster and single-cell modes to predict miRNA expression from genome-wide mRNA profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **miReact** (cross-gene-correlation-based target-only method run as a control on the same induced/uninduced profiles)
- **Seurat** (reads and prepares the scRNA-seq Seurat object via modifySeuratObject before prediction)
- **xgboost** (R dependency providing the boosted-tree backend for the pretrained miRSCAPE models)
- **R 4.1.0** (runtime environment required to run the miRSCAPE pipeline)

## Examples

```
denem = modifySeuratObject(pbmc = example, clusterId = clustId); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Pooled miRSCAPE logFC should exceed single-cell miRSCAPE logFC for the same miRNA/cell-type (e.g. 6.2 vs 2.6), reflecting reduced sparsity after pooling
- All four pooled cases (2 miRNAs in 2 cell types) should show significantly higher predictions in induced cells with Wilcoxon p < E-72; single-cell mode p < E-36
- The miReact target-only control should yield near-zero or negative logFC (0.33, 0.15, -1.00, -1.13), confirming the signal is not an artifact of target-list methods
- modifySeuratObject output must contain genome-wide mRNA profiles with gene names as row names and be restricted to the specified clusterId
- Results table must contain rows for all 4 miRNA/cell-type combinations across the 3 strategies and be written to mir199a_logfc_results.csv

## Limitations

- miRSCAPE requires training on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no specific training
- Model accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort
- Due to scRNA-seq dropout, a pre-trained model may not work on individual cells and miRSCAPE may need to be retrained using only genes detected at a reasonable level
- A substantive loss in performance is expected when the training context differs substantially from the application context, as quantified in Figure S9

## Evidence
