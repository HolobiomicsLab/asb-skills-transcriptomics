---
name: malignancy-probability-interpretation
description: Use when you have staged scRNA-seq expression matrices (.h5ad) from a
  cancerous epithelial lineage and want to test whether predicted malignancy increases
  with progression stage, rather than performing a one-off malignant/normal classification.
when_to_use_negative:
- The input is a cancer type with unknown or controversial cellular origin (e.g.,
  synovial sarcoma or certain brain tumors), where the classifier's capabilities are
  untested
- The dataset lacks prerequisite basic cell-type annotation or you need to classify
  non-epithelial lineages, since the method identifies malignant cells only from cancerous
  lineages
- The goal is de novo malignant-cell discovery without any stage/condition contrast
  — a single binary classification run suffices and per-stage trend interpretation
  adds no value
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Supervised classifier producing the malignancy_probability and Normal/Malignant
    predictions that are aggregated per carcinogenesis stage
  repo: github.com/Jonyyqn/scMalignantFinder
- name: scATOMIC
  role: Optionally integrated upstream for pan-cancer cell type annotation before
    malignancy probability interpretation
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:636384f82025c24b1d76de92345ac7c497f1772a6cf159b8a43dc96aa8d0c429
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - scATOMIC
  - scMalignantFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# malignancy-probability-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets scMalignantFinder's malignancy_probability output and Normal/Malignant predictions to quantify how the malignant fraction of epithelial cells changes across carcinogenesis stages (e.g., normal mucosa → CAG/IM → gastric cancer, or normal mucosa → polyp → colorectal tumor), using probability as a dynamic readout of tumor progression.

## When to use

Use when you have staged scRNA-seq expression matrices (.h5ad) from a cancerous epithelial lineage and want to test whether predicted malignancy increases with progression stage, rather than performing a one-off malignant/normal classification.

## When NOT to use

- The input is a cancer type with unknown or controversial cellular origin (e.g., synovial sarcoma or certain brain tumors), where the classifier's capabilities are untested
- The dataset lacks prerequisite basic cell-type annotation or you need to classify non-epithelial lineages, since the method identifies malignant cells only from cancerous lineages
- The goal is de novo malignant-cell discovery without any stage/condition contrast — a single binary classification run suffices and per-stage trend interpretation adds no value

## Inputs

- Progression-stage scRNA-seq expression matrices (.h5ad), subset to the epithelial lineage
- Pretrained classifier model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140

## Outputs

- result_adata with obs columns scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability
- Per-stage summary table (progression_summary.csv) with percentage of Malignant-labeled cells and mean malignancy probability per stage

## How to apply

Subset each stage's expression matrix (.h5ad) to the epithelial lineage, since scMalignantFinder is designed to separate malignant cells from their originating normal cells within a cancerous lineage, then run inference with classifier.scMalignantFinder using pretrain_dir pointing at the pretrained model.joblib and ordered_feature.tsv from Zenodo record 17888140, norm_type=True (sc.pp.normalize_total with target_sum=1e4), and n_thread=1. Call model.load() and model.predict() to obtain result_adata, then extract the scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns from result_adata.obs. Group cells by tissue and progression stage, compute the percentage of Malignant-labeled cells and the mean malignancy probability per stage, and evaluate whether the ordering is monotonic — nearly 0% malignant in normal mucosa/NAG, elevated in polyps or IM relative to CAG, and peaking in tumors/EGC. Write the per-stage summary to progression_summary.csv. The rationale is that the graded probability, not just the binary label, captures intermediate states during colorectal and gastric carcinogenesis.

## Related tools

- **scMalignantFinder** (Supervised classifier producing the malignancy_probability and Normal/Malignant predictions that are aggregated per carcinogenesis stage) — github.com/Jonyyqn/scMalignantFinder
- **scATOMIC** (Optionally integrated upstream for pan-cancer cell type annotation before malignancy probability interpretation) — github.com/abelson-lab/scATOMIC

## Examples

```
result_adata = classifier.scMalignantFinder(test_input, pretrain_dir='pretrained_model/', norm_type=True, n_thread=1); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Normal mucosa/NAG samples yield nearly 0% of epithelial cells labeled Malignant by scMalignantFinder
- Malignant percentage increases monotonically across colorectal stages (normal mucosa < polyp < tumor) and gastric stages (NAG < CAG < IM < EGC)
- Mean malignancy probability per stage orders with IM exceeding CAG in gastric progression
- result_adata.obs contains both scMalignantFinder_prediction and malignancy_probability columns with probabilities in [0, 1]
- Per-stage summary table matches the schema of progression_summary.csv with one row per tissue × stage

## Limitations

- scMalignantFinder requires prerequisite identification of basic cell types and is designed to identify malignant cells only from cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Molecular mechanisms underlying the intermediate states in gastric and colorectal carcinogenesis (e.g., CAG vs IM differences) are not yet resolved and remain a stated future aim

## Evidence
