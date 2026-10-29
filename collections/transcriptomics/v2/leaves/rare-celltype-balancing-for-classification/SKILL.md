---
name: rare-celltype-balancing-for-classification
description: Use when preparing a cell-by-gene reference matrix (e.g., SELINA xx_expr.txt
  with paired xx_meta.txt labels) for supervised cell-type annotation and some cell
  types are far outnumbered by majority types.
when_to_use_negative:
- The query dataset is being annotated with an already-trained SELINA model — balancing
  is applied to reference data before pre-training, not to query data
- Cell-type counts are already roughly balanced relative to Nmax, so no rare types
  fall below the rarity threshold
- Working with disease-mode data where diverse abnormal-cell expression profiles are
  expected and the paper recommends users train with their own data rather than relying
  on curated references
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SELINA
  role: Implements SMOTE-based cell type balancing as the first step of its train
    pipeline (cell type balancing, pre-training and fine-tuning)
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Used with pandas/numpy to load the expression matrix and labels, count cells
    per type, and synthesize oversampled cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:584b94f1dcfce3e8d0ae7db3add48e1293a34c749deaa78af0d83a335a8c2fbe
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rare-celltype-balancing-for-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Balances a reference scRNA-seq expression matrix before supervised training by oversampling rare cell types with SMOTE, so that minority cell types are not ignored during classifier modeling (as in SELINA's cell-type balancing step).

## When to use

Use when preparing a cell-by-gene reference matrix (e.g., SELINA xx_expr.txt with paired xx_meta.txt labels) for supervised cell-type annotation and some cell types are far outnumbered by majority types.

## When NOT to use

- The query dataset is being annotated with an already-trained SELINA model — balancing is applied to reference data before pre-training, not to query data
- Cell-type counts are already roughly balanced relative to Nmax, so no rare types fall below the rarity threshold
- Working with disease-mode data where diverse abnormal-cell expression profiles are expected and the paper recommends users train with their own data rather than relying on curated references

## Inputs

- Reference cell-by-gene expression matrix (e.g., xx_expr.txt in SELINA reference format)
- Paired cell-type label/meta file (e.g., xx_meta.txt)
- Rarity threshold / target class size (100, 500, or 1000, chosen as a function of Nmax)

## Outputs

- Balanced reference expression matrix including synthetic SMOTE cells
- Updated cell-type label file covering original plus synthesized cells

## How to apply

Load the reference cell-by-gene expression matrix and paired cell-type labels with Python (pandas/numpy), and count cells per type; flag a type as rare using an Nmax-dependent threshold (100, 500, or 1000). For each rare type, repeatedly draw pairs of real cells (xa, xn) of that type and synthesize new cells as xs = xa + lambda*(xn - xa), with lambda drawn uniformly at random, until the class reaches its target magnitude or is not less than 1,000 cells, matching majority-type abundance. Concatenate the synthesized cells with the original matrix, assign them the corresponding cell-type labels, and write the balanced matrix and updated labels out. Rationale: imbalanced cell-type numbers cause minority types to be ignored during modeling, and this SMOTE balancing is the first of SELINA's three steps (cell type balancing, pre-training and fine-tuning).

## Related tools

- **SELINA** (Implements SMOTE-based cell type balancing as the first step of its train pipeline (cell type balancing, pre-training and fine-tuning)) — https://github.com/SELINA-team/SELINA.py
- **Python** (Used with pandas/numpy to load the expression matrix and labels, count cells per type, and synthesize oversampled cells)

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- Every originally rare cell type in the output contains at least its target number of cells, not exceeding the 1,000-cell cap
- Synthetic cells equal linear combinations of real same-type cells (xs = xa + lambda*(xn - xa)) and inherit the correct cell-type label
- Number of output columns/cells equals original cell count plus the number of synthesized cells
- Rare types are flagged consistently with the Nmax-dependent threshold choice (100, 500, or 1000)
- Downstream model (pre-training) no longer ignores minority classes, consistent with the Figure 3H ablation of the balancing step

## Limitations

- Oversampled rare cell types are capped at 1,000 cells or matched to majority magnitude, so synthetic cells may not capture full within-type diversity
- Despite the large-scale reference (1.7 million cells, 230 cell types), cell-type abundance in each tissue is still imbalanced
- SMOTE synthesizes cells as weighted linear combinations, which may not represent biologically realistic expression profiles for very small rare populations

## Evidence
