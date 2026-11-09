---
name: matched-tissue-accuracy-comparison
description: Use when quantifying how sample/tissue heterogeneity affects miRSCAPE
  cross-validation accuracy, e.g. comparing pooled multi-cancer TCGA training against
  tissue-specific or matched GTEx normal configurations.
when_to_use_negative:
- Input is scRNA-seq data needing cluster-level miRNA prediction rather than bulk-cohort
  accuracy benchmarking — use the modifySeuratObject/bulkTransform/miRSCAPE prediction
  workflow instead
- The application sample's cellular/tissue context substantially differs from the
  training cohort, where Figure S9 shows a more substantive performance loss beyond
  what heterogeneity pooling compensates for
- No large paired miRNA-mRNA training cohort is available, since miRSCAPE must be
  trained on such data unlike target-based approaches (e.g. miReact)
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: miRSCAPE
  role: Machine learning tool whose XGBoost model is cross-validated on pooled and
    matched-tissue TCGA/GTEx configurations to measure accuracy
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: R 4.1.0
  role: Runtime environment required to run the miRSCAPE code
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying XGBoost learner trained with default booster, objective, depth,
    eta, rounds, and early-stopping parameters during cross-validation
- name: Seurat
  role: Package dependency of miRSCAPE used for scRNA-seq object handling; loaded
    in the initialization step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:ced626fe03d36f2e569d402df9d12a94676abc95eabd2935dbf7551f24ed0d4b
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# matched-tissue-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark miRSCAPE cross-validation accuracy across pooled versus tissue-specific and matched-normal configurations, using large paired bulk miRNA-mRNA cohorts from TCGA and GTEx to quantify how inter-sample heterogeneity improves model accuracy.

## When to use

Use when quantifying how sample/tissue heterogeneity affects miRSCAPE cross-validation accuracy, e.g. comparing pooled multi-cancer TCGA training against tissue-specific or matched GTEx normal configurations.

## When NOT to use

- Input is scRNA-seq data needing cluster-level miRNA prediction rather than bulk-cohort accuracy benchmarking — use the modifySeuratObject/bulkTransform/miRSCAPE prediction workflow instead
- The application sample's cellular/tissue context substantially differs from the training cohort, where Figure S9 shows a more substantive performance loss beyond what heterogeneity pooling compensates for
- No large paired miRNA-mRNA training cohort is available, since miRSCAPE must be trained on such data unlike target-based approaches (e.g. miReact)

## Inputs

- Paired bulk miRNA expression matrix from TCGA (8,089 samples, brain/breast/colon/liver/lung cancers), genes as row names
- Paired bulk mRNA expression matrix from the same TCGA samples, genes as row names
- Matched GTEx normal tissue miRNA and mRNA profiles for the five cancer types
- R 4.1.0 environment with xgboost and Seurat and the miRSCAPE.R source from github.com/hannenhalli-lab/miRSCAPE

## Outputs

- Average cross-validation accuracy per configuration (pooled TCGA ~0.62, TCGA-only ~0.34, GTEx-only ~0.18, pooled TCGA+GTEx ~0.54)
- Comparison table written to figure3_accuracy_table.csv reproducing Figure 3

## How to apply

Load paired bulk miRNA and mRNA cohorts with gene names as row names: 8,089 TCGA tumor samples spanning brain, breast, colon, liver, and lung cancers plus matched GTEx normal tissue profiles. Prepare each matrix with the miRSCAPE bulkTransform function in R 4.1.0, log-transforming mRNA data and setting justNorm=TRUE for miRNA so it is not scaled. Train the miRSCAPE XGBoost model with default booster (bstr), objective (objt), maximum depth (mdpth), eta (ett), rounds (nrnds), and early-stopping (esr) parameters under cross-validation on each configuration: pooled 5-cancer TCGA, individual TCGA cancer types alone, matched GTEx normals alone, and pooled TCGA+GTEx. The rationale is that pooling across heterogeneous samples expands the training context and improves accuracy, whereas tissue-mismatched training (e.g. GTEx normals) reduces it. Tabulate the resulting accuracy configurations into comparison tables reproducing Figure 3 and write them to figure3_accuracy_table.csv.

## Related tools

- **miRSCAPE** (Machine learning tool whose XGBoost model is cross-validated on pooled and matched-tissue TCGA/GTEx configurations to measure accuracy) — https://github.com/hannenhalli-lab/miRSCAPE
- **R 4.1.0** (Runtime environment required to run the miRSCAPE code) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying XGBoost learner trained with default booster, objective, depth, eta, rounds, and early-stopping parameters during cross-validation)
- **Seurat** (Package dependency of miRSCAPE used for scRNA-seq object handling; loaded in the initialization step)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna)
```

## Evaluation signals

- Pooled 5-cancer TCGA cohort (8,089 samples) yields average CV accuracy ~0.62, an average increase of ~0.2 over tissue-specific models
- TCGA-only cross-validation accuracy ~0.34 versus GTEx-only matched-tissue accuracy ~0.18, showing context-mismatched training reduces accuracy
- Pooled TCGA+GTEx configuration yields accuracy ~0.54
- figure3_accuracy_table.csv contains the three (four) accuracy configurations tabulated as in Figure 3
- Input matrices carry gene names as row names and miRNA data was transformed with justNorm=TRUE (log-transform only, not scaled)

## Limitations

- miRSCAPE is expected to have slightly reduced accuracy when the cellular/tissue context of the application sample does not perfectly match the training cohort
- A substantive loss in performance is expected when the training context differs substantially from the application context, as quantified in Figure S9
- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches

## Evidence
