---
name: cross-cohort-tcga-data-pooling-and-harmonization
description: Use when you have paired bulk mRNA/miRNA expression matrices spanning
  several TCGA cancer types and need one harmonized pooled cohort (genes/miRNAs as
  row names, patients as columns) to compare per-miRNA prediction accuracy across
  methods.
when_to_use_negative:
- Input is single-cell (scRNA-seq) data stored in a Seurat object — use modifySeuratObject-based
  scRNA preparation instead of pooling bulk TCGA cohorts
- The training and application cellular/tissue contexts differ substantially — the
  article warns miRSCAPE accuracy drops when the application sample does not match
  the training cohort (Figure S9)
- Only one cancer type or an unpaired miRNA/mRNA dataset is available, so a pooled
  pan-cancer accuracy comparison is not meaningful
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Provides bulkTransform used to log-transform and harmonize the pooled TCGA
    mRNA/miRNA matrices, and the XGBoost-based prediction used for the pooled-cohort
    accuracy benchmark
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: miReact
  role: Baseline tool whose cross-gene correlation-based miRNA activity inference
    is evaluated on the same pooled transcriptomic profiles
- name: xgboost
  role: Underlying learner for miRSCAPE prediction on the pooled cohort (default booster,
    objective, maximum depth, eta, rounds, early stopping)
- name: Seurat
  role: Dependency of the miRSCAPE R package used for expression data handling during
    preparation steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:809a9b1ae54fed480a9171a7861433bc89f3f53f5a54543ae04addaae5836c94
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Cross-cohort TCGA data pooling and harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pools paired TCGA bulk mRNA and bulk miRNA expression matrices from multiple cancer types into a single cohort, harmonizes them with miRSCAPE's bulkTransform log-transform (justNorm=TRUE for miRNA, no miRNA scaling), so per-sample miRNA prediction accuracy (e.g. miRSCAPE ~0.86 vs miReact ~0.24 top-50 average) can be benchmarked pan-cancer.

## When to use

Use when you have paired bulk mRNA/miRNA expression matrices spanning several TCGA cancer types and need one harmonized pooled cohort (genes/miRNAs as row names, patients as columns) to compare per-miRNA prediction accuracy across methods.

## When NOT to use

- Input is single-cell (scRNA-seq) data stored in a Seurat object — use modifySeuratObject-based scRNA preparation instead of pooling bulk TCGA cohorts
- The training and application cellular/tissue contexts differ substantially — the article warns miRSCAPE accuracy drops when the application sample does not match the training cohort (Figure S9)
- Only one cancer type or an unpaired miRNA/mRNA dataset is available, so a pooled pan-cancer accuracy comparison is not meaningful

## Inputs

- Paired TCGA bulk mRNA expression matrices for the five cancer types (genes as row names, patients as columns)
- Paired TCGA bulk miRNA expression matrices for the same five cancer types (miRNAs as row names, patients as columns)

## Outputs

- Single pooled, bulkTransform-harmonized mRNA/miRNA cohort ready for miRSCAPE/miReact prediction
- Per-miRNA accuracy comparison table written to accuracy_comparison.csv
- Per-miRNA accuracy scatter plot rendered to accuracy_scatter.png
- Top-50 average accuracy per method (expected ~0.86 for miRSCAPE vs ~0.24 for miReact)

## How to apply

Load each cancer type's paired TCGA bulk mRNA and bulk miRNA matrices, verify genes/miRNAs are row names and patient values are columns, then concatenate the samples into a single pooled cohort. Harmonize both matrices with miRSCAPE's bulkTransform function: log-transform only, setting justNorm=TRUE for the miRNA data and explicitly NOT scaling the miRNA matrix, as the README prescribes. Run the per-miRNA prediction on the pooled cohort (e.g. miRSCAPE with default XGBoost booster, objective, maximum depth, eta, number of rounds and early stopping, versus miReact's cross-gene correlation over putative targets). Compute per-miRNA accuracy across the five cancer types, derive each tool's top-50 most predictable miRNA average accuracy, and write the comparison table to accuracy_comparison.csv plus a per-miRNA scatter plot to accuracy_scatter.png.

## Related tools

- **miRSCAPE** (Provides bulkTransform used to log-transform and harmonize the pooled TCGA mRNA/miRNA matrices, and the XGBoost-based prediction used for the pooled-cohort accuracy benchmark) — https://github.com/hannenhalli-lab/miRSCAPE
- **miReact** (Baseline tool whose cross-gene correlation-based miRNA activity inference is evaluated on the same pooled transcriptomic profiles)
- **xgboost** (Underlying learner for miRSCAPE prediction on the pooled cohort (default booster, objective, maximum depth, eta, rounds, early stopping))
- **Seurat** (Dependency of the miRSCAPE R package used for expression data handling during preparation steps)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna)
```

## Evaluation signals

- Recapitulate the reported benchmark: pooled top-50 average accuracy ~0.86 for miRSCAPE versus ~0.24 for miReact
- Schema check: both input matrices have genes/miRNAs as row names and patient values as columns before pooling
- Harmonization check: miRNA matrix is log-transformed with justNorm=TRUE and is NOT scaled, per the README's bulkTransform prescription
- Output check: accuracy_comparison.csv contains per-miRNA accuracy for each tool across all five cancer types and accuracy_scatter.png renders the per-miRNA comparison

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, so the pooled TCGA cohort is a prerequisite rather than an optional convenience
- Model accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort, and a substantive loss is expected when contexts differ substantially (Figure S9)
- Given the large dropout rate in scRNA-seq, a pre-trained pooled-cohort model may not work and retraining on detected genes may be required

## Evidence
