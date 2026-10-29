---
name: tissue-specific-xgboost-model-training
description: Use when you have a large cohort of paired bulk miRNA and mRNA matrices
  for one tissue (e.g., TCGA skin, breast, or kidney) and need a trained regression
  model to infer miRNA expression from mRNA-only profiles such as pooled scRNA-seq.
when_to_use_negative:
- You only need miRNA activity inferred from putative miRNA-target relationships without
  training data (e.g., target-set enrichment approaches); miRSCAPE requires a large
  cohort of paired miRNA-mRNA data.
- The application sample's tissue/cell context differs substantially from the available
  training cohort, where a substantive loss in performance is expected (Figure S9).
- The scRNA-seq input is too sparse for a pre-trained model due to dropout and cannot
  be retrained on the genes detected at a reasonable level.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0203
tools:
- name: miRSCAPE
  role: Provides the bulkTransform data-preparation function and the miRSCAPE function
    that trains the XGBoost regression model on paired bulk miRNA-mRNA data
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying gradient-boosted regression engine used with default parameters
    (booster, objective, maximum depth, eta, number of rounds) to fit each tissue
    model
- name: Seurat
  role: Handles the Seurat-object scRNA-seq data that the trained model is subsequently
    applied to via modifySeuratObject
- name: R 4.1.0
  role: Required runtime for running the miRSCAPE training workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:6b79de77e21d275640eec37f17dc80c823d420d64792488435794be966e3bbac
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tissue-specific-xgboost-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a tissue-specific miRSCAPE XGBoost regression model on paired bulk miRNA-mRNA profiles (e.g., TCGA) so that genome-wide mRNA profiles can later be used to predict miRNA expression, as implemented in the miRSCAPE R package.

## When to use

Use when you have a large cohort of paired bulk miRNA and mRNA matrices for one tissue (e.g., TCGA skin, breast, or kidney) and need a trained regression model to infer miRNA expression from mRNA-only profiles such as pooled scRNA-seq.

## When NOT to use

- You only need miRNA activity inferred from putative miRNA-target relationships without training data (e.g., target-set enrichment approaches); miRSCAPE requires a large cohort of paired miRNA-mRNA data.
- The application sample's tissue/cell context differs substantially from the available training cohort, where a substantive loss in performance is expected (Figure S9).
- The scRNA-seq input is too sparse for a pre-trained model due to dropout and cannot be retrained on the genes detected at a reasonable level.

## Inputs

- Paired bulk miRNA expression matrix (rows = miRNAs, columns = patients/samples, gene identifiers as row names), e.g., TCGA skin/breast/kidney miRNA data
- Paired bulk mRNA expression matrix (rows = genes, columns = patients/samples) from the same tissue cohort
- R 4.1.0 environment with xgboost and Seurat packages and miRSCAPE.R sourced

## Outputs

- Tissue-specific trained XGBoost regression model (miRSCAPE model) mapping genome-wide mRNA profiles to miRNA expression
- Predicted miRNA expression per input profile, available in the 'pred' output of the miRSCAPE function

## How to apply

Load the paired bulk miRNA and mRNA matrices (rows = genes/miRNAs, columns = patients; gene names as row names) and log-transform them with the bulkTransform function, setting justNorm=TRUE for the miRNA data (do not scale miRNA). Train the tissue-specific model by calling the miRSCAPE function with the transformed bulk mRNA (bulkmRNA) and bulk miRNA (bulkmiRNA) as training inputs; under the hood it fits an XGBoost regressor using default parameters (booster bstr, objective objt, maximum depth mdpth, eta ett, number of rounds nrnds), all of which can be overridden via the corresponding arguments. R 4.1.0 with the xgboost and Seurat packages is required. Train one model per tissue context, because miRSCAPE accuracy degrades when the training tissue context differs from the application context. The trained model is then applied to scRNA-seq mRNA profiles prepared via modifySeuratObject, with predictions retrieved from the 'pred' output.

## Related tools

- **miRSCAPE** (Provides the bulkTransform data-preparation function and the miRSCAPE function that trains the XGBoost regression model on paired bulk miRNA-mRNA data) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying gradient-boosted regression engine used with default parameters (booster, objective, maximum depth, eta, number of rounds) to fit each tissue model)
- **Seurat** (Handles the Seurat-object scRNA-seq data that the trained model is subsequently applied to via modifySeuratObject)
- **R 4.1.0** (Required runtime for running the miRSCAPE training workflow)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Apply the trained model to pooled scRNA-seq profiles and compute Spearman rank correlation between predicted and observed miRNA fold-differences across cell-type pairs; for skin/breast/kidney the correlations should fall in the 0.76-0.89 range.
- Check that the average predicted-versus-observed fold-difference correlation is near the reported ~0.81 for the Kidney-Breast-Skin dataset.
- Confirm miRNA matrices were log-transformed with justNorm=TRUE (not scaled) before training, per the bulkTransform contract.
- Verify that mRNA and miRNA input matrices have gene/miRNA identifiers as row names and samples as columns before calling miRSCAPE.
- Cross-validate model accuracy within the bulk cohort (TCGA/GTEx/CCLE-style paired samples) to confirm the trained regressor generalizes before applying to single-cell data.

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no training.
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort, with substantive loss expected when contexts differ substantially (Figure S9).
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need retraining using only genes detected at a reasonable level.

## Evidence
