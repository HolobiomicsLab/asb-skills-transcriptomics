---
name: pretrained-model-transfer-to-scrna-data
description: Use when scRNA-seq data lack miRNA measurements (poly(A) capture precludes
  miRNA profiling) and you need to infer miRNA expression per cluster or per cell
  using a model pretrained on paired bulk miRNA-mRNA cohorts such as TCGA, GTEx, or
  CCLE.
when_to_use_negative:
- The application sample's cellular/tissue context differs substantially from the
  model's training cohort (e.g. applying a CCLE-trained model to very different tissue),
  where performance loss is expected per Figure S9
- Too few genes are detected in the single-cell data due to dropout, so a pre-trained
  model may fail and retraining on detected genes is required rather than transfer
- Paired miRNA-mRNA expression data already exist for the sample, making inference
  unnecessary
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: xgboost-based model inferring miRNA expression from mRNA profiles; applied
    to pooled or unpooled scRNA-seq data after training on paired bulk cohorts
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Stores and prepares the single-cell data; modifySeuratObject extracts and
    formats clusters for prediction
- name: xgboost
  role: Gradient-boosting backend for miRSCAPE models; parameters (booster, objective,
    max depth, eta, rounds, early stopping) are configurable via miRSCAPE function
    arguments
- name: miReact
  role: Cross-gene-correlation-based target-only control method run on the same induced/uninduced
    profiles to benchmark miRSCAPE predictions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:cd0de917fb85c935c753d82c6ceddf0a896b5e272175123ab24eaa784d679ae6
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
  - miReact
  - xgboost
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pretrained-model-transfer-to-scrna-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply a pretrained, bulk-trained machine learning model (miRSCAPE, xgboost-based) to single-cell RNA-seq data by pooling cells within clusters to overcome scRNA-seq sparsity, or by predicting per-cell in unpooled mode. Enables miRNA activity inference from mRNA-only scRNA-seq profiles.

## When to use

Use when scRNA-seq data lack miRNA measurements (poly(A) capture precludes miRNA profiling) and you need to infer miRNA expression per cluster or per cell using a model pretrained on paired bulk miRNA-mRNA cohorts such as TCGA, GTEx, or CCLE.

## When NOT to use

- The application sample's cellular/tissue context differs substantially from the model's training cohort (e.g. applying a CCLE-trained model to very different tissue), where performance loss is expected per Figure S9
- Too few genes are detected in the single-cell data due to dropout, so a pre-trained model may fail and retraining on detected genes is required rather than transfer
- Paired miRNA-mRNA expression data already exist for the sample, making inference unnecessary

## Inputs

- scRNA-seq expression data stored as a Seurat object (rows = genes with gene names as row names)
- Paired bulk mRNA and bulk miRNA training matrices (e.g. CCLE, TCGA, or GTEx profiles; pdac_mrna.txt / pdac_mirna.txt style .txt files)
- Cluster/cell-type labels for the scRNA data (e.g. clustId = c('Ductal cell type 1'))
- Induced vs uninduced cell group labels for differential validation

## Outputs

- Predicted miRNA expression values per cluster or per cell (the `pred` object from the miRSCAPE function)
- Induced-vs-uninduced log-fold changes and Wilcoxon p-values per miRNA per strategy (e.g. written to mir199a_logfc_results.csv)
- miReact control log-fold changes for comparison

## How to apply

First prepare the Seurat-stored scRNA data with modifySeuratObject in R 4.1.0 (optionally restricted to clusters of interest via the clusterId parameter). Pool cells within each cluster so the transcriptomic profile is no longer sparse, since the pretrained model (trained on TCGA/GTEx/CCLE bulk cohorts) expects non-sparse input; alternatively apply the model per cell in unpooled mode, accepting lower fold-change resolution due to dropout. Run the miRSCAPE function with bulkTransform-prepared bulk mRNA and bulk miRNA training data (miRNA log-transformed via justNorm=TRUE, not scaled) and the prepared scRNA matrix; predictions appear in the `pred` output. For validation of an induction experiment (e.g. miR-199a-5p/3p), compute induced-vs-uninduced log-fold changes and Wilcoxon p-values, and compare against a target-only control such as miReact, expecting pooled logFC values (e.g. 6.2, 6.0, 5.9, 4.9; p < E-72) far above the control's (~0.33 to -1.13).

## Related tools

- **miRSCAPE** (xgboost-based model inferring miRNA expression from mRNA profiles; applied to pooled or unpooled scRNA-seq data after training on paired bulk cohorts) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Stores and prepares the single-cell data; modifySeuratObject extracts and formats clusters for prediction)
- **xgboost** (Gradient-boosting backend for miRSCAPE models; parameters (booster, objective, max depth, eta, rounds, early stopping) are configurable via miRSCAPE function arguments)
- **miReact** (Cross-gene-correlation-based target-only control method run on the same induced/uninduced profiles to benchmark miRSCAPE predictions)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Pooled induced-vs-uninduced logFC for the target miRNA should be large and significant (e.g. 6.2, 6.0, 5.9, 4.9 with Wilcoxon p < E-72 in the miR-199a benchmark), far exceeding the miReact control (0.33, 0.15, -1.00, -1.13)
- Unpooled single-cell predictions should still rank induced cells higher (logFC 2.6, 2.6, 1.9, 1.5; p < E-36), though attenuated relative to pooled mode
- Predicted-vs-observed fold-difference correlation ~0.81 in independent scRNA datasets with known miRNA profiles (HEK-GBM, Kidney-Breast-Skin)
- Input matrices must have gene names as row names; miRNA training data must be log-transformed only (bulkTransform with justNorm=TRUE), not scaled
- All rows of computed logFC/p-values should be written to mir199a_logfc_results.csv and match the reported values

## Limitations

- miRSCAPE must be trained on a large cohort of paired miRNA-mRNA data, unlike target-based approaches such as miReact that need no training
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort; a more substantive loss occurs when contexts differ substantially (Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need retraining using only genes detected at a reasonable level
- Large cohorts of paired miRNA-protein expression data are unavailable, so a protein-based model cannot be trained or assessed

## Evidence
