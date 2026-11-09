---
name: mirna-target-correlation-activity-scoring
description: Use when inferring miRNA activity from paired or single-sample mRNA profiles
  with correlation-to-putative-target scoring (miReact), or when benchmarking such
  target-only methods against a trained predictor like miRSCAPE.
when_to_use_negative:
- 'You need cell-level resolution, not cluster-level summaries: miReact predictions
  in single cells were only interpretable at the level of cell clusters'
- You have no large paired miRNA-mRNA cohort and require a trained model — this target-only
  correlation approach is explicitly contrasted with miRSCAPE, which 'needs to be
  trained on a large cohort of paired miRNA-mRNA data'
- Your hypothesis requires detecting induced miRNA expression with high sensitivity
  — target-only scoring returned logFC of only 0.33, 0.15, -1.00, -1.13 where pooled
  miRSCAPE returned logFC 6.2, 6.0, 5.9, 4.9
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: miReact
  role: Cross-gene-correlation-based target-only method used to infer miRNA activity;
    serves as the baseline/control in accuracy and induction benchmarks
- name: miRSCAPE
  role: XGBoost-trained comparator that predicts miRNA expression from mRNA profiles,
    benchmarked against the correlation-based scoring
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Handles Seurat-object scRNA-seq inputs via modifySeuratObject before profiles
    are pooled for scoring
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying learner for the miRSCAPE comparator trained on paired bulk miRNA-mRNA
    cohorts
  repo: https://github.com/hannenhalli-lab/miRSCAPE
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:220cf875b1136d21603335be74ec3a24ec494533f84f7d44c497ceca4d38ced9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# mirna-target-correlation-activity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Target-only inference of miRNA activity from a transcriptomic profile via cross-gene correlation between miRNA expression and its putative targets (the miReact strategy), used in the miRSCAPE paper as the untrained baseline against which the XGBoost-based miRSCAPE predictor is benchmarked.

## When to use

Use when inferring miRNA activity from paired or single-sample mRNA profiles with correlation-to-putative-target scoring (miReact), or when benchmarking such target-only methods against a trained predictor like miRSCAPE.

## When NOT to use

- You need cell-level resolution, not cluster-level summaries: miReact predictions in single cells were only interpretable at the level of cell clusters
- You have no large paired miRNA-mRNA cohort and require a trained model — this target-only correlation approach is explicitly contrasted with miRSCAPE, which 'needs to be trained on a large cohort of paired miRNA-mRNA data'
- Your hypothesis requires detecting induced miRNA expression with high sensitivity — target-only scoring returned logFC of only 0.33, 0.15, -1.00, -1.13 where pooled miRSCAPE returned logFC 6.2, 6.0, 5.9, 4.9

## Inputs

- Paired bulk mRNA and bulk miRNA expression matrices (genes/miRNAs as row names, patient values as columns), e.g., TCGA cohorts
- scRNA-seq data stored as a Seurat object (e.g., Rzepiela miR-199a induced vs. uninduced data)
- Putative miRNA-target relationships used for cross-gene correlation scoring

## Outputs

- Per-miRNA activity scores / predicted miRNA expression per sample or cell cluster
- accuracy_comparison.csv (per-miRNA accuracy table for miReact vs. miRSCAPE)
- accuracy_scatter.png (per-miRNA accuracy scatter plot)
- mir199a_logfc_results.csv (induced-vs-uninduced logFC and p-values for pooled miRSCAPE, single-cell miRSCAPE, and miReact control)

## How to apply

Preprocess expression matrices so genes/miRNAs are row names and patients or cells are columns; log-transform bulk data with the bulkTransform function (justNorm=TRUE for miRNA, miRNA must not be scaled). Run miReact, which ascertains miRNA activity from the cross-gene correlation structure between each miRNA and its putative targets, on the same pooled transcriptomic profiles used for the comparator model (e.g., pooled TCGA bulk mRNA/miRNA matrices across five cancer types, or pooled/bootstrapped induced vs. uninduced single-cell profiles prepared with modifySeuratObject). Compute per-miRNA prediction accuracy or induced-vs-uninduced log-fold changes and Wilcoxon p-values, then compare against miRSCAPE, which instead trains XGBoost models (default booster, objective, maximum depth, eta, rounds, early stopping) on large paired miRNA-mRNA cohorts such as TCGA, GTEx, and CCLE. Write per-miRNA results to tables such as accuracy_comparison.csv and mir199a_logfc_results.csv and render comparison plots (e.g., accuracy_scatter.png). Expected outcome: target-only correlation scoring is substantially weaker, averaging ~0.24 top-50 accuracy versus ~0.86 for miRSCAPE, and shows no significant miR-199a induction signal (logFC 0.33, 0.15, -1.00, -1.13).

## Related tools

- **miReact** (Cross-gene-correlation-based target-only method used to infer miRNA activity; serves as the baseline/control in accuracy and induction benchmarks)
- **miRSCAPE** (XGBoost-trained comparator that predicts miRNA expression from mRNA profiles, benchmarked against the correlation-based scoring) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Handles Seurat-object scRNA-seq inputs via modifySeuratObject before profiles are pooled for scoring) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying learner for the miRSCAPE comparator trained on paired bulk miRNA-mRNA cohorts) — https://github.com/hannenhalli-lab/miRSCAPE

## Evaluation signals

- On pooled TCGA cohorts, target-only correlation scoring should yield a top-50 average accuracy of ~0.24, versus ~0.86 for miRSCAPE
- On the miR-199a induction dataset, the miReact control should produce logFC 0.33, 0.15, -1.00, -1.13 (no significant induction), while pooled miRSCAPE gives 6.2, 6.0, 5.9, 4.9 (p < E-72)
- All four induced-vs-uninduced cases (2 miRNAs x 2 cell types) run per strategy; rows must be present in mir199a_logfc_results.csv
- Per-miRNA accuracy table written to accuracy_comparison.csv and scatter plot rendered to accuracy_scatter.png
- bulkTransform applied with justNorm=TRUE on miRNA matrices (log-transform only, no scaling of miRNA data)

## Limitations

- Target-only inference is unreliable because miRNA-target relationships are highly incomplete and noisy, and miRNA induction has a variable effect on target expression
- Although miReact was applied to predict miRNA expression in single cells, its authors interpreted predicted miRNA at the level of cell clusters, limiting single-cell resolution
- Empirically, top-50 average accuracy of only ~0.24 versus ~0.86 for the trained miRSCAPE model on pooled TCGA cohorts
- Pre-trained models may fail on scRNA-seq data due to large dropout rates, potentially requiring retraining on detected genes only

## Evidence
