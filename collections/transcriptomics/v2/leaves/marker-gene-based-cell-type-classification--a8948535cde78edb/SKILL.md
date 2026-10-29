---
name: marker-gene-based-cell-type-classification
description: Use when a scRNA-seq dataset (e.g., T/NK cells from BRCA/KIRC or myeloid
  cells from PAAD) needs cell-type/subtype annotation via a canonical marker-gene
  set, or when marker-based tools must be benchmarked against a learned classifier.
when_to_use_negative:
- Tissue-specific cell types without canonical marker gene sets (e.g., alveolar cells
  in lung cancers), which marker-based tools may fail to identify and leave in an
  Unknown cluster
- When the goal is to leverage hierarchical major-type-to-subtype relationships learned
  from a large reference — a trained classifier like scWizard (113,976-cell reference)
  is the appropriate choice
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: CellAssign
  role: Marker-based annotator run on the classical T/NK subtype markers (Table S3)
    and myeloid DE genes to produce subtype predictions for the baseline benchmark
- name: SCSA
  role: Marker-based annotator run alongside CellAssign on the same Table S3 marker
    inputs (T/NK classical markers; myeloid Wilcoxon DE genes) as the second marker-based
    baseline
- name: Seurat
  role: Used to define the standardized pipeline (dimension reduction, clustering,
    differential gene expression detection) from which myeloid subcluster marker genes
    are derived
- name: scWizard
  role: Deep-learning hierarchical annotation tool against which the marker-based
    classification results are benchmarked on identical training/test splits
  repo: https://github.com/Dulab2020/scWizard
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:ca24845aebf5a579d37aba83c750b633d2b34ea2bdeb1f8fe267bc328fbc6ffa
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
  - Seurat
  - SingleCellNet
  - SingleR
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# marker-gene-based-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies scRNA-seq cells by matching a marker-gene panel (e.g., classical T/NK subtype markers or Wilcoxon-derived myeloid DE genes from Table S3) against the expression matrix using tools such as CellAssign and SCSA, and scores the predictions as accuracy/F1 baselines against deep-learning annotation methods like scWizard.

## When to use

Use when a scRNA-seq dataset (e.g., T/NK cells from BRCA/KIRC or myeloid cells from PAAD) needs cell-type/subtype annotation via a canonical marker-gene set, or when marker-based tools must be benchmarked against a learned classifier.

## When NOT to use

- Tissue-specific cell types without canonical marker gene sets (e.g., alveolar cells in lung cancers), which marker-based tools may fail to identify and leave in an Unknown cluster
- When the goal is to leverage hierarchical major-type-to-subtype relationships learned from a large reference — a trained classifier like scWizard (113,976-cell reference) is the appropriate choice

## Inputs

- scRNA-seq count matrices with original study annotations (e.g., T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM datasets from Table S1)
- Marker-gene sets: classical T/NK subtype markers (Table S3) or differentially expressed genes per myeloid subcluster (Wilcoxon, Table S3)
- Training/testing split definitions from Table S1

## Outputs

- Per-cell annotation labels classified as consistent, inconsistent, or unknown
- Per-cluster and per-dataset accuracy and F1 scores computed against total cell counts
- Aggregated comparison table written to annotation_accuracy_comparison.csv

## How to apply

First define the marker input: for T/NK datasets use the classical T/NK subtype markers from Table S3; for the myeloid dataset derive markers as differentially expressed genes of each myeloid subcluster using a Wilcoxon test in Seurat (also recorded in Table S3). Run the marker-based tools CellAssign and SCSA on these marker panels with default settings, then label each predicted cell as 'consistent' (matches the original study annotation), 'inconsistent' (unmatched), or 'unknown' (unidentifiable). Compute per-cluster and per-dataset accuracy and F1 against total cell counts, and aggregate three-dataset averages to benchmark against trained tools (scWizard, ACTINN, SingleR, SingleCellNet) applied to the identical training/test splits from Table S1.

## Related tools

- **CellAssign** (Marker-based annotator run on the classical T/NK subtype markers (Table S3) and myeloid DE genes to produce subtype predictions for the baseline benchmark)
- **SCSA** (Marker-based annotator run alongside CellAssign on the same Table S3 marker inputs (T/NK classical markers; myeloid Wilcoxon DE genes) as the second marker-based baseline)
- **Seurat** (Used to define the standardized pipeline (dimension reduction, clustering, differential gene expression detection) from which myeloid subcluster marker genes are derived)
- **scWizard** (Deep-learning hierarchical annotation tool against which the marker-based classification results are benchmarked on identical training/test splits) — https://github.com/Dulab2020/scWizard

## Evaluation signals

- T/NK_BRCA F1 values should approximate the reported benchmark: ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118
- Three-dataset average accuracies for the five baselines should be approximately 0.57 (ACTINN), 0.45 (SingleR), 0.50 (SingleCellNet), 0.50 (SCSA), and 0.10 (CellAssign)
- Every predicted cell must be assigned one of exactly three labels: consistent, inconsistent, or unknown
- The comparison table annotation_accuracy_comparison.csv must contain per-dataset metrics plus the three-dataset averages for all five tools
- Myeloid markers must originate from Wilcoxon differential expression of each myeloid subcluster (Table S3), not arbitrary gene lists

## Limitations

- Marker-based baselines performed poorly on these datasets — CellAssign achieved an average accuracy of only 0.1 and SCSA 0.5 on the three independent datasets
- Existing annotation tools struggle with lack of canonical marker gene sets, poorly matched references, and overlooking hierarchical relationships between cell types
- Tissue-specific subtypes without markers (e.g., alveolar cells in lung cancers) may not be identified and land in the Unknown cluster

## Evidence
