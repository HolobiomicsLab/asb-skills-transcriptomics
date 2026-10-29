---
name: bulk-mirna-mrna-gradient-boosted-model-training
description: Use when you have paired bulk mRNA and miRNA expression matrices for
  a tissue/cancer cohort and need a trained model to later infer miRNA expression
  in scRNA-seq clusters.
when_to_use_negative:
- You only have a miRNA-target relationship list and no paired miRNA-mRNA training
  cohort — target-set-based approaches (e.g. miReact, SCENIC-style activity inference)
  are the alternative when training data is unavailable
- The application context differs substantially from the training cohort — a substantive
  loss in performance is expected (quantified in Figure S9), so a context-matched
  training set should be built instead
- The data are already single-cell sparse profiles with no paired bulk cohort to train
  on and no pre-trained model for that tissue context
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: Provides the bulkTransform data-preparation function and the miRSCAPE training
    function (XGBoost regression) that fits the miRNA-from-mRNA model
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Gradient-boosted regression backend used by miRSCAPE with default booster,
    objective, maximum depth, eta, and number of rounds settings
- name: R 4.1.0
  role: Runtime environment; the miRSCAPE.R source is loaded and functions called
    from R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:c1f12e3d33be3dd21e8719a95f459a4d3fcaf8ef333511a539bda82cb0c94f66
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
  - miRSCAPE
  - xgboost
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bulk-mirna-mrna-gradient-boosted-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train miRSCAPE XGBoost regression models that predict miRNA expression from genome-wide mRNA profiles, using large cohorts of paired bulk miRNA-mRNA samples (e.g. TCGA, GTEx, CCLE) as training data.

## When to use

Use when you have paired bulk mRNA and miRNA expression matrices for a tissue/cancer cohort and need a trained model to later infer miRNA expression in scRNA-seq clusters.

## When NOT to use

- You only have a miRNA-target relationship list and no paired miRNA-mRNA training cohort — target-set-based approaches (e.g. miReact, SCENIC-style activity inference) are the alternative when training data is unavailable
- The application context differs substantially from the training cohort — a substantive loss in performance is expected (quantified in Figure S9), so a context-matched training set should be built instead
- The data are already single-cell sparse profiles with no paired bulk cohort to train on and no pre-trained model for that tissue context

## Inputs

- Paired bulk mRNA expression matrix (rows = genes, columns = patient samples, gene names as row names)
- Paired bulk miRNA expression matrix from the same cohort (e.g. TCGA, GTEx, CCLE)
- Optional scRNA-seq mRNA matrix of genes detected at reasonable level for dropout-aware retraining

## Outputs

- Trained XGBoost regression model(s) mapping bulk mRNA profiles to miRNA expression (e.g. tissue-specific and pooled models)
- Predicted miRNA expression values for supplied mRNA profiles, retrieved from the pred output of the miRSCAPE function

## How to apply

Load the paired bulk mRNA and bulk miRNA matrices (rows are genes/miRNAs, columns are patients, with gene names as row names) and prepare each with the bulkTransform function: log-transform for mRNA, and set justNorm = TRUE for the miRNA data so it is log-transformed but not scaled. Train the model with the miRSCAPE function, which fits an XGBoost regression with default settings (booster, objective, maximum depth, eta, number of rounds; overridable via bstr, objt, mdpth, ett, nrnds parameters, with esr for early stopping). Fit tissue-matched models (e.g. separate brain and kidney TCGA models) and optionally a pooled cross-tissue model; tissue-matched models outperform pooled ones (Spearman rho ≈ 0.8 vs 0.73 on HCL brain/kidney validation). Train on a large cohort of paired miRNA-mRNA data, since miRSCAPE requires substantial training samples unlike target-set-based alternatives. If applying to scRNA-seq with high dropout, consider retraining using only genes detected at a reasonable level in the single-cell data.

## Related tools

- **miRSCAPE** (Provides the bulkTransform data-preparation function and the miRSCAPE training function (XGBoost regression) that fits the miRNA-from-mRNA model) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Gradient-boosted regression backend used by miRSCAPE with default booster, objective, maximum depth, eta, and number of rounds settings)
- **R 4.1.0** (Runtime environment; the miRSCAPE.R source is loaded and functions called from R)

## Examples

```
source('~/miRSCAPE-main/code/miRSCAPE.R'); bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- bulkTransform applied with justNorm = TRUE to miRNA data (log-transform only, NOT scaled) and default log-transform to mRNA data
- Model reproduces paper benchmark: predicted vs observed inter-cell-type fold-difference Spearman rho ≈ 0.8 for tissue-matched models and ≈ 0.73 for the pooled model (paper benchmark ~0.81)
- Training matrices satisfy the README schema: rows are genes/miRNAs, columns are patient samples, gene names present as row names
- Predictions are retrievable from the pred output of the miRSCAPE function call without errors from the xgboost backend
- Cross-validation accuracy established on the training cohort (e.g. TCGA/GTEx/CCLE) before applying to single-cell data

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no specific training
- Model accuracy is slightly reduced when the cellular/tissue context of the application sample does not match the training cohort, with a substantive loss when contexts differ substantially (Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need retraining using only genes detected at a reasonable level

## Evidence
