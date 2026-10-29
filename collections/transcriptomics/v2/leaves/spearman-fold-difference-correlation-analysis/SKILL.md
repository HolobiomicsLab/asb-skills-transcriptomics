---
name: spearman-fold-difference-correlation-analysis
description: Use when validating inferred miRNA activity against measured single-cell
  or purified-cell-type miRNA profiles, by correlating predicted vs observed fold-differences
  between cell-type pairs across miRNAs.
when_to_use_negative:
- When no measured miRNA expression for the cell types of interest is available —
  the observed side of the fold-difference comparison cannot be computed
- When the training context differs substantially from the application context (e.g.
  cross-species or unmatched tissue), since miRSCAPE accuracy is expected to drop,
  per Figure S9 and the hematopoietic mouse case (auROC = 0.67)
- When the goal is differential expression of mRNAs rather than validating inferred
  miRNA activity — this skill only assesses miRNA prediction concordance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: miRSCAPE
  role: XGBoost-based model that predicts miRNA expression from mRNA profiles; its
    'pred' output supplies the predicted fold-differences being correlated
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Regression engine underlying miRSCAPE with default booster, objective, maximum
    depth, eta, and number of rounds parameters
- name: Seurat
  role: Handles scRNA-seq data as Seurat objects; modifySeuratObject pools (bootstrapped-samples)
    cells within each cluster into non-sparse pseudo-bulk profiles used for prediction
- name: R 4.1.0
  role: Runtime environment in which the miRSCAPE pipeline and Spearman correlation
    validation are executed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:b8330fb56cc1aebd549a84b3e3d7930d2900c5d9bb61d39094d3702401be9e2f
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spearman-fold-difference-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation strategy for miRSCAPE that quantifies agreement between miRSCAPE-predicted miRNA expression and observed cell-type-specific miRNA profiles by computing Spearman rank correlations between predicted and observed inter-cell-type fold-differences across miRNAs.

## When to use

Use when validating inferred miRNA activity against measured single-cell or purified-cell-type miRNA profiles, by correlating predicted vs observed fold-differences between cell-type pairs across miRNAs.

## When NOT to use

- When no measured miRNA expression for the cell types of interest is available — the observed side of the fold-difference comparison cannot be computed
- When the training context differs substantially from the application context (e.g. cross-species or unmatched tissue), since miRSCAPE accuracy is expected to drop, per Figure S9 and the hematopoietic mouse case (auROC = 0.67)
- When the goal is differential expression of mRNAs rather than validating inferred miRNA activity — this skill only assesses miRNA prediction concordance

## Inputs

- Predicted miRNA expression per cell type (pred output from miRSCAPE function, XGBoost regression)
- Observed miRNA expression profiles of matched cell types (e.g. HEK-GBM Faridani GSE81287 brain/kidney; Isakova GSE151334 kidney/breast/skin Smart-seq-total)
- Cell-type labels / cluster identities defining the cell-type pairs for fold-difference computation

## Outputs

- Per-cell-type-pair Spearman rank correlation (rho) between predicted and observed miRNA fold-differences
- Comparison of rho across modeling strategies (cell-type-specific tissue-matched models vs single pooled model)

## How to apply

After training miRSCAPE XGBoost regression models on paired bulk miRNA-mRNA matrices (e.g. TCGA bulk data prepared with bulkTransform, setting justNorm=TRUE for miRNA so it is only log-transformed), apply the models to pooled non-sparse scRNA-seq mRNA profiles from cell clusters prepared with modifySeuratObject. Retrieve predicted miRNA expression from the 'pred' output of the miRSCAPE function, which uses default XGBoost settings (booster, objective, maximum depth, eta, number of rounds). For each cell-type pair (e.g. kidney vs brain, or kidney, breast, and skin pairs), compute predicted fold-differences and observed fold-differences across all miRNAs (e.g. the 262 miRNAs in the HCL brain/kidney evaluation) and correlate them with Spearman rank correlation. Judge success against the published benchmarks: ~0.81 average correlation for cell-type-specific inference (rho ≈ 0.8 with cell-type-specific models and rho ≈ 0.73 with a single pooled model on HCL brain/kidney; 0.76–0.89 per pair on Isakova Smart-seq-total kidney/breast/skin).

## Related tools

- **miRSCAPE** (XGBoost-based model that predicts miRNA expression from mRNA profiles; its 'pred' output supplies the predicted fold-differences being correlated) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Regression engine underlying miRSCAPE with default booster, objective, maximum depth, eta, and number of rounds parameters)
- **Seurat** (Handles scRNA-seq data as Seurat objects; modifySeuratObject pools (bootstrapped-samples) cells within each cluster into non-sparse pseudo-bulk profiles used for prediction)
- **R 4.1.0** (Runtime environment in which the miRSCAPE pipeline and Spearman correlation validation are executed)

## Examples

```
denem = modifySeuratObject(pbmc = example, clusterId = clustId); bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Spearman rho ≈ 0.8 between predicted and observed kidney-vs-brain fold-changes across 262 miRNAs with cell-type-specific TCGA-trained models on HCL data
- Spearman rho ≈ 0.73 for the single pooled cross-tissue model on the same HCL brain/kidney comparison
- Per-pair Spearman correlations fall in the 0.76–0.89 range on Isakova kidney/breast/skin Smart-seq-total data, consistent with the ~0.81 benchmark
- Predictions are read from the 'pred' element returned by the miRSCAPE function, confirming the correct output slot is used
- miRNA bulk input was log-transformed only (justNorm=TRUE), not scaled, ensuring fold-differences are computed on the intended scale

## Limitations

- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not match the training cohort, and a more substantive loss when the contexts differ substantially (Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and may need to be retrained using only genes detected at a reasonable level
- Standard scRNA-seq protocols rely on poly(A) capture, which mature miRNAs lack, so miRNA validation is only possible against datasets with measured cell-type-specific miRNA profiles
- miRSCAPE must be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches such as miReact

## Evidence
