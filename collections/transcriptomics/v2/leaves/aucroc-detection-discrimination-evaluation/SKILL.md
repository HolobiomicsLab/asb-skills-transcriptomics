---
name: aucroc-detection-discrimination-evaluation
description: Use when you have predicted miRNA expression values from mRNA profiles
  (e.g., miRSCAPE output) plus ground-truth detection calls (detected/non-zero vs
  undetected) per sample or cell type, and you want an auROC-based accuracy metric.
when_to_use_negative:
- Predictions must be validated against continuous measured miRNA abundance correlations
  (e.g., fold-difference correlation ~0.81) rather than binary detection calls
- The ground truth lacks detection labels, so no detected/undetected classification
  can be constructed from measured miRNA values
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: xgboost-based model that predicts miRNA expression from mRNA profiles; its
    predicted values serve as the ranking scores evaluated by auROC
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Runtime environment in which miRSCAPE, xgboost, and Seurat libraries are loaded
    and auROC computation is performed
- name: xgboost
  role: Dependency providing the gradient-boosted models used by miRSCAPE to generate
    predicted miRNA values
- name: Seurat
  role: Handles scRNA-seq data; modifySeuratObject prepares and pools mouse hematopoietic
    cell-type clusters before prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:534a411d824e2d591d9e5072e441a860cf9cf17726597ff52dbd63cb7d6c9db1
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# aucroc-detection-discrimination-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates predicted miRNA expression (e.g., from miRSCAPE) by computing the auROC of predictions as a score separating experimentally detected (non-zero) miRNAs from undetected miRNAs, averaged across ENCODE cell lines (~0.71) and mouse hematopoietic cell types (~0.67).

## When to use

Use when you have predicted miRNA expression values from mRNA profiles (e.g., miRSCAPE output) plus ground-truth detection calls (detected/non-zero vs undetected) per sample or cell type, and you want an auROC-based accuracy metric.

## When NOT to use

- Predictions must be validated against continuous measured miRNA abundance correlations (e.g., fold-difference correlation ~0.81) rather than binary detection calls
- The ground truth lacks detection labels, so no detected/undetected classification can be constructed from measured miRNA values

## Inputs

- miRSCAPE-predicted miRNA expression matrix (rows = miRNAs, columns = cell lines or pooled cell-type clusters)
- Experimentally measured miRNA profiles (McCall measurements for ENCODE cell lines) with non-zero values interpretable as detected
- Detection labels derived from measured miRNA values (detected = non-zero, undetected = zero)
- Per-sample groupings: 10 ENCODE cell lines; mouse hematopoietic cell types from pooled Zilionis GSE127465 clusters

## Outputs

- Per-cell-line auROC values (expected average ~0.71 across 10 ENCODE cell lines)
- Per-cell-type auROC values for mouse hematopoietic cell types (expected average ~0.67)
- Summary table written to encode_zilionis_auroc_results.csv

## How to apply

Run the predictor (miRSCAPE with the CCLE-trained model for ENCODE/McCall cell-line bulk data, or the AML-TCGA-trained model for pooled Zilionis GSE127465 mouse hematopoietic scRNA-seq clusters) to obtain per-miRNA predicted expression values. Label each miRNA in the experimental (McCall) measurements as detected if its measured expression is non-zero and undetected otherwise. Per cell line or per pooled cell-type cluster, compute the auROC of the predicted values for discriminating detected vs undetected miRNAs, then average auROCs across the 10 ENCODE cell lines and/or across mouse hematopoietic cell types. Write the per-sample and per-cell-type auROC tables to a results file (encode_zilionis_auroc_results.csv). Rationale: auROC is threshold-free and robust to the scale of predicted values, directly quantifying whether predictions rank truly expressed miRNAs above unexpressed ones.

## Related tools

- **miRSCAPE** (xgboost-based model that predicts miRNA expression from mRNA profiles; its predicted values serve as the ranking scores evaluated by auROC) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Runtime environment in which miRSCAPE, xgboost, and Seurat libraries are loaded and auROC computation is performed)
- **xgboost** (Dependency providing the gradient-boosted models used by miRSCAPE to generate predicted miRNA values)
- **Seurat** (Handles scRNA-seq data; modifySeuratObject prepares and pools mouse hematopoietic cell-type clusters before prediction)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Average auROC across the 10 ENCODE cell lines should approximate 0.71
- Average auROC across mouse hematopoietic cell types should approximate 0.67
- Each miRNA must be assigned a binary detected/undetected label derived from non-zero McCall measurement values
- auROC must be computed per cell line / per cell type before averaging, and the per-sample values must be present in encode_zilionis_auroc_results.csv
- Predictions for scRNA-seq data must come from AML-TCGA-trained model applied to pooled (non-sparse) cell-type profiles, not single cells

## Limitations

- Detection labels (non-zero measured expression) are a coarse binary proxy; auROC here measures detection discrimination, not quantitative accuracy of predicted miRNA abundance
- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not match the training cohort
- Due to large scRNA-seq dropout, a pre-trained model may not work and retraining on detected genes may be required
- When the training context is very different from the application context, a more substantive loss in performance is expected (Figure S9)

## Evidence

- [full_text] In the 10 ENCODE cell lines, miRSCAPE's predicted values significantly discriminate miRNAs with detected (non-zero) expression from undetected miRNAs, with reasonable accuracy (average auROC = 0.71).: [evidence span withheld — anchor status: not_found]
- [readme] When trained on human hematopoietic cancers, miRSCAPE can identify active miRNAs in 8 hematopoietic cell lines in mouse with a reasonable accuracy (auROC = 0.67).: "When trained on human hematopoietic cancers, miRSCAPE can identify active miRNAs in 8 hematopoietic cell lines in mouse with a reasonable accuracy (auROC = 0.67)."
- [discussion] we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity: "we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity"
- [readme] Don't scale the miRNA data, please set `justNorm` parameter to `TRUE` in miRNA data.: "Don't scale the miRNA data, please set `justNorm` parameter to `TRUE` in miRNA data."
- [readme] To predict the miRNAs, `miRSCAPE` function should be used. The predicted miRNA expressions can be found in `pred`.: "To predict the miRNAs, `miRSCAPE` function should be used. The predicted miRNA expressions can be found in `pred`."
