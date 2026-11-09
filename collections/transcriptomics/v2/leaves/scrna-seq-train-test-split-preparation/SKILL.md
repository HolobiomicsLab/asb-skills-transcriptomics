---
name: scrna-seq-train-test-split-preparation
description: Use when a labeled cancer scRNA-seq dataset must be partitioned into
  training and testing sets to benchmark cell-type or cell-subtype annotation tools
  on the same cells, e.g., before computing per-cluster accuracy/F1 against original
  study annotations.
when_to_use_negative:
- The dataset lacks original study cell annotations, so no 'consistent'/'inconsistent'/'unknown'
  labels or test-set ground truth can be defined
- The target cell types are tissue-specific (e.g., alveolar cells in lung cancers)
  outside scWizard's TME reference of 7 major types and 47 subtypes
- Only the scWizard built-in reference (113,976 cells across 13 cancer types) is needed
  for inference without any benchmarking or retraining
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Hierarchical deep-neural-network annotation model trained on the identical
    training sets during benchmarking; accepts custom H5 trainset
  repo: https://github.com/Dulab2020/scWizard
- name: ACTINN
  role: Baseline annotator trained on the same training splits with default settings
- name: SingleR
  role: Baseline annotator trained on the same training splits with default settings
- name: SingleCellNet
  role: Baseline annotator trained on the same training splits with default settings
- name: SCSA
  role: Marker-based baseline run on T/NK and myeloid test sets using Table S3 markers
- name: CellAssign
  role: Marker-based baseline run on T/NK and myeloid test sets using Table S3 markers
- name: Seurat
  role: Standard scRNA-seq pipeline (dimension reduction, clustering, DE detection)
    used within scWizard for data preparation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:a484858606eeb961327b53eadf1cad535ea3641972df8898ce9d3072522ebd4e
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# scRNA-seq-train-test-split-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare labeled cancer scRNA-seq datasets (e.g., T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM from Table S1) by splitting each into training and testing sets so that cell-annotation models such as scWizard's hierarchical deep neural network and the ACTINN/SingleR/SingleCellNet/SCSA/CellAssign baselines can be trained and evaluated on identical data partitions.

## When to use

Use when a labeled cancer scRNA-seq dataset must be partitioned into training and testing sets to benchmark cell-type or cell-subtype annotation tools on the same cells, e.g., before computing per-cluster accuracy/F1 against original study annotations.

## When NOT to use

- The dataset lacks original study cell annotations, so no 'consistent'/'inconsistent'/'unknown' labels or test-set ground truth can be defined
- The target cell types are tissue-specific (e.g., alveolar cells in lung cancers) outside scWizard's TME reference of 7 major types and 47 subtypes
- Only the scWizard built-in reference (113,976 cells across 13 cancer types) is needed for inference without any benchmarking or retraining

## Inputs

- Labeled cancer scRNA-seq count matrices for T/NK_BRCA, T/NK_KIRC, Mye_PAAD, and LYM (Table S1)
- Original study cell-type/subtype annotations for each dataset
- Classical T/NK subtype marker table (Table S3)
- Wilcoxon differentially expressed genes per myeloid subcluster (Table S3)
- Custom scWizard trainset in H5 format (rows = genes, columns = cells) with matching label file

## Outputs

- Per-dataset training and testing splits used identically across scWizard and the four learning-based baselines
- Per-cluster and per-dataset accuracy/F1 metrics against total cell counts
- Aggregated three-dataset averages for the five baselines written to annotation_accuracy_comparison.csv

## How to apply

Load the comparison datasets (T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM) from Table S1 and split each into training and testing sets as described in the methods; for model validation scWizard used a tenfold scheme, dividing the dataset into ten parts with nine used as training data and one as test in turn. Train scWizard, ACTINN, SingleR, and SingleCellNet on the identical training sets with each tool's default settings; for custom trainsets in scWizard, training data must be in H5 format with rows as genes and columns as cells, downloaded and copied to the 'app/www/python' folder under the scWizard installation directory. Run the marker-based tools CellAssign and SCSA on the T/NK datasets using classical T/NK subtype markers from Table S3 and on the myeloid dataset using per-subcluster differentially expressed genes (Wilcoxon, Table S3). Label each predicted cell 'consistent', 'inconsistent', or 'unknown' relative to the original study annotation and compute per-cluster and per-dataset accuracy/F1 against total cell counts.

## Related tools

- **scWizard** (Hierarchical deep-neural-network annotation model trained on the identical training sets during benchmarking; accepts custom H5 trainset) — https://github.com/Dulab2020/scWizard
- **ACTINN** (Baseline annotator trained on the same training splits with default settings)
- **SingleR** (Baseline annotator trained on the same training splits with default settings)
- **SingleCellNet** (Baseline annotator trained on the same training splits with default settings)
- **SCSA** (Marker-based baseline run on T/NK and myeloid test sets using Table S3 markers)
- **CellAssign** (Marker-based baseline run on T/NK and myeloid test sets using Table S3 markers)
- **Seurat** (Standard scRNA-seq pipeline (dimension reduction, clustering, DE detection) used within scWizard for data preparation)

## Examples

```
write.csv(avg_metrics, "annotation_accuracy_comparison.csv")
```

## Evaluation signals

- Reproduced T/NK_BRCA F1 values match the report: ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118
- Three-dataset average accuracies for the baselines approximate 0.57 (ACTINN), 0.45 (SingleR), 0.50 (SingleCellNet), 0.50 (SCSA), and 0.10 (CellAssign)
- Every predicted cell is labeled as exactly one of 'consistent', 'inconsistent', or 'unknown' with no missing assignments
- Custom scWizard training data validate as H5 with rows representing genes and columns representing cells before training
- All four learning-based tools were trained on identical training sets per dataset, so per-dataset metrics are directly comparable

## Limitations

- Tissue-specific cell types, such as alveolar cells in lung cancers, may not be identified and land in the Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future scWizard updates
- Marker-based baselines (CellAssign, SCSA) depend on the supplied marker tables (Table S3), so marker choice directly shapes their split-level performance

## Evidence
