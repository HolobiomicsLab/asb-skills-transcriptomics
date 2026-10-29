---
name: k-means-medoid-sample-selection
description: Use when training miRSCAPE on paired bulk miRNA–mRNA matrices spanning
  thousands of tumor and normal samples and you need a compact, representative training
  set (k-means medoids) before xgboost model fitting.
when_to_use_negative:
- The training cohort is already small (e.g. the pdac_mirna/pdac_mrna example data)
  — medoid subsampling is meant for ~10,000-sample compendia and would discard data
  unnecessarily
- The application scRNA-seq context differs strongly from the training tissues — miRSCAPE
  accuracy drops when training and application contexts do not match, so a medoid-selected
  multi-tissue model may not be the right training set
- You need per-sample (not pooled) predictions and cannot afford any loss of cohort
  granularity from collapsing clusters to medoids
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: miRSCAPE
  role: Consumes the k-means-medoid-selected paired bulk miRNA–mRNA matrices as training
    data for its xgboost-based miRNA prediction model
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: bulkTransform
  role: Log-transforms/normalizes the medoid-selected bulk matrices before training;
    justNorm=TRUE for the miRNA matrix
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Backend learner of miRSCAPE that is trained on the medoid subsample; default
    booster/objective parameters unless tuning is warranted
- name: R 4.1.0
  role: Runtime environment in which k-means medoid selection and the miRSCAPE pipeline
    are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:2901457f1d2da93fff3ab4e8dd8483e005000156976b3b4d04ad86cc7fdb3094
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
  - bulkTransform
  - miRSCAPE
  - xgboost
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# k-means-medoid-sample-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces a large paired bulk miRNA–mRNA training compendium (e.g. ~10,000 TCGA/GTEx/CCLE tumor and normal samples across 10 tissues) to ~1,000 representative samples chosen as k-means medoids, so the miRSCAPE xgboost model can be trained efficiently without losing cohort diversity.

## When to use

Use when training miRSCAPE on paired bulk miRNA–mRNA matrices spanning thousands of tumor and normal samples and you need a compact, representative training set (k-means medoids) before xgboost model fitting.

## When NOT to use

- The training cohort is already small (e.g. the pdac_mirna/pdac_mrna example data) — medoid subsampling is meant for ~10,000-sample compendia and would discard data unnecessarily
- The application scRNA-seq context differs strongly from the training tissues — miRSCAPE accuracy drops when training and application contexts do not match, so a medoid-selected multi-tissue model may not be the right training set
- You need per-sample (not pooled) predictions and cannot afford any loss of cohort granularity from collapsing clusters to medoids

## Inputs

- Paired bulk miRNA expression matrix (genes/miRNAs as row names, patients as columns)
- Paired bulk mRNA expression matrix (gene row names required)
- Cohort metadata spanning ~10,000 tumor and normal samples across 10 tissues (TCGA, GTEx, CCLE)

## Outputs

- Matrix of 1,000 k-means medoid samples (representative paired miRNA–mRNA profiles) used as the miRSCAPE training set
- Downstream: trained global miRSCAPE xgboost model ready to predict 523 miRNAs from scRNA-seq profiles

## How to apply

Load the paired bulk miRNA and mRNA training matrices (rows = genes/miRNAs, columns = patient samples) covering ~10,000 tumor and normal samples across 10 tissues from TCGA, GTEx, and CCLE. Run k-means clustering over the sample-space and select 1,000 representative samples as k-means medoids, so each cluster's medoid stands in for a coherent subgroup of the cohort. Transform the selected matrices with bulkTransform (justNorm=TRUE for the miRNA matrix to log-transform without scaling; default log-transform/normalization for mRNA) before feeding them to the miRSCAPE function backed by xgboost. The medoid subsampling preserves the multi-tissue diversity that miRSCAPE's supervised model needs while keeping xgboost training tractable.

## Related tools

- **miRSCAPE** (Consumes the k-means-medoid-selected paired bulk miRNA–mRNA matrices as training data for its xgboost-based miRNA prediction model) — https://github.com/hannenhalli-lab/miRSCAPE
- **bulkTransform** (Log-transforms/normalizes the medoid-selected bulk matrices before training; justNorm=TRUE for the miRNA matrix) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Backend learner of miRSCAPE that is trained on the medoid subsample; default booster/objective parameters unless tuning is warranted)
- **R 4.1.0** (Runtime environment in which k-means medoid selection and the miRSCAPE pipeline are executed)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Selected set contains exactly ~1,000 medoid samples drawn proportionally from the ~10,000 tumor and normal samples across 10 tissues
- Every selected medoid sample has both a paired miRNA and mRNA profile, with gene names present as row names as required by miRSCAPE
- bulkTransform was applied with justNorm=TRUE on the miRNA matrix (log-transform only, unscaled) and default processing on the mRNA matrix
- Trained miRSCAPE model reproduces the expected behavior: predicted-vs-observed fold-difference correlation ~0.81 on independent scRNA-seq validation datasets (HEK-GBM, Kidney-Breast-Skin)
- Cross-validation accuracy of the medoid-trained model remains comparable to the full-cohort model across the 10 tumor/normal cohorts

## Limitations

- miRSCAPE (and hence the medoid-selected training set) is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not match the training cohort; a substantive performance loss occurs when training context differs substantially (Figure S9)
- Given the large scRNA-seq dropout rate, a model trained on the medoid subset may not work and miRSCAPE may need retraining using only genes detected at a reasonable level
- Medoid subsampling is a compression of the cohort; rare subgroups not captured as medoids may be under-represented relative to the full ~10,000-sample training cohort

## Evidence

- [other] Load the paired bulk miRNA and mRNA training matrices covering ~10,000 tumor and normal samples across 10 tissues and select 1,000 representative samples as k-means medoids.: [evidence span withheld — anchor status: not_found]
