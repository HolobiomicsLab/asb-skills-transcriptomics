---
name: cross-species-pretrained-model-transfer-evaluation
description: Use when you have human-trained miRSCAPE models (xgboost-based) and need
  to test transfer to non-training transcriptomic contexts — e.g. ENCODE cell-line
  mRNA profiles or mouse scRNA-seq clusters — with binary detectability labels for
  miRNAs.
when_to_use_negative:
- The application sample's cellular/tissue context closely matches an available training
  cohort — retraining or context-matched models are preferred over cross-context transfer
- Your scRNA-seq data has severe dropout and no pooling is possible; the article notes
  a pre-trained model may not work and retraining on detected genes may be required
- You need miRNA expression measurements rather than model-based inference — standard
  scRNA-seq protocols rely on poly(A) capture which mature miRNAs lack, but this skill
  produces predictions, not measurements
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: miRSCAPE
  role: Pre-trained xgboost models applied to bulk mRNA (CCLE-trained) and pooled
    scRNA-seq profiles (AML-TCGA-trained) to predict miRNA expression across species
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: bulkTransform
  role: Log-transforms mRNA data; with justNorm=TRUE log-transforms miRNA data without
    scaling for detectability labeling
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: modifySeuratObject
  role: Pools cells within each mouse hematopoietic cell type cluster in the Seurat
    object to produce non-sparse transcriptomic profiles for prediction
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Gradient-boosting backend underlying the miRSCAPE prediction models; default
    booster/objective/depth/eta parameters are used
- name: Seurat
  role: R library required for handling the GSE127465 Seurat object prior to modifySeuratObject
    pooling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:1ae0c941506872afb43be9b914370edd40c11830e98455793108bd1fd7398368
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-species-pretrained-model-transfer-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate whether miRSCAPE models pre-trained on human paired miRNA-mRNA bulk cohorts (CCLE, AML-TCGA) transfer across species and contexts by predicting miRNA expression for ENCODE cell lines and Zilionis mouse hematopoietic cell types, and scoring predicted values by auROC for detected vs undetected miRNAs.

## When to use

Use when you have human-trained miRSCAPE models (xgboost-based) and need to test transfer to non-training transcriptomic contexts — e.g. ENCODE cell-line mRNA profiles or mouse scRNA-seq clusters — with binary detectability labels for miRNAs.

## When NOT to use

- The application sample's cellular/tissue context closely matches an available training cohort — retraining or context-matched models are preferred over cross-context transfer
- Your scRNA-seq data has severe dropout and no pooling is possible; the article notes a pre-trained model may not work and retraining on detected genes may be required
- You need miRNA expression measurements rather than model-based inference — standard scRNA-seq protocols rely on poly(A) capture which mature miRNAs lack, but this skill produces predictions, not measurements

## Inputs

- Pre-trained miRSCAPE models (CCLE-trained and AML-TCGA-trained) as R objects from the miRSCAPE repository
- ENCODE cell-line bulk mRNA profiles (genes as row names, samples as columns)
- McCall measured miRNA profiles for the ENCODE cell lines (for detectability labels)
- Zilionis GSE127465 mouse hematopoietic scRNA-seq data stored as a Seurat object
- R 4.1.0 with xgboost and Seurat libraries

## Outputs

- Predicted miRNA expression values per cell line / pooled cell type (miRSCAPE `pred` output)
- Per-cell-line and per-cell-type auROC tables for detected vs undetected miRNA discrimination
- encode_zilionis_auroc_results.csv with averaged auROC (~0.71 ENCODE, ~0.67 mouse)

## How to apply

In R 4.1.0 with xgboost and Seurat, load the pre-trained CCLE- and AML-TCGA-trained miRSCAPE models from github.com/hannenhalli-lab/miRSCAPE. Prepare bulk mRNA with bulkTransform (log-transform) and measured miRNA profiles with bulkTransform(justNorm=TRUE) — do not scale the miRNA data. For scRNA-seq input (Zilionis GSE127465 mouse hematopoietic data), pool cells within each cell type cluster using modifySeuratObject, since pooling yields a non-sparse profile suitable for the pre-trained model. Run the miRSCAPE function with the appropriate model (CCLE-trained for the 10 ENCODE cell lines; AML-TCGA-trained for mouse hematopoietic cell types) and read predictions from `pred`. Label each miRNA as detected (non-zero measured expression) or undetected in McCall measurements, compute the auROC of predicted expression for this discrimination per sample/cell type, average across the 10 cell lines (expect ~0.71) and across mouse cell types (expect ~0.67), and write per-sample auROC tables to encode_zilionis_auroc_results.csv.

## Related tools

- **miRSCAPE** (Pre-trained xgboost models applied to bulk mRNA (CCLE-trained) and pooled scRNA-seq profiles (AML-TCGA-trained) to predict miRNA expression across species) — https://github.com/hannenhalli-lab/miRSCAPE
- **bulkTransform** (Log-transforms mRNA data; with justNorm=TRUE log-transforms miRNA data without scaling for detectability labeling) — https://github.com/hannenhalli-lab/miRSCAPE
- **modifySeuratObject** (Pools cells within each mouse hematopoietic cell type cluster in the Seurat object to produce non-sparse transcriptomic profiles for prediction) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Gradient-boosting backend underlying the miRSCAPE prediction models; default booster/objective/depth/eta parameters are used)
- **Seurat** (R library required for handling the GSE127465 Seurat object prior to modifySeuratObject pooling)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Average auROC across the 10 ENCODE cell lines is approximately 0.71 for detected vs undetected miRNA discrimination using the CCLE-trained model
- Average auROC across mouse hematopoietic cell types is approximately 0.67 using the AML-TCGA-trained model (README reports 'reasonable accuracy (auROC = 0.67)')
- miRNA input must remain unscaled: bulkTransform was called with justNorm=TRUE (log-transform only), otherwise auROC comparisons are invalid
- scRNA-seq predictions are only valid on pooled (non-sparse) per-cluster profiles produced by modifySeuratObject, not on individual sparse cells
- Output table encode_zilionis_auroc_results.csv contains one auROC row per cell line and per mouse cell type before averaging

## Limitations

- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not perfectly match those of the training cohort
- When the training context is very different from the application context, a more substantive loss in performance is expected (quantified in Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need to be retrained using only genes detected at a reasonable level
- Cross-species transfer (human-trained model applied to mouse hematopoietic cell lines) yields reasonable but reduced accuracy (auROC ~0.67 vs ~0.71 in-context)

## Evidence
