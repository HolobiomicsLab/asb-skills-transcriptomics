---
name: bootstrapped-pseudobulk-cell-type-aggregation
description: Use when scRNA-seq data are sparse (high dropout) and miRNA activity
  must be inferred per cell type/cluster, e.g. profiling 523 miRNAs across all 56
  HCL cell types with a global miRSCAPE model.
when_to_use_negative:
- When single-cell-level (per-cell) miRNA resolution is required — miRSCAPE predictions
  are interpreted at the level of cell clusters, since pooling merges cells within
  a cluster
- When the bulk training cohort's tissue/cellular context differs substantially from
  the application context — accuracy is expected to drop (quantified in Figure S9)
- When nearly no genes are detected in a cluster even after pooling — a pre-trained
  model may fail and retraining on only detected genes may be required
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Machine-learning model (backed by xgboost) applied to each bootstrapped cell-type-pooled
    profile to predict miRNA expression from the pooled mRNA profile
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Stores the scRNA-seq data; the object is processed with modifySeuratObject
    to prepare per-cell-type profiles with gene row names preserved
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying gradient-boosting engine of the miRSCAPE predictor, run with default
    booster/objective parameters unless tuning is warranted
- name: R 4.1.0
  role: Runtime environment in which the pooling, bootstrapping, and miRSCAPE prediction
    are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:f2dc046f28d3d6d26e4b391e1f52750941dc5323804275fb7a24d3e627c98e0a
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bootstrapped-pseudobulk-cell-type-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates single-cell expression profiles into bootstrapped, cell-type-pooled (pseudobulk) matrices so that a trained miRSCAPE model can be applied to non-sparse transcriptomic profiles, e.g. for the 56 cell types of the Human Cell Landscape (HCL).

## When to use

Use when scRNA-seq data are sparse (high dropout) and miRNA activity must be inferred per cell type/cluster, e.g. profiling 523 miRNAs across all 56 HCL cell types with a global miRSCAPE model.

## When NOT to use

- When single-cell-level (per-cell) miRNA resolution is required — miRSCAPE predictions are interpreted at the level of cell clusters, since pooling merges cells within a cluster
- When the bulk training cohort's tissue/cellular context differs substantially from the application context — accuracy is expected to drop (quantified in Figure S9)
- When nearly no genes are detected in a cluster even after pooling — a pre-trained model may fail and retraining on only detected genes may be required

## Inputs

- Seurat object with genome-wide scRNA-seq profiles (e.g. HCL data) and cell-type/cluster annotations
- Trained miRSCAPE model (built from paired bulk miRNA-mRNA matrices, e.g. ~10,000 TCGA/GTEx/CCLE tumor and normal samples across 10 tissues)

## Outputs

- Bootstrapped cell-type-pooled (pseudobulk) mRNA expression matrices, one set per cell type
- Predicted miRNA expression (pred output) for each cell type, covering 523 miRNAs
- Ranked miRNA table per cell type (e.g. hcl_top10_mirnas_per_celltype.csv with top 10 up- and downregulated miRNAs per cell type)

## How to apply

Group cells by their cluster or cell-type label (e.g. the clusterId used with modifySeuratObject), then pool cells within each cluster to obtain a non-sparse transcriptomic profile; repeat via bootstrapping to generate multiple pooled profiles per cell type and mitigate sparsity/dropout. Prepare the Seurat object with modifySeuratObject, retaining gene row names, so each bootstrapped pooled profile can serve as the scmRNA input to miRSCAPE. Apply the trained miRSCAPE model (xgboost-backed, default booster/objective parameters) to each bootstrapped cell-type profile to predict miRNA expression in the pred output. Finally rank predicted miRNAs per cell type — for the HCL, extract the top 10 up- and downregulated miRNAs for each of the 56 cell types and save as hcl_top10_mirnas_per_celltype.csv, sanity-checking recapitulated activities against known cell-type miRNA biology.

## Related tools

- **miRSCAPE** (Machine-learning model (backed by xgboost) applied to each bootstrapped cell-type-pooled profile to predict miRNA expression from the pooled mRNA profile) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Stores the scRNA-seq data; the object is processed with modifySeuratObject to prepare per-cell-type profiles with gene row names preserved) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying gradient-boosting engine of the miRSCAPE predictor, run with default booster/objective parameters unless tuning is warranted)
- **R 4.1.0** (Runtime environment in which the pooling, bootstrapping, and miRSCAPE prediction are executed)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Each bootstrapped pooled profile is non-sparse (a dense genome-wide mRNA vector per cell type) and retains gene names as row names
- The miRSCAPE pred output contains predicted expression for 523 miRNAs for each of the 56 cell types
- Ranked table hcl_top10_mirnas_per_celltype.csv contains top 10 up- and downregulated miRNAs for every cell type
- Recapitulated miRNA activities are consistent with known cell-type miRNA biology (sanity check)
- Predicted vs observed fold-difference correlation in validated contexts is ~0.81, providing an expected accuracy reference

## Limitations

- Pooling cells within a cluster yields cluster-level (not single-cell) miRNA activity; predictions are interpreted at cell-cluster level
- Given the large scRNA-seq dropout rate, a pre-trained model may not work and miRSCAPE may need to be retrained using only genes detected at a reasonable level
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; very different contexts cause a substantive performance loss (Figure S9)

## Evidence

- [discussion] we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity: "we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity"
- [abstract] We apply miRSCAPE to infer miRNA activities in scRNA clusters in pancreatic and lung adenocarcinomas, as well as in 56 cell types in the human cell landscape (HCL).: "We apply miRSCAPE to infer miRNA activities in scRNA clusters in pancreatic and lung adenocarcinomas, as well as in 56 cell types in the human cell landscape (HCL)."
- [readme] To prepare the data which is stored in Seurat object, `modifySeuratObject` function should be used.: "To prepare the data which is stored in Seurat object, `modifySeuratObject` function should be used."
