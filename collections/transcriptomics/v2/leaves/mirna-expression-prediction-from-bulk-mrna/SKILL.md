---
name: mirna-expression-prediction-from-bulk-mrna
description: Use when you have paired bulk mRNA and miRNA expression matrices (genes
  as rows, samples as columns) and want to predict miRNA expression in samples or
  pooled single-cell clusters lacking direct miRNA measurement.
when_to_use_negative:
- The query sample's cellular/tissue context differs substantially from the training
  cohort — accuracy loss is expected (quantified in Figure S9)
- scRNA-seq data has severe dropout such that detected genes are too sparse — a pre-trained
  model may not work and retraining on detected genes is required instead
- 'You need paired miRNA-protein prediction: large cohorts of paired miRNA-protein
  expression data are not available to train such a model'
edam_operation: http://edamontology.org/operation_0463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3518
tools:
- name: miRSCAPE
  role: Core XGBoost regression model that trains on paired bulk mRNA-miRNA data and
    predicts miRNA expression from scRNA-seq mRNA profiles; predictions retrieved
    from 'pred' output
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: bulkTransform
  role: Preprocesses bulk matrices; justNorm=TRUE log-transforms (does not scale)
    miRNA data
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: modifySeuratObject
  role: Prepares Seurat-object scRNA-seq data, selecting specified cell clusters and
    pooling cells within clusters to form non-sparse profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying gradient boosting engine with default booster, objective, maximum
    depth, eta, number of rounds, and early-stopping settings
- name: Seurat
  role: Data structure for scRNA-seq input consumed by modifySeuratObject
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:70a425196a650536507ad8552d6f23daa6451b7bb8d9ee4f062729eaaf4fa50a
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
  - Seurat
  - bulkTransform
  - miRSCAPE
  - modifySeuratObject
  - xgboost
  edam_operation: http://edamontology.org/operation_0463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mirna-expression-prediction-from-bulk-mrna

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and apply miRSCAPE, an XGBoost-based machine learning tool that predicts miRNA expression from paired bulk mRNA-miRNA cohorts (TCGA, GTEx, CCLE) and infers miRNA expression in single-cell clusters from scRNA-seq profiles.

## When to use

Use when you have paired bulk mRNA and miRNA expression matrices (genes as rows, samples as columns) and want to predict miRNA expression in samples or pooled single-cell clusters lacking direct miRNA measurement.

## When NOT to use

- The query sample's cellular/tissue context differs substantially from the training cohort — accuracy loss is expected (quantified in Figure S9)
- scRNA-seq data has severe dropout such that detected genes are too sparse — a pre-trained model may not work and retraining on detected genes is required instead
- You need paired miRNA-protein prediction: large cohorts of paired miRNA-protein expression data are not available to train such a model

## Inputs

- Bulk mRNA expression matrix (rows = genes, columns = patients/samples, gene names as row names)
- Paired bulk miRNA expression matrix (rows = miRNAs, columns = samples)
- scRNA-seq data stored in a Seurat object (e.g. example.rds) with cell cluster annotations
- Cell cluster identifiers of interest (e.g. clustId = c('Ductal cell type 1'))

## Outputs

- Predicted miRNA expression per sample or pooled cell cluster, returned in the 'pred' slot of the miRSCAPE output
- Per-miRNA cross-validation Spearman accuracy table (e.g. mirna_cv_spearman_accuracy.csv)
- Per-cell-type-pair Spearman correlations between predicted and observed miRNA fold-differences

## How to apply

Load bulk mRNA and miRNA matrices with gene names as row names, then preprocess each with the miRSCAPE bulkTransform function — log-transform only the miRNA data by setting justNorm=TRUE (do not scale miRNA), and apply default normalization to mRNA. If the query data is a Seurat object, first use modifySeuratObject with a clusterId parameter to select cell clusters and pool cells within each cluster so profiles are non-sparse. Train the XGBoost model with the miRSCAPE function using default parameters (booster bstr, objective objt, maximum depth mdpth, eta ett, number of rounds nrnds, early stopping esr) and retrieve predicted miRNA expression from the 'pred' output. Assess accuracy via cross-sample Spearman correlation between predicted and observed expression in 5-fold cross-validation (~0.45 average across 10 TCGA cohorts) or via Spearman correlation of predicted vs observed miRNA fold-differences between cell-type pairs (~0.81 expected).

## Related tools

- **miRSCAPE** (Core XGBoost regression model that trains on paired bulk mRNA-miRNA data and predicts miRNA expression from scRNA-seq mRNA profiles; predictions retrieved from 'pred' output) — https://github.com/hannenhalli-lab/miRSCAPE
- **bulkTransform** (Preprocesses bulk matrices; justNorm=TRUE log-transforms (does not scale) miRNA data) — https://github.com/hannenhalli-lab/miRSCAPE
- **modifySeuratObject** (Prepares Seurat-object scRNA-seq data, selecting specified cell clusters and pooling cells within clusters to form non-sparse profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying gradient boosting engine with default booster, objective, maximum depth, eta, number of rounds, and early-stopping settings)
- **Seurat** (Data structure for scRNA-seq input consumed by modifySeuratObject)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- 5-fold cross-validation on 10 TCGA cancer cohorts yields average cross-sample Spearman accuracy ~0.45 (range 0.39–0.51), with 58% of miRNAs at Spearman >= 0.4
- Pooling 8,089 samples across five cancer types raises average CV accuracy to ~0.62 (~0.2 above tissue-specific models); matched TCGA-only ~0.34 vs GTEx-only ~0.18; pooled TCGA+GTEx ~0.54
- Cell-type-specific validation: Spearman correlation between predicted and observed miRNA fold-differences per cell-type pair falls in 0.76–0.89 (average ~0.81)
- Predictions are returned in the 'pred' output of the miRSCAPE call; verify output matrix shape covers queried samples/clusters x miRNAs
- Confirm miRNA matrix was log-transformed only (justNorm=TRUE), not scaled, and that gene names are row names of both bulk matrices

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no specific training
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; substantive loss when contexts differ substantially (Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and retraining using only reasonably detected genes may be required
- R 4.1.0 is required; the package depends on xgboost and Seurat

## Evidence
