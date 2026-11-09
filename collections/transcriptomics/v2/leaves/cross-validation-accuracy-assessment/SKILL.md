---
name: cross-validation-accuracy-assessment
description: Use when evaluating how training-cohort composition (tissue-specific
  vs pooled multi-cancer cohorts) changes the cross-validation accuracy of a trained
  miRNA-inference model such as miRSCAPE.
when_to_use_negative:
- The goal is miRNA prediction on new scRNA-seq clusters (use the miRSCAPE prediction
  workflow with modifySeuratObject/bulkTransform/miRSCAPE, not the accuracy-benchmarking
  setup)
- No paired miRNA-mRNA bulk training cohort is available — miRSCAPE must be trained
  on a large paired cohort, so accuracy assessment is not applicable
- The application sample's cellular/tissue context differs from the training cohort,
  where the paper expects reduced accuracy that this pooled-cohort benchmark does
  not model
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2640
tools:
- name: miRSCAPE
  role: XGBoost-based model trained on paired bulk miRNA-mRNA cohorts whose cross-validation
    accuracy is assessed under pooled vs tissue-specific configurations
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Gradient-boosting backend used by miRSCAPE with default booster, objective,
    maximum depth, eta, rounds, and early-stopping parameters during cross-validation
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Runtime environment in which bulkTransform, training, and cross-validation
    accuracy computations are executed
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Dependency of miRSCAPE used for scRNA-seq object handling in the broader workflow
    (not directly exercised in the bulk CV benchmark)
  repo: https://github.com/hannenhalli-lab/miRSCAPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:f81d3ed6bdf9bcc01637b00f090f0afa31088d0b1932212f6dc31f2a786b2631
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-validation-accuracy-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the miRSCAPE XGBoost model's cross-validation accuracy across tissue-specific, tissue-vs-normal, and pooled multi-cohort configurations (TCGA and GTEx bulk miRNA-mRNA profiles) to quantify how sample heterogeneity affects miRNA-inference accuracy.

## When to use

Use when evaluating how training-cohort composition (tissue-specific vs pooled multi-cancer cohorts) changes the cross-validation accuracy of a trained miRNA-inference model such as miRSCAPE.

## When NOT to use

- The goal is miRNA prediction on new scRNA-seq clusters (use the miRSCAPE prediction workflow with modifySeuratObject/bulkTransform/miRSCAPE, not the accuracy-benchmarking setup)
- No paired miRNA-mRNA bulk training cohort is available — miRSCAPE must be trained on a large paired cohort, so accuracy assessment is not applicable
- The application sample's cellular/tissue context differs from the training cohort, where the paper expects reduced accuracy that this pooled-cohort benchmark does not model

## Inputs

- Paired bulk miRNA expression matrix from TCGA (gene names as row names, samples as columns)
- Paired bulk mRNA expression matrix from TCGA (gene names as row names, samples as columns)
- Matched GTEx normal-tissue bulk miRNA and mRNA matrices
- Sample/cancer-type metadata for the 5 cancer types (brain, breast, colon, liver, lung)

## Outputs

- Per-configuration cross-validation accuracy values (tissue-specific, TCGA vs GTEx, pooled TCGA, pooled TCGA+GTEx)
- Comparison table written to figure3_accuracy_table.csv reproducing Figure 3

## How to apply

Load the paired bulk miRNA and mRNA cohorts (rows are gene names, columns are patients) from TCGA (~8,089 samples spanning brain, breast, colon, liver, and lung cancers) and matched GTEx normal tissues. Prepare each matrix with miRSCAPE's bulkTransform in R 4.1.0: log-transform mRNA, and for miRNA set justNorm=TRUE so the data are only log-transformed and not scaled. Train the miRSCAPE XGBoost model (default booster, objective, maximum depth, eta, number of rounds, and early-stopping parameters) under cross-validation for each configuration: each of the 5 TCGA cancer types alone, each matched GTEx normal tissue alone, the pooled 5-cancer TCGA cohort, TCGA-vs-GTEx, and pooled TCGA+GTEx. Tabulate per-configuration accuracies into a comparison table (figure3_accuracy_table.csv) to reproduce the Figure 3 analysis showing that pooling heterogeneous samples raises average CV accuracy to ~0.62, roughly +0.2 over tissue-specific models (~0.34 TCGA-only, ~0.18 GTEx-only, ~0.54 pooled TCGA+GTEx).

## Related tools

- **miRSCAPE** (XGBoost-based model trained on paired bulk miRNA-mRNA cohorts whose cross-validation accuracy is assessed under pooled vs tissue-specific configurations) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Gradient-boosting backend used by miRSCAPE with default booster, objective, maximum depth, eta, rounds, and early-stopping parameters during cross-validation) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Runtime environment in which bulkTransform, training, and cross-validation accuracy computations are executed) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Dependency of miRSCAPE used for scRNA-seq object handling in the broader workflow (not directly exercised in the bulk CV benchmark)) — https://github.com/hannenhalli-lab/miRSCAPE

## Examples

```
source("~/miRSCAPE-main/code/miRSCAPE.R"); bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna)
```

## Evaluation signals

- Average CV accuracy on the pooled 8,089-sample TCGA cohort is ~0.62
- Average CV accuracy is ~0.34 on TCGA-only tissues and ~0.18 on matched GTEx-only normal tissues
- Average CV accuracy on the pooled TCGA+GTEx cohort is ~0.54
- Pooled TCGA accuracy exceeds the average of the five tissue-specific models by ~0.2
- figure3_accuracy_table.csv contains all three accuracy configurations (pooled TCGA, TCGA vs GTEx, pooled TCGA+GTEx) matching the reported values

## Limitations

- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not perfectly match the training cohort
- When the training context is very different from the application context, a more substantive loss in performance is expected, as quantified in Figure S9
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need to be retrained using only the genes detected at a reasonable level

## Evidence
