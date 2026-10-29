---
name: cell-type-abundance-thresholding
description: Use when building a supervised cell-type annotation model from a reference
  scRNA-seq matrix whose cell-type counts are imbalanced, and minority types risk
  being ignored during training.
when_to_use_negative:
- Input is a query dataset to be annotated — balancing is applied to the reference
  data during training, not to query cells
- Cell-type counts are already balanced or rare types exceed the 1,000-cell cap, so
  oversampling would add redundant synthetic cells
- The downstream method does not train a supervised classifier on the reference (e.g.,
  marker-based methods like Garnett or SCINA), where imbalanced abundances matter
  less
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Framework whose cell-type balancing step implements SMOTE oversampling of
    rare reference cell types before pre-training
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Used with pandas/numpy to load the reference matrix, count cells per type,
    and synthesize rare-type cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:d8996055582110b807d59e6711b1f13217f96e6b884b41103f8fc4626cfd72e5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA.py
  tools:
  - Python
  - SELINA
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cell-type-abundance-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Flags cell types in a single-cell reference expression matrix as rare based on abundance thresholds (100/500/1000 cells, chosen as a function of Nmax) and oversamples them with SMOTE so classifiers do not ignore minority cell types, as done in SELINA's cell-type balancing step.

## When to use

Use when building a supervised cell-type annotation model from a reference scRNA-seq matrix whose cell-type counts are imbalanced, and minority types risk being ignored during training.

## When NOT to use

- Input is a query dataset to be annotated — balancing is applied to the reference data during training, not to query cells
- Cell-type counts are already balanced or rare types exceed the 1,000-cell cap, so oversampling would add redundant synthetic cells
- The downstream method does not train a supervised classifier on the reference (e.g., marker-based methods like Garnett or SCINA), where imbalanced abundances matter less

## Inputs

- Reference cell-by-gene expression matrix (e.g., SELINA xx_expr.txt plain format)
- Paired cell-type label/metadata file (e.g., xx_meta.txt)

## Outputs

- Balanced expression matrix containing original plus SMOTE-synthesized cells
- Updated cell-type label file in which rare types meet their target magnitude (capped at 1,000 cells)

## How to apply

Load the reference cell-by-gene expression matrix with its paired cell-type label file (e.g., xx_expr.txt and xx_meta.txt in the SELINA reference format) using pandas/numpy, then count cells per type and flag types as rare when their abundance falls below a rarity threshold of 100, 500, or 1000 chosen as a function of Nmax (the majority-class size). For each rare type, repeatedly draw pairs of real cells (xa, xn) of that type and synthesize new cells as xs = xa + lambda*(xn - xa) with lambda drawn uniformly at random — SELINA's formulation of SMOTE — until the class reaches its target magnitude or is not less than 1,000 cells. Concatenate the synthesized cells with the original matrix, assign them the corresponding cell-type labels, and write the balanced matrix and updated labels to output files. Rationale: SELINA applies this balancing before pre-training so the MADA-based deep learning model is not dominated by majority cell types.

## Related tools

- **SELINA** (Framework whose cell-type balancing step implements SMOTE oversampling of rare reference cell types before pre-training) — https://github.com/SELINA-team/SELINA.py
- **Python** (Used with pandas/numpy to load the reference matrix, count cells per type, and synthesize rare-type cells)

## Examples

```
import pandas as pd; counts = pd.read_csv('xx_meta.txt', sep='\t')['CellType'].value_counts(); rare = counts[counts < 1000].index  # flag rare types before SMOTE oversampling
```

## Evaluation signals

- Every originally rare cell type in the output now contains at least its target number of cells, not exceeding the 1,000-cell cap
- Per-type cell counts after balancing are comparable in magnitude to the majority types (Nmax)
- Synthesized cells are linear combinations of two real cells of the same type: xs = xa + lambda*(xn - xa) with random lambda
- Cell-type labels of concatenated synthetic cells exactly match the source rare type
- Improved annotation accuracy for minority types, consistent with the Figure 3H ablation of the SMOTE balancing step

## Limitations

- Synthetic cells are linear combinations of real cells and may not capture genuine transcriptomic diversity of very rare types
- Reference still suffers from imbalances in tissue-specific datasets and cell-type abundance despite its large scale

## Evidence
