---
name: supplementary-table-data-compilation
description: Use when a reproduction task requires assembling per-method performance
  metrics (F1, accuracy, G-mean, Cohen's Kappa, MCC) and rare-identification success
  counts from multiple method outputs into a single benchmark table.
when_to_use_negative:
- The goal is to re-run the scCAD algorithm or comparator methods themselves — this
  skill only aggregates already-computed per-type predictions and metrics, not execution
  of the tools.
- Only one method on one dataset is available — the skill's value is the cross-method,
  cross-dataset compilation matching the reported 25-dataset benchmark table.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: Top-ranked method whose F1 (0.4172) and 20/25 success count anchor the compiled
    benchmark table
  repo: github.com/xuyp-csu/scCAD
- name: CellSIUS
  role: Comparator method; per-dataset rare predictions contribute F1 (0.2812) and
    success count (16) rows to the table
  repo: github.com/Novartis/CellSIUS
- name: SCA
  role: Comparator method (shannonca); contributes second-ranked F1 (0.3359) and success
    count (18)
  repo: github.com/bendemeo/shannonca
- name: EDGE
  role: Comparator method whose rare-call counts (16 datasets) are tabulated alongside
    scCAD and CellSIUS
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:aed3d5e473ef85fed860603cad390103a44ed7c594fe75648922962fe171e90e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - CIARA
  - CellSIUS
  - EDGE
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - SCA
  - SCISSORS
  - Seurat
  - scCAD
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# supplementary-table-data-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles per-method F1 scores and rare-type success counts across the 25 real scRNA-seq benchmark datasets of the scCAD study into a summary table (benchmark_f1_scores.csv), reproducing the paper's reported ranking.

## When to use

Use when a reproduction task requires assembling per-method performance metrics (F1, accuracy, G-mean, Cohen's Kappa, MCC) and rare-identification success counts from multiple method outputs into a single benchmark table.

## When NOT to use

- The goal is to re-run the scCAD algorithm or comparator methods themselves — this skill only aggregates already-computed per-type predictions and metrics, not execution of the tools.
- Only one method on one dataset is available — the skill's value is the cross-method, cross-dataset compilation matching the reported 25-dataset benchmark table.

## Inputs

- Per-dataset, per-method predicted rare cell sets across 25 real scRNA-seq benchmark datasets (Supplementary Table 1 list)
- Original study rare cell type annotations for each benchmark dataset
- Per-rare-type metric values (F1, accuracy, G-mean, Cohen's Kappa, MCC) computed for each method
- Method-level rare-identification success flags derived from the 30% recovery criterion

## Outputs

- benchmark_f1_scores.csv — summary table of F1 scores per method across 25 datasets
- Success-count table listing datasets with at least one successfully identified rare type per method (e.g. scCAD 20, SCA 18, CellSIUS 16, EDGE 16, GapClust 12)

## How to apply

For each of the 25 real scRNA-seq datasets, collect each method's predicted rare cell sets (from scCAD, SCA, CellSIUS, EDGE, GapClust, FiRE, GiniClust2/3, RaceID3, SCISSORS, CIARA) and compute the F1 score per rare cell type against the original study annotations, alongside accuracy, G-mean, Cohen's Kappa, and MCC. Apply the 30% recovery criterion — a method successfully identifies rare type t when |S_pre ∩ S_t| / |S_t| > 30% — to count datasets with at least one successfully identified rare type per method. Compile the F1 scores and success counts into a summary CSV, expecting the reported ranking: scCAD F1 = 0.4172, SCA 0.3359, CellSIUS 0.2812; success counts scCAD 20, SCA 18, CellSIUS 16, EDGE 16, GapClust 12 out of 25 datasets. Validate that the compiled table matches the reported ordering, with scCAD achieving the overall highest F1 and successful rare-type identification in 20 of the 25 datasets.

## Related tools

- **scCAD** (Top-ranked method whose F1 (0.4172) and 20/25 success count anchor the compiled benchmark table) — github.com/xuyp-csu/scCAD
- **CellSIUS** (Comparator method; per-dataset rare predictions contribute F1 (0.2812) and success count (16) rows to the table) — github.com/Novartis/CellSIUS
- **SCA** (Comparator method (shannonca); contributes second-ranked F1 (0.3359) and success count (18)) — github.com/bendemeo/shannonca
- **EDGE** (Comparator method whose rare-call counts (16 datasets) are tabulated alongside scCAD and CellSIUS) — github.com/shawnstat/EDGE

## Examples

```
python -c "import pandas as pd; pd.DataFrame({'method':['scCAD','SCA','CellSIUS'],'F1':[0.4172,0.3359,0.2812],'success_count':[20,18,16]}).to_csv('benchmark_f1_scores.csv', index=False)"
```

## Evaluation signals

- Compiled F1 ranking matches reported values: scCAD 0.4172 > SCA 0.3359 > CellSIUS 0.2812 as the top three.
- Success counts match reported totals: scCAD 20, SCA 18, CellSIUS 16, EDGE 16, GapClust 12 of 25 datasets.
- Every success flag derives from the 30% recovery criterion: |S_pre ∩ S_t| / |S_t| > 30% per rare cell type.
- Output file benchmark_f1_scores.csv contains one F1 entry per method-dataset pair plus aggregated success counts, with scCAD achieving the overall highest F1.
- Per-type metrics cover the full reported set (accuracy, G-mean, Cohen's Kappa, MCC) in addition to F1.

## Limitations

- Source data availability/accession IDs for the 25 benchmark datasets are not given in the extracted section, so compilation depends on external dataset retrieval.
- Exact version numbers for most comparator packages (CellSIUS, CIARA, EDGE, FiRE, GapClust, GiniClust2/3, RaceID3, SCISSORS) are not stated, so reproduced numbers may vary with package versions.
- Counts depend on the chosen size threshold for calling rare global clusterings (1% or 5%) when comparator methods are scored as global clusterings.

## Evidence

- [methods] Combine each method's predicted rare cell sets per dataset and compute the F1 score per rare cell type against the original study annotations: [evidence span withheld — anchor status: not_found]
- [methods] 30% recovery criterion: [evidence span withheld — anchor status: not_found]
- [results] reported ranking: [evidence span withheld — anchor status: not_found]
- [discussion] independence threshold: [evidence span withheld — anchor status: not_found]
