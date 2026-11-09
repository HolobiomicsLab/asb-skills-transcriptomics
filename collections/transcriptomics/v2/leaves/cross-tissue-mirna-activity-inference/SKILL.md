---
name: cross-tissue-mirna-activity-inference
description: Use when scRNA-seq profiles lack miRNA measurements (poly(A) capture
  precludes mature miRNAs) and paired bulk mRNA-miRNA training cohorts exist for the
  tissue(s) of interest, so miRNA expression can be predicted from mRNA profiles.
when_to_use_negative:
- The goal is to infer miRNA activity from putative targets alone without a large
  cohort of paired miRNA-mRNA training data — miRSCAPE explicitly requires such training,
  unlike target-set enrichment approaches (e.g., miReact-style)
- The training tissue/context differs substantially from the application context —
  a substantive performance loss is expected (quantified in Figure S9)
- A pre-trained model is being applied to scRNA-seq with high dropout where most model
  genes are undetected — retraining on only reasonably detected genes may be required
  instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: XGBoost-based tool that trains on paired bulk mRNA-miRNA data and predicts
    miRNA expression in single-cell clusters from their mRNA profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying gradient-boosting regression engine for miRSCAPE model training
    (booster, objective, max depth, eta, rounds parameters)
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Holds and prepares the scRNA-seq data; modifySeuratObject pools cells within
    clusters into non-sparse pseudo-bulk profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:a0b3c102d8dcf03621d567178b5cf0c85b49a8be3df6ea15795c29163884f50d
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-tissue-mirna-activity-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train miRSCAPE XGBoost regression models on paired bulk mRNA-miRNA cohorts (e.g., TCGA brain and kidney) and apply them to pooled scRNA-seq pseudo-bulk profiles to infer cell type-specific miRNA expression, validated by correlating predicted vs observed inter-cell-type fold-differences.

## When to use

Use when scRNA-seq profiles lack miRNA measurements (poly(A) capture precludes mature miRNAs) and paired bulk mRNA-miRNA training cohorts exist for the tissue(s) of interest, so miRNA expression can be predicted from mRNA profiles.

## When NOT to use

- The goal is to infer miRNA activity from putative targets alone without a large cohort of paired miRNA-mRNA training data — miRSCAPE explicitly requires such training, unlike target-set enrichment approaches (e.g., miReact-style)
- The training tissue/context differs substantially from the application context — a substantive performance loss is expected (quantified in Figure S9)
- A pre-trained model is being applied to scRNA-seq with high dropout where most model genes are undetected — retraining on only reasonably detected genes may be required instead

## Inputs

- Paired bulk mRNA and bulk miRNA expression matrices (rows = genes/miRNAs, columns = samples) from TCGA tissue cohorts (e.g., brain and kidney)
- Seurat object with scRNA-seq profiles and cell type/cluster annotations (e.g., HCL brain and kidney cell types)
- R 4.1.0 environment with xgboost and Seurat installed

## Outputs

- Predicted miRNA expression matrix for each cell-type cluster (miRSCAPE $pred output)
- Spearman rank correlation between predicted and observed inter-cell-type miRNA fold-differences across 262 miRNAs (rho ≈ 0.8 tissue-specific; rho ≈ 0.73 pooled)

## How to apply

Prepare paired TCGA bulk mRNA and miRNA matrices with bulkTransform (log-transform only; set justNorm = TRUE for miRNA so it is not scaled), then train miRSCAPE XGBoost models (default booster, objective, maximum depth, eta, number of rounds) — separate tissue-specific models plus a pooled cross-tissue model. Prepare the scRNA-seq Seurat object with modifySeuratObject, pooling (bootstrapped sampling of) cells within each cluster to obtain non-sparse pseudo-bulk mRNA profiles. Apply the trained models via the miRSCAPE function and retrieve predicted miRNA expression from the pred output. Compute predicted and observed inter-cell-type miRNA fold-differences and report Spearman rank correlation across the 262 miRNAs; expect rho ≈ 0.8 for cell-type-specific (tissue-matched) models and rho ≈ 0.73 for the pooled model, consistent with the paper's ~0.81 benchmark.

## Related tools

- **miRSCAPE** (XGBoost-based tool that trains on paired bulk mRNA-miRNA data and predicts miRNA expression in single-cell clusters from their mRNA profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying gradient-boosting regression engine for miRSCAPE model training (booster, objective, max depth, eta, rounds parameters)) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Holds and prepares the scRNA-seq data; modifySeuratObject pools cells within clusters into non-sparse pseudo-bulk profiles) — https://github.com/hannenhalli-lab/miRSCAPE

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Spearman rho between predicted and observed kidney-versus-brain miRNA fold-changes across 262 miRNAs should be ≈ 0.8 with cell-type-specific models
- The pooled cross-tissue model should yield rho ≈ 0.73, i.e., slightly lower than tissue-matched models
- miRNA input matrix must be log-transformed but NOT scaled (justNorm = TRUE); scaled miRNA training data invalidates the regression target
- Predictions must be made on pooled (non-sparse) cluster profiles rather than individual cells, since scRNA dropout breaks the pre-trained model
- Both mRNA and miRNA training matrices must have gene/miRNA identifiers as row names

## Limitations

- miRSCAPE requires training on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that need no specific training
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; a substantive loss is expected when contexts differ greatly (Figure S9)
- Due to the large scRNA-seq dropout rate, a pre-trained model may not work and miRSCAPE may need retraining using only genes detected at a reasonable level
- Standard scRNA-seq protocols rely on poly(A) capture, which mature miRNAs lack, so miRNAs must be inferred rather than measured directly

## Evidence
