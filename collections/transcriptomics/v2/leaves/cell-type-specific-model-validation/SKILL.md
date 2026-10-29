---
name: cell-type-specific-model-validation
description: Use when a model trained on paired bulk omics data (e.g., TCGA miRNA-mRNA)
  must be validated for cell-type-specific inference against scRNA-seq data with known
  cell-type miRNA profiles (e.g., Isakova Kidney-Breast-Skin GSE151334).
when_to_use_negative:
- When the cellular/tissue context of the application sample does not match the training
  cohort — miRSCAPE is expected to have slightly reduced accuracy, and a substantive
  loss in performance is expected when the training context differs substantially
  (Figure S9)
- When the scRNA-seq profile is too sparse (large dropout) for a pre-trained model
  — miRSCAPE needs to be retrained using only genes detected at a reasonable level
  rather than validated as-is
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: Trains tissue-specific XGBoost regression models on bulk paired miRNA-mRNA
    data and predicts miRNA expression in pooled scRNA-seq profiles from the 'pred'
    output
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying regression engine for the miRSCAPE model, run with default parameter
    settings (bstr, objt, mdpth, ett, nrnds)
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Stores the scRNA-seq object; modifySeuratObject selects cell clusters and
    pools cells within clusters into non-sparse profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Required runtime environment for running the miRSCAPE pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:12861dbf911e5fb4bcee3d4984309fd6d3c84925e11535ac1f784753d48fa49c
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-type-specific-model-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation skill for assessing whether a bulk-trained regression model (e.g., miRSCAPE XGBoost models trained on TCGA paired miRNA-mRNA data) correctly infers cell-type-specific expression when applied to pooled scRNA-seq profiles, by correlating predicted vs observed fold-differences across cell-type pairs with Spearman rank correlation.

## When to use

Use when a model trained on paired bulk omics data (e.g., TCGA miRNA-mRNA) must be validated for cell-type-specific inference against scRNA-seq data with known cell-type miRNA profiles (e.g., Isakova Kidney-Breast-Skin GSE151334).

## When NOT to use

- When the cellular/tissue context of the application sample does not match the training cohort — miRSCAPE is expected to have slightly reduced accuracy, and a substantive loss in performance is expected when the training context differs substantially (Figure S9)
- When the scRNA-seq profile is too sparse (large dropout) for a pre-trained model — miRSCAPE needs to be retrained using only genes detected at a reasonable level rather than validated as-is

## Inputs

- Paired TCGA bulk miRNA and mRNA matrices for the tissue of interest (genes/miRNAs as row names, patients as columns)
- scRNA-seq data stored in a Seurat object (e.g., GSE151334 Isakova Smart-seq-total kidney/breast/skin data) with defined cell clusters
- Cell cluster identifiers for the cell types of interest (e.g., clustId = c('Ductal cell type 1'))

## Outputs

- Per-cell-type predicted miRNA expression profiles (from the 'pred' output of miRSCAPE)
- Per-cell-type-pair Spearman rank correlations between predicted and observed miRNA fold-differences (expected range 0.76-0.89)
- Validation verdict confirming cell-type-specific inference accuracy (~0.81 average correlation)

## How to apply

Train the tissue-specific model (e.g., miRSCAPE XGBoost with default booster, objective, maximum depth, eta, and number of rounds) on log-transformed bulk paired data prepared with bulkTransform, setting justNorm=TRUE for the miRNA matrix. Prepare the scRNA-seq Seurat object with modifySeuratObject, selecting the cell clusters of interest, and pool cells within each cluster to form non-sparse profiles, mitigating scRNA-seq dropout. Apply the trained model to the pooled profiles and retrieve predictions from the 'pred' output. For each cell-type pair, compute observed miRNA fold-differences and Spearman-correlate them with model-predicted fold-differences; correctness is judged by correlations falling in the reported range 0.76-0.89 (average ~0.81 for the Kidney-Breast-Skin dataset).

## Related tools

- **miRSCAPE** (Trains tissue-specific XGBoost regression models on bulk paired miRNA-mRNA data and predicts miRNA expression in pooled scRNA-seq profiles from the 'pred' output) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying regression engine for the miRSCAPE model, run with default parameter settings (bstr, objt, mdpth, ett, nrnds)) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Stores the scRNA-seq object; modifySeuratObject selects cell clusters and pools cells within clusters into non-sparse profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Required runtime environment for running the miRSCAPE pipeline)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Per-cell-type-pair Spearman rank correlations between predicted and observed miRNA fold-differences fall in the reported range 0.76-0.89
- Average correlation across the Kidney-Breast-Skin cell-type pairs is approximately 0.81, matching the reported value
- The 'pred' output from miRSCAPE contains non-empty predicted miRNA expression values for every pooled cell-type profile
- miRNA data were log-transformed with bulkTransform(justNorm=TRUE) and NOT scaled, per the README requirement
- Pooled scRNA-seq profiles are non-sparse (cells pooled within each cluster) before model application

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no specific training
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; a substantive loss in performance is expected when the training context is very different (quantified in Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE needs to be retrained using only the genes that are detected at a reasonable level
- Validation relies on datasets where cell type-specific miRNA profiles are independently available (HEK-GBM, Kidney-Breast-Skin), which are rare

## Evidence
