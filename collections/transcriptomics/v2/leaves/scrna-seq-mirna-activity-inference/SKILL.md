---
name: scrna-seq-mirna-activity-inference
description: Use when you need to predict miRNA expression or activity from scRNA-seq
  clusters or bulk RNA-seq profiles lacking miRNA measurements, e.g. Seurat objects
  from HCL, TCGA, CCLE, or induced-miRNA experiments.
when_to_use_negative:
- You already have directly measured miRNA expression for the samples — miRSCAPE is
  designed for profiles where mature miRNAs are not captured by poly(A)-based scRNA-seq
  protocols.
- Single-cell data is too sparse and cannot be pooled by cluster — the article warns
  that given the large dropout rate, a pre-trained model may not work and retraining
  on detected genes may be required.
- The application sample's cellular/tissue context is very different from the training
  cohort — a substantive loss in performance is expected (Figure S9), so a context-matched
  model should be trained instead.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: 'Core XGBoost-based model: trained on paired bulk miRNA-mRNA cohorts and applied
    to pooled scRNA-seq profiles; predictions returned in pred'
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: miReact
  role: Target-only baseline/control method for cross-gene-correlation-based miRNA
    activity inference used in benchmarking
- name: xgboost
  role: Gradient-boosting backend powering the miRSCAPE model's default booster, objective,
    depth, eta, rounds, and early stopping parameters
- name: Seurat
  role: Stores and provides the scRNA-seq objects processed by modifySeuratObject
    before prediction
- name: R 4.1.0
  role: Runtime environment required to source miRSCAPE.R and run the workflow
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:ba52994f6bd5ce8e1380217c3f07b05f1b85e5d3081760c7e631cd4eff0a3182
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
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scrna-seq-mirna-activity-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train or apply miRSCAPE, an XGBoost-based machine learning model, to infer miRNA expression/activity from genome-wide mRNA profiles of single-cell clusters, compensating for the inability of poly(A)-based scRNA-seq protocols to capture mature miRNAs.

## When to use

Use when you need to predict miRNA expression or activity from scRNA-seq clusters or bulk RNA-seq profiles lacking miRNA measurements, e.g. Seurat objects from HCL, TCGA, CCLE, or induced-miRNA experiments.

## When NOT to use

- You already have directly measured miRNA expression for the samples — miRSCAPE is designed for profiles where mature miRNAs are not captured by poly(A)-based scRNA-seq protocols.
- Single-cell data is too sparse and cannot be pooled by cluster — the article warns that given the large dropout rate, a pre-trained model may not work and retraining on detected genes may be required.
- The application sample's cellular/tissue context is very different from the training cohort — a substantive loss in performance is expected (Figure S9), so a context-matched model should be trained instead.

## Inputs

- Seurat object with scRNA-seq expression (e.g. HCL, Zilionis GSE127465, Rzepiela miR-199a induction data), prepared via modifySeuratObject
- Paired bulk mRNA and bulk miRNA expression matrices (TCGA, GTEx, CCLE; rows = genes/miRNAs, columns = patient samples)
- Pre-trained miRSCAPE models (CCLE-trained or AML-TCGA-trained) from the miRSCAPE repository

## Outputs

- Predicted miRNA expression values in the pred output (e.g. 523 miRNAs per cell type for HCL)
- Ranked top 10 up/downregulated miRNAs per cell type (e.g. hcl_top10_mirnas_per_celltype.csv)
- Per-miRNA accuracy or auROC validation tables (e.g. accuracy_comparison.csv, encode_zilionis_auroc_results.csv, mir199a_logfc_results.csv)
- Induced-vs-uninduced log-fold changes and Wilcoxon p-values per prediction strategy

## How to apply

In R 4.1.0 with xgboost and Seurat, load data (rows = genes, columns = samples) and prepare Seurat-stored scRNA-seq with modifySeuratObject (optionally restricted via clusterId to specific cell types). Prepare bulk paired miRNA/miRNA training matrices (e.g. TCGA, GTEx, CCLE; ~10,000 samples across 10 tissues) with bulkTransform, setting justNorm=TRUE for the miRNA matrix (log-transform only, do not scale). Train with the miRSCAPE function using default XGBoost parameters (booster bstr, objective objt, maximum depth mdpth, eta ett, number of rounds nrnds, early stopping esr), then apply the model to pooled single-cell profiles — pool cells within each cluster to obtain non-sparse profiles and mitigate dropout — and read predicted miRNA expression from the pred output. Rationale: direct and downstream indirect regulatory links between miRNAs and mRNAs carry more signal than putative-target-only methods like miReact, which reached only ~0.24 top-50 accuracy versus ~0.86 for miRSCAPE on pooled TCGA cohorts.

## Related tools

- **miRSCAPE** (Core XGBoost-based model: trained on paired bulk miRNA-mRNA cohorts and applied to pooled scRNA-seq profiles; predictions returned in pred) — https://github.com/hannenhalli-lab/miRSCAPE
- **miReact** (Target-only baseline/control method for cross-gene-correlation-based miRNA activity inference used in benchmarking)
- **xgboost** (Gradient-boosting backend powering the miRSCAPE model's default booster, objective, depth, eta, rounds, and early stopping parameters)
- **Seurat** (Stores and provides the scRNA-seq objects processed by modifySeuratObject before prediction)
- **R 4.1.0** (Runtime environment required to source miRSCAPE.R and run the workflow)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Pan-cancer benchmark on pooled TCGA cohorts: top-50 most predictable miRNAs reach average accuracy ~0.86 for miRSCAPE versus ~0.24 for miReact.
- ENCODE cell-line validation: auROC of predicted expression discriminating detected vs undetected McCall miRNAs averages ~0.71 across 10 cell lines; mouse hematopoietic cell types average ~0.67 with the AML-TCGA-trained model.
- miR-199a induction test: pooled miRSCAPE logFC of 6.2, 6.0, 5.9, 4.9 with Wilcoxon p < E-72 for all 4 cases; unpooled single-cell mode gives logFC 2.6, 2.6, 1.9, 1.5 (p < E-36); miReact control gives logFC 0.33, 0.15, -1.00, -1.13.
- Predicted vs observed fold-difference correlation ~0.81 in two independent cell-type-specific miRNA datasets (HEK-GBM, Kidney-Breast-Skin).
- Schema checks: input matrices must have gene/miRNA names as row names and samples as columns; miRNA data must be log-transformed only (justNorm=TRUE) and not scaled.

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no training.
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; a very different context causes a substantive performance loss (Figure S9).
- Given the large scRNA-seq dropout rate, a pre-trained model may not work and miRSCAPE may need to be retrained using only genes detected at a reasonable level.
- Large cohorts of paired miRNA-protein expression data are not available to train and assess a protein-based model.

## Evidence
