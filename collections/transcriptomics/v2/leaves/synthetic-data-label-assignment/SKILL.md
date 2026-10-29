---
name: synthetic-data-label-assignment
description: Use when preparing a reference cell-by-gene matrix (e.g., xx_expr.txt
  with xx_meta.txt labels) for classifier training and some cell types are too rare
  relative to Nmax, causing models to ignore minority classes.
when_to_use_negative:
- Query-data preprocessing or prediction — SMOTE balancing applies only to the reference
  training data before pre-training, not to the query being annotated
- The input is already class-balanced or no cell type falls below the rarity threshold
  (100/500/1000 relative to Nmax)
- Disease-mode prediction where the paper removed fine-tuning and disease expression
  profiles deviate substantially — balancing synthetic cells may not reflect abnormal-cell
  expression profiles
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Framework whose first algorithm step ('cell type balancing') performs SMOTE
    oversampling of rare cell types before MADA pre-training
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Implementation environment (pandas/numpy) for loading the reference matrix
    and synthesizing cells via weighted linear combination
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:5a7cadbb04acb58d50778f0f77cc155f42cc3b7a17da74851e723c1a0ae1e0bb
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
yes_rate: 0.625
n_claims: 8
---

# synthetic-data-label-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SMOTE-based oversampling of rare cell types in a reference scRNA-seq expression matrix, as used in SELINA's cell-type balancing step, so that minority cell types are not ignored during downstream classifier training.

## When to use

Use when preparing a reference cell-by-gene matrix (e.g., xx_expr.txt with xx_meta.txt labels) for classifier training and some cell types are too rare relative to Nmax, causing models to ignore minority classes.

## When NOT to use

- Query-data preprocessing or prediction — SMOTE balancing applies only to the reference training data before pre-training, not to the query being annotated
- The input is already class-balanced or no cell type falls below the rarity threshold (100/500/1000 relative to Nmax)
- Disease-mode prediction where the paper removed fine-tuning and disease expression profiles deviate substantially — balancing synthetic cells may not reflect abnormal-cell expression profiles

## Inputs

- Reference cell-by-gene expression matrix in SELINA plain format (xx_expr.txt)
- Cell-type label / metadata file paired to the expression matrix (xx_meta.txt)
- Rarity threshold and target class magnitude (Nmax-dependent: 100, 500, or 1000; 1,000-cell cap)

## Outputs

- Balanced expression matrix containing original plus synthesized cells
- Updated cell-type label file in which every rare type has at least its target number of cells

## How to apply

Load the reference cell-by-gene expression matrix and paired cell-type label file (SELINA reference format, e.g., xx_expr.txt and xx_meta.txt) with Python (pandas/numpy). Count cells per type and flag rare types below a rarity threshold (100, 500, or 1000, chosen as a function of Nmax). For each rare type, repeatedly draw pairs of real cells (xa, xn) of that type and synthesize a new cell as xs = xa + lambda*(xn - xa) with lambda drawn uniformly at random, until the class reaches its target magnitude or is not less than 1,000 cells. Concatenate the synthesized cells with the original matrix, assign them the corresponding cell-type labels, and write the balanced expression matrix and updated labels to output files.

## Related tools

- **SELINA** (Framework whose first algorithm step ('cell type balancing') performs SMOTE oversampling of rare cell types before MADA pre-training) — https://github.com/SELINA-team/SELINA.py
- **Python** (Implementation environment (pandas/numpy) for loading the reference matrix and synthesizing cells via weighted linear combination)

## Examples

```
selina train --path-in ./reference/normal_data --path-out ./res/
```

## Evaluation signals

- Every originally rare cell type in the output contains at least its target number of cells, not exceeding the 1,000-cell cap
- Synthesized cells equal weighted sums of two real cells of the same type (xs = xa + lambda*(xn - xa)), so gene values stay within the convex combination range of real expression
- Output label file count per cell type matches the balanced expression matrix row count
- Balanced reference improves minority-type annotation accuracy, consistent with the Figure 3H ablation of the SMOTE step

## Limitations

- Synthesized cells are linear combinations of real cells and may not capture expression variability beyond the observed rare-type manifold
- The reference still suffers from imbalances in tissue-specific datasets and cell-type abundance despite its large scale
- Rarity threshold and target magnitude depend on Nmax, and balancing stops at a 1,000-cell cap, so very large majority classes remain comparatively larger

## Evidence

- [methods] SELINA synthesizes new cells for rare cell types by taking a pair of randomly selected rare-type cells, multiplying each gene expression vector by a random weight, and summing the weighted vectors: [evidence span withheld — anchor status: not_found]
