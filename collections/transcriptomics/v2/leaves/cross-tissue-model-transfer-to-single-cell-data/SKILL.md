---
name: cross-tissue-model-transfer-to-single-cell-data
description: Use when you need to chart miRNA activity across many cell types in a
  scRNA-seq atlas using a model trained on diverse bulk paired miRNA-mRNA cohorts,
  rather than tissue-matched training.
when_to_use_negative:
- When the application sample's cellular/tissue context matches a specific training
  cohort — a context-matched (non-global) model is expected to be more accurate
- When the scRNA-seq data has very high dropout such that a pre-trained model fails
  — miRSCAPE should instead be retrained using only genes detected at a reasonable
  level
- When the goal is inferring miRNA activity from putative target expression alone
  (e.g., target-set enrichment approaches) rather than a trained paired miRNA-mRNA
  model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Core function that trains the xgboost model on transformed bulk miRNA-mRNA
    matrices and predicts miRNA expression (returned in `pred`) from pooled scRNA-seq
    profiles
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: bulkTransform
  role: Log-transforms and normalizes the bulk training matrices; justNorm=TRUE is
    used for the miRNA matrix so it is only log-transformed, not scaled
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: modifySeuratObject
  role: Prepares Seurat-object scRNA-seq data for selected cell clusters (clusterId)
    or all clusters, keeping gene names as row names
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Gradient-boosting backend used by the miRSCAPE function; default booster/objective
    parameters are used unless tuning is warranted (bstr, objt, mdpth, ett, nrnds,
    esr)
- name: Seurat
  role: R package dependency for handling the scRNA-seq data (e.g., HCL) stored as
    a Seurat object
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:c38add6136675774c80cb58498f62470675346200005de84a9da7c473f33f04f
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# cross-tissue-model-transfer-to-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a single global miRSCAPE XGBoost model on a multi-tissue compendium of paired bulk miRNA-mRNA profiles (~10,000 TCGA/GTEx/CCLE samples across 10 tissues) and transfer it to bootstrapped, cell-type-pooled scRNA-seq profiles (e.g., the 56 cell types of the Human Cell Landscape) to predict 523 miRNA activities per cell type.

## When to use

Use when you need to chart miRNA activity across many cell types in a scRNA-seq atlas using a model trained on diverse bulk paired miRNA-mRNA cohorts, rather than tissue-matched training.

## When NOT to use

- When the application sample's cellular/tissue context matches a specific training cohort — a context-matched (non-global) model is expected to be more accurate
- When the scRNA-seq data has very high dropout such that a pre-trained model fails — miRSCAPE should instead be retrained using only genes detected at a reasonable level
- When the goal is inferring miRNA activity from putative target expression alone (e.g., target-set enrichment approaches) rather than a trained paired miRNA-mRNA model

## Inputs

- Paired bulk miRNA matrix (rows = miRNAs, columns = samples; e.g., TCGA/GTEx/CCLE ~10,000 samples across 10 tissues)
- Paired bulk mRNA matrix with gene names as row names
- scRNA-seq data stored in a Seurat object (e.g., Human Cell Landscape covering 56 cell types)
- Cell-type/cluster identifiers (clusterId) for the scRNA-seq object

## Outputs

- pred object containing predicted expression of 523 miRNAs for each cell-type pooled profile
- Ranked per-cell-type miRNA tables (top 10 up- and downregulated miRNAs per cell type, e.g., hcl_top10_mirnas_per_celltype.csv)

## How to apply

Load the paired bulk miRNA and mRNA training matrices (~10,000 tumor and normal samples spanning 10 tissues) and select ~1,000 representative samples as k-means medoids. Log-transform/normalize the bulk data with bulkTransform, setting justNorm=TRUE for the miRNA matrix (miRNA must not be scaled). Train the global miRSCAPE model — an xgboost-backed regressor — with default booster/objective parameters unless tuning is warranted. Prepare the target scRNA-seq data stored in a Seurat object with modifySeuratObject (retaining gene row names), pool cells within each cell-type cluster, and use bootstrapped pooled profiles to mitigate sparsity/dropout. Apply the trained model to each bootstrapped cell-type profile; the predicted 523 miRNA expressions are returned in `pred`, then rank per cell type and extract top 10 up- and downregulated miRNAs (e.g., saved as hcl_top10_mirnas_per_celltype.csv).

## Related tools

- **miRSCAPE** (Core function that trains the xgboost model on transformed bulk miRNA-mRNA matrices and predicts miRNA expression (returned in `pred`) from pooled scRNA-seq profiles) — https://github.com/hannenhalli-lab/miRSCAPE
- **bulkTransform** (Log-transforms and normalizes the bulk training matrices; justNorm=TRUE is used for the miRNA matrix so it is only log-transformed, not scaled) — https://github.com/hannenhalli-lab/miRSCAPE
- **modifySeuratObject** (Prepares Seurat-object scRNA-seq data for selected cell clusters (clusterId) or all clusters, keeping gene names as row names) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Gradient-boosting backend used by the miRSCAPE function; default booster/objective parameters are used unless tuning is warranted (bstr, objt, mdpth, ett, nrnds, esr))
- **Seurat** (R package dependency for handling the scRNA-seq data (e.g., HCL) stored as a Seurat object)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Sanity-check that recapitulated miRNA activities are consistent with known cell-type miRNA biology (e.g., stemness-associated miRNAs in pancreatic cancer, EMT-associated miRNAs in breast cancer)
- Compare predicted vs observed fold-difference correlation in datasets with cell-type miRNA profiles — the paper reports ~0.81 (HEK-GBM, Kidney-Breast-Skin)
- Verify `pred` contains predicted expression for 523 miRNAs across all 56 HCL cell types
- Confirm the ranked output table (hcl_top10_mirnas_per_celltype.csv) has top 10 up- and downregulated miRNAs for each of the 56 cell types
- Cross-validate the global model on held-out bulk samples from the 10-tissue compendium to confirm accuracy before transfer

## Limitations

- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not perfectly match those of the training cohort
- When the training context is very different from the application context, a more substantive loss in performance is expected (quantified in Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE needs to be retrained using only genes detected at a reasonable level
- Standard scRNA-seq protocols cannot capture miRNAs because reverse transcription relies on poly(A) capture, which mature miRNAs lack — predictions are inferred, not measured

## Evidence
