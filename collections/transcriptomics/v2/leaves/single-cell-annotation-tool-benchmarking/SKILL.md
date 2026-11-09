---
name: single-cell-annotation-tool-benchmarking
description: Use when evaluating how accurately established cell-annotation tools
  (e.g. ACTINN, SingleR, SCSA) classify tumor T/NK or myeloid scRNA-seq cell subtypes
  relative to scWizard on independent datasets.
when_to_use_negative:
- The task is annotating tissue-specific cell types (e.g. alveolar cells in lung cancers)
  outside the TME — scWizard may name them as an Unknown cluster
- The goal is major (not subtype) cell-type annotation, where scWizard already reaches
  0.98 accuracy and baseline comparison adds little
- Input data lacks any reference annotation or marker gene sets (Table S3), making
  consistent/inconsistent labeling and marker-based SCSA/CellAssign runs impossible
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Deep-neural-network hierarchical annotation model being benchmarked; trains
    on identical training sets and annotates 7 major cell types and 47 TME subtypes
  repo: https://github.com/Dulab2020/scWizard
- name: ACTINN
  role: Baseline classifier trained on identical training sets with default settings
- name: SingleR
  role: Baseline reference-based annotation tool run with default settings on identical
    training sets
- name: SingleCellNet
  role: Baseline classifier run with default settings on identical training sets
- name: SCSA
  role: Marker-based baseline run on T/NK datasets using classical T/NK subtype markers
    and on myeloid data using Wilcoxon DE genes (Table S3)
- name: CellAssign
  role: Marker-based baseline run on T/NK datasets using classical T/NK subtype markers
    and on myeloid data using Wilcoxon DE genes (Table S3)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:a123485626f33f774bf85b06db3edaa957c29a75d591c9cc4e7c2a6473d1b52c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - ACTINN
  - CellAssign
  - SCSA
  - SingleCellNet
  - SingleR
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-annotation-tool-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the scWizard paper's benchmark of five widely used scRNA-seq cell-annotation baselines (ACTINN, SingleR, SingleCellNet, SCSA, CellAssign) against the deep-neural-network-based scWizard on independent myeloid (Mye_PAAD) and T/NK (T/NK_BRCA, T/NK_KIRC) datasets, quantifying accuracy and F1 per tool.

## When to use

Use when evaluating how accurately established cell-annotation tools (e.g. ACTINN, SingleR, SCSA) classify tumor T/NK or myeloid scRNA-seq cell subtypes relative to scWizard on independent datasets.

## When NOT to use

- The task is annotating tissue-specific cell types (e.g. alveolar cells in lung cancers) outside the TME — scWizard may name them as an Unknown cluster
- The goal is major (not subtype) cell-type annotation, where scWizard already reaches 0.98 accuracy and baseline comparison adds little
- Input data lacks any reference annotation or marker gene sets (Table S3), making consistent/inconsistent labeling and marker-based SCSA/CellAssign runs impossible

## Inputs

- Independent scRNA-seq test datasets from Table S1: T/NK_BRCA, T/NK_KIRC, Mye_PAAD (plus LYM)
- Training/testing set splits per the scWizard methods
- Marker gene sets from Table S3: classical T/NK subtype markers (for CellAssign/SCSA) and Wilcoxon DE genes per myeloid subcluster
- Original study cell annotations per dataset

## Outputs

- Per-cluster and per-dataset accuracy and F1 values per tool
- Aggregated comparison table: annotation_accuracy_comparison.csv
- Per-cell label assignments: consistent / inconsistent / unknown

## How to apply

Load the four comparison datasets (T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM) from Table S1 and split each into training and testing sets per the scWizard methods. Train scWizard, ACTINN, SingleR and SingleCellNet on identical training sets using each tool's default settings; run the marker-based tools CellAssign and SCSA on the T/NK datasets using classical T/NK subtype markers (Table S3) and on the myeloid dataset using Wilcoxon differentially expressed genes of each myeloid subcluster (Table S3). Label each predicted cell 'consistent' if it matches the original study annotation, 'inconsistent' if unmatched, or 'unknown' if unidentifiable, then compute per-cluster and per-dataset accuracy/F1 against total cell counts. Aggregate per-dataset metrics and three-dataset averages for the five baselines and write annotation_accuracy_comparison.csv, checking the reproduced values against reported baselines (e.g. T/NK_BRCA F1: ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118; three-dataset averages ~0.57/0.45/0.50/0.50/0.10).

## Related tools

- **scWizard** (Deep-neural-network hierarchical annotation model being benchmarked; trains on identical training sets and annotates 7 major cell types and 47 TME subtypes) — https://github.com/Dulab2020/scWizard
- **ACTINN** (Baseline classifier trained on identical training sets with default settings)
- **SingleR** (Baseline reference-based annotation tool run with default settings on identical training sets)
- **SingleCellNet** (Baseline classifier run with default settings on identical training sets)
- **SCSA** (Marker-based baseline run on T/NK datasets using classical T/NK subtype markers and on myeloid data using Wilcoxon DE genes (Table S3))
- **CellAssign** (Marker-based baseline run on T/NK datasets using classical T/NK subtype markers and on myeloid data using Wilcoxon DE genes (Table S3))

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard")
```

## Evaluation signals

- Reproduced T/NK_BRCA F1 values match the report: ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118
- Three-dataset average accuracies for the five baselines approximate 0.57 (ACTINN), 0.45 (SingleR), 0.50 (SingleCellNet), 0.50 (SCSA), 0.10 (CellAssign)
- Every cell receives exactly one of the labels 'consistent', 'inconsistent', or 'unknown' relative to the original study annotation
- Per-dataset accuracy/F1 are computed against total cell counts and aggregated per tool across all three independent datasets
- The output file annotation_accuracy_comparison.csv contains per-dataset and three-dataset average metrics for all five tools

## Limitations

- Tissue-specific cell types (e.g. alveolar cells in lung cancers) may not be identified and are named in an Unknown cluster; tissue-specific subtypes and cellular states are deferred to future updates
- Marker-based baselines (SCSA, CellAssign) depend entirely on the supplied marker sets: classical T/NK markers vs Wilcoxon-derived myeloid DE genes; CellAssign dropped to ~0.1 average accuracy
- The five existing comparison tools are not named in the discussion section, so cross-referencing the methods/supplementary (Tables S1, S3) is required
