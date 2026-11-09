---
name: tool-benchmark-accuracy-comparison-analysis
description: Use when two miRNA-inference or similar transcriptomic-prediction tools
  must be benchmarked head-to-head on the same paired expression cohorts, with per-feature
  accuracy tables and scatter plots as deliverables.
when_to_use_negative:
- Input is single-cell (sparse) scRNA-seq data with large dropout — retraining on
  detected genes is required rather than the bulk-cohort benchmark protocol
- The goal is to apply miRSCAPE to infer miRNA activity in new samples rather than
  to compare tool accuracies on a common benchmark cohort
- No paired miRNA-mRNA expression profiles are available, so per-miRNA prediction
  accuracy cannot be computed for either tool
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: XGBoost-based tool trained on pooled bulk mRNA/miRNA data to predict per-sample
    miRNA expression; benchmarked arm whose top-50 accuracy (~0.86) is compared
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: miReact
  role: Comparator tool that ascertains miRNA activity via cross-gene correlation
    with putative targets; its top-50 accuracy (~0.24) provides the baseline
- name: xgboost
  role: R dependency providing the default-gradient-boosted regression backend used
    by miRSCAPE with default booster, objective, depth, eta, rounds, and early stopping
- name: Seurat
  role: R dependency for miRSCAPE data preparation (modifySeuratObject) when scRNA
    inputs are used; listed under Depends
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:82c5522e9a6bbff1b593c744562a99c1cb6d24bcb0c7a4f9182a2699c9102ef2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tool-benchmark-accuracy-comparison-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the per-miRNA prediction accuracy of miRSCAPE (XGBoost-based miRNA inference) against the target-correlation-based tool miReact on pooled TCGA bulk mRNA/miRNA cohorts, quantifying top-50 average accuracy to recapitulate the reported ~0.86 vs ~0.24 gap.

## When to use

Use when two miRNA-inference or similar transcriptomic-prediction tools must be benchmarked head-to-head on the same paired expression cohorts, with per-feature accuracy tables and scatter plots as deliverables.

## When NOT to use

- Input is single-cell (sparse) scRNA-seq data with large dropout — retraining on detected genes is required rather than the bulk-cohort benchmark protocol
- The goal is to apply miRSCAPE to infer miRNA activity in new samples rather than to compare tool accuracies on a common benchmark cohort
- No paired miRNA-mRNA expression profiles are available, so per-miRNA prediction accuracy cannot be computed for either tool

## Inputs

- Paired TCGA bulk mRNA expression matrix (genes as row names, patients as columns)
- Paired TCGA bulk miRNA expression matrix (miRNAs as row names, patients as columns)
- miRSCAPE R package (source ~/miRSCAPE-main/code/miRSCAPE.R) and miReact tool

## Outputs

- accuracy_comparison.csv (per-miRNA accuracy values for miRSCAPE and miReact)
- accuracy_scatter.png (per-miRNA accuracy scatter plot)
- Top-50 average accuracy per tool (expected ~0.86 for miRSCAPE vs ~0.24 for miReact)

## How to apply

Load paired TCGA bulk mRNA and bulk miRNA matrices for the five cancer types, with genes/miRNAs as row names and patients as columns, and pool them into a single cohort. Preprocess both matrices with miRSCAPE's bulkTransform function, log-transforming only (justNorm=TRUE for miRNA; do not scale miRNA). Train miRSCAPE with default XGBoost parameters (default booster, objective, maximum depth, eta, number of rounds, and early stopping) to predict per-sample miRNA expression, and run miReact on the same pooled profiles to infer miRNA activity via cross-gene correlation with putative targets. Compute per-miRNA accuracy for each tool across the five cancer types, derive each method's top-50 average accuracy, write the comparison table to accuracy_comparison.csv, and render the per-miRNA scatter plot to accuracy_scatter.png.

## Related tools

- **miRSCAPE** (XGBoost-based tool trained on pooled bulk mRNA/miRNA data to predict per-sample miRNA expression; benchmarked arm whose top-50 accuracy (~0.86) is compared) — https://github.com/hannenhalli-lab/miRSCAPE
- **miReact** (Comparator tool that ascertains miRNA activity via cross-gene correlation with putative targets; its top-50 accuracy (~0.24) provides the baseline)
- **xgboost** (R dependency providing the default-gradient-boosted regression backend used by miRSCAPE with default booster, objective, depth, eta, rounds, and early stopping)
- **Seurat** (R dependency for miRSCAPE data preparation (modifySeuratObject) when scRNA inputs are used; listed under Depends)

## Examples

```
pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Top-50 average accuracy of miRSCAPE on the pooled TCGA cohorts is ~0.86, recapitulating the published benchmark
- Top-50 average accuracy of miReact on the same pooled profiles is ~0.24
- accuracy_comparison.csv contains per-miRNA accuracy columns for both tools across the five cancer types
- accuracy_scatter.png renders the per-miRNA accuracy comparison and most points lie far above the diagonal in miRSCAPE's favor
- Both tools consumed identically preprocessed matrices (bulkTransform applied; miRNA log-transformed only with justNorm=TRUE and not scaled) so the comparison is apples-to-apples

## Limitations

- miReact was designed to predict miRNA expression in single cells and its authors interpreted predicted miRNAs at the cell-cluster level, so bulk pooled-cohort comparison may not reflect its intended use
- miRSCAPE accuracy drops when the cellular/tissue context of the application sample does not match the training cohort (see Figure S9 for quantified loss)
- miRSCAPE must be trained on a large cohort of paired miRNA-mRNA data; target-based approaches like miReact require no specific training

## Evidence

- [results] miReact achieves an average accuracy of only 0.24 for its top 50 most predictable miRNAs, whereas miRSCAPE achieves an average accuracy of 0.86: [evidence span withheld — anchor status: not_found]
