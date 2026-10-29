---
name: tcga-gtex-cohort-harmonization
description: Use when training or evaluating miRSCAPE on bulk TCGA/GTex paired miRNA-mRNA
  matrices and you need pooled multi-cancer or tumor-vs-normal cross-validation accuracy
  comparisons.
when_to_use_negative:
- The input is already a single-cell/scRNA-seq dataset — use the modifySeuratObject
  (Seurat) preparation path instead, and note a pre-trained model may need retraining
  due to dropout.
- Only one tissue or a small cohort is available — tissue-specific models lose ~0.2
  accuracy relative to the pooled 8,089-sample TCGA model, so pooling is the point
  of this skill.
- The application sample's tissue context does not match the training cohort — accuracy
  is expected to drop (see Figure S9), so retrain on a matching cohort.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Trains the XGBoost model on the harmonized bulk cohorts and performs cross-validated
    miRNA prediction; provides the bulkTransform preparation function
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Required runtime for running the miRSCAPE pipeline and cohort harmonization
    scripts
- name: xgboost
  role: Gradient-boosted model used by miRSCAPE with default booster, objective, depth,
    eta, rounds, and early-stopping parameters
- name: Seurat
  role: Dependency of miRSCAPE for single-cell data preparation; not central to the
    bulk cohort harmonization path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:50426323f4ce01aac750214d36408efc6d60a9f1f83fabc09870f27873092b43
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# tcga-gtex-cohort-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes and pools paired bulk miRNA–mRNA cohorts from TCGA (~5,000 tumor samples) and GTEx (~4,500 normal samples) so that miRSCAPE's XGBoost model can be trained and cross-validated on large multi-tissue cohorts, quantifying how tissue heterogeneity affects prediction accuracy.

## When to use

Use when training or evaluating miRSCAPE on bulk TCGA/GTex paired miRNA-mRNA matrices and you need pooled multi-cancer or tumor-vs-normal cross-validation accuracy comparisons.

## When NOT to use

- The input is already a single-cell/scRNA-seq dataset — use the modifySeuratObject (Seurat) preparation path instead, and note a pre-trained model may need retraining due to dropout.
- Only one tissue or a small cohort is available — tissue-specific models lose ~0.2 accuracy relative to the pooled 8,089-sample TCGA model, so pooling is the point of this skill.
- The application sample's tissue context does not match the training cohort — accuracy is expected to drop (see Figure S9), so retrain on a matching cohort.

## Inputs

- TCGA bulk miRNA expression matrix (rows = gene names, columns = patient samples) for brain, breast, colon, liver, lung cancers
- TCGA bulk mRNA expression matrix (rows = gene names, columns = patient samples)
- GTEx bulk miRNA and mRNA matrices for the matched normal tissues
- R 4.1.0 with xgboost and Seurat, and miRSCAPE.R sourced from the repository

## Outputs

- Cross-validation accuracy per cohort configuration (pooled TCGA, GTEx-only, pooled TCGA+GTEx)
- figure3_accuracy_table.csv — tabulated accuracy comparison across the three configurations

## How to apply

Load paired bulk miRNA and mRNA matrices (genes as row names, samples as columns) spanning brain, breast, colon, liver, and lung cancers from TCGA (8,089 samples) and their matched GTEx normal tissues. Prepare each matrix with the miRSCAPE bulkTransform function in R 4.1.0, log-transforming mRNA data and setting justNorm=TRUE for miRNA so it is only log-transformed, not scaled. Train the miRSCAPE XGBoost model (default booster, objective, maximum depth, eta, number of rounds, and early stopping) under cross-validation on three configurations: pooled TCGA tumors only, matched GTEx normals only, and pooled TCGA+GTEx. Tabulate the resulting accuracies to reproduce the Figure 3 comparison showing pooled 8,089-sample TCGA accuracy (~0.62) exceeds tissue-specific (~0.34) and GTEx-only (~0.18) models, with pooled TCGA+GTEx at ~0.54.

## Related tools

- **miRSCAPE** (Trains the XGBoost model on the harmonized bulk cohorts and performs cross-validated miRNA prediction; provides the bulkTransform preparation function) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Required runtime for running the miRSCAPE pipeline and cohort harmonization scripts)
- **xgboost** (Gradient-boosted model used by miRSCAPE with default booster, objective, depth, eta, rounds, and early-stopping parameters)
- **Seurat** (Dependency of miRSCAPE for single-cell data preparation; not central to the bulk cohort harmonization path)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna)
```

## Evaluation signals

- Pooled 8,089-sample TCGA cohort cross-validation accuracy averages ~0.62
- Matched-tissue comparisons: TCGA-only ~0.34 and GTEx-only ~0.18
- Pooled TCGA+GTEx cohort accuracy ~0.54
- Output table figure3_accuracy_table.csv contains the three accuracy configurations and matches reported values
- All matrices have gene names as row names and were passed through bulkTransform (mRNA log-transformed; miRNA with justNorm=TRUE, unscaled)

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data; it cannot be applied training-free like target-set-based methods.
- Model accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort, with substantive loss quantified in Figure S9.
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE needs to be retrained using only the genes detected at a reasonable level.

## Evidence
