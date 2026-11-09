---
name: cross-tool-result-aggregation
description: Use when several annotation tools have been run on the same scRNA-seq
  train/test splits and their per-cell predictions must be harmonized, scored against
  study labels, and aggregated into a single comparison table.
when_to_use_negative:
- Only a single annotation tool has been run and no cross-tool comparison is intended
- The datasets lack reference/ground-truth annotations from the original study, so
  'consistent'/'inconsistent' labels cannot be assigned
- The goal is scWizard's own major-type annotation performance rather than cross-tool
  subtype benchmarking on the independent T/NK and myeloid datasets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scWizard
  role: Reference method whose subtype-annotation performance is compared against
    the five baselines on identical training sets
  repo: https://github.com/Dulab2020/scWizard
- name: ACTINN
  role: Baseline classifier trained on identical training sets for accuracy/F1 aggregation
- name: SingleR
  role: Baseline reference-based annotator run with default settings for the comparison
- name: SingleCellNet
  role: Baseline classifier trained on identical training sets for the comparison
    table
- name: SCSA
  role: Marker-based annotator run with Table S3 marker inputs on T/NK and myeloid
    datasets
- name: CellAssign
  role: Marker-based annotator run with Table S3 marker inputs on T/NK and myeloid
    datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:474ac0c19c29c75806fd03bd16513620a02e25b78ac8987f1301c20bd0dc3f57
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tool-result-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-tool, per-dataset annotation results from multiple cell-annotation methods into a unified accuracy/F1 comparison table, enabling head-to-head benchmarking on identical train/test splits as done for ACTINN, SingleR, SingleCellNet, SCSA and CellAssign versus scWizard.

## When to use

Use when several annotation tools have been run on the same scRNA-seq train/test splits and their per-cell predictions must be harmonized, scored against study labels, and aggregated into a single comparison table.

## When NOT to use

- Only a single annotation tool has been run and no cross-tool comparison is intended
- The datasets lack reference/ground-truth annotations from the original study, so 'consistent'/'inconsistent' labels cannot be assigned
- The goal is scWizard's own major-type annotation performance rather than cross-tool subtype benchmarking on the independent T/NK and myeloid datasets

## Inputs

- Train/test-split scRNA-seq count matrices for T/NK_BRCA, T/NK_KIRC, Mye_PAAD and LYM (Table S1)
- Per-cell predicted labels from ACTINN, SingleR, SingleCellNet, SCSA and CellAssign runs (default settings)
- Original study cell-type annotations for each dataset
- Marker gene sets: classical T/NK subtype markers and Wilcoxon differentially expressed myeloid subcluster markers (Table S3)

## Outputs

- annotation_accuracy_comparison.csv with per-tool accuracy and F1 per dataset and three-dataset averages
- Per-cluster and per-dataset accuracy/F1 values computed against total cell counts

## How to apply

Load the comparison datasets (T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM) from Table S1 and split each into training and testing sets as described in the methods. For each dataset, collect predictions from tools trained on identical training sets with default settings (scWizard, ACTINN, SingleR, SingleCellNet) and marker-based predictions from CellAssign and SCSA, using classical T/NK subtype markers (Table S3) or Wilcoxon-derived myeloid subcluster markers as marker inputs. Label each predicted cell as 'consistent' (matches the original study annotation), 'inconsistent' (unmatched), or 'unknown' (unidentifiable), then compute per-cluster and per-dataset accuracy and F1 against total cell counts. Aggregate per-dataset metrics into three-dataset averages and write the comparison to annotation_accuracy_comparison.csv, checking reproduction against reported values such as T/NK_BRCA F1 of ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118.

## Related tools

- **scWizard** (Reference method whose subtype-annotation performance is compared against the five baselines on identical training sets) — https://github.com/Dulab2020/scWizard
- **ACTINN** (Baseline classifier trained on identical training sets for accuracy/F1 aggregation)
- **SingleR** (Baseline reference-based annotator run with default settings for the comparison)
- **SingleCellNet** (Baseline classifier trained on identical training sets for the comparison table)
- **SCSA** (Marker-based annotator run with Table S3 marker inputs on T/NK and myeloid datasets)
- **CellAssign** (Marker-based annotator run with Table S3 marker inputs on T/NK and myeloid datasets)

## Examples

```
devtools::install_github("Dulab2020/scWizard")
```

## Evaluation signals

- Reproduced T/NK_BRCA F1 values match reported ones: ACTINN 0.574, SingleR 0.553, SingleCellNet 0.572, SCSA 0.297, CellAssign 0.118
- Three-dataset average accuracies approximate the reported 0.57 (ACTINN), 0.45 (SingleR), 0.5 (SingleCellNet), 0.5 (SCSA) and 0.1 (CellAssign)
- Every predicted cell carries exactly one label of 'consistent', 'inconsistent' or 'unknown' relative to the original study annotation
- All five tools were trained/run on the identical training sets per dataset with each tool's default settings
- annotation_accuracy_comparison.csv contains both per-dataset metrics and three-dataset averages for all five tools

## Limitations

- Baseline accuracies are dataset- and marker-set-dependent: marker-based tools depend on Table S3 marker quality (classical T/NK markers vs Wilcoxon-derived myeloid DE genes)
- Cells that cannot be identified are scored as 'unknown', which can depress aggregate accuracy for tools that return ambiguous labels
- The five existing tools used as comparison baselines are not named in every article section, so the full comparison scope may need supplementary tables (Table S1, S3)

## Evidence

- [results] five baseline methods achieved only low average accuracies: [evidence span withheld — anchor status: not_found]
