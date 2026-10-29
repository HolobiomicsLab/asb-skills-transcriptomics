---
name: smote-minority-cell-synthesis
description: Use when preparing a reference cell-by-gene expression matrix (e.g.,
  SELINA xx_expr.txt + xx_meta.txt) for classifier training where some cell types
  are far less abundant than the majority and would otherwise be ignored during modeling.
when_to_use_negative:
- The query data being preprocessed for prediction (selina preprocess), which is not
  part of reference training and should not be oversampled
- The reference cell-type abundance is already balanced, so no types fall below the
  rarity threshold
- Disease-mode prediction where the article recommends users train on their own disease
  data because disease expression profiles are extremely diverse across patients
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Annotation framework whose cell-type balancing step performs the SMOTE oversampling
    before pre-training
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Language used to load the reference expression matrix and labels and implement
    the pairwise weighted-vector synthesis
- name: imbalanced-learn
  role: Conda dependency of SELINA (v0.8.1) relevant to synthetic minority oversampling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:818e0c5e8c9296eaf603c26aad4db33afb54e134da3d00343bfe8b535913b9b1
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
  - imbalanced-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# smote-minority-cell-synthesis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SMOTE-based oversampling of rare cell types in a single-cell reference expression matrix, as used in SELINA's cell-type balancing step so that classifiers do not ignore minority cell types during pre-training.

## When to use

Use when preparing a reference cell-by-gene expression matrix (e.g., SELINA xx_expr.txt + xx_meta.txt) for classifier training where some cell types are far less abundant than the majority and would otherwise be ignored during modeling.

## When NOT to use

- The query data being preprocessed for prediction (selina preprocess), which is not part of reference training and should not be oversampled
- The reference cell-type abundance is already balanced, so no types fall below the rarity threshold
- Disease-mode prediction where the article recommends users train on their own disease data because disease expression profiles are extremely diverse across patients

## Inputs

- Reference cell-by-gene expression matrix (plain text, SELINA reference format, e.g., xx_expr.txt)
- Paired cell-type label file for the reference (e.g., xx_meta.txt)

## Outputs

- Balanced cell-by-gene expression matrix containing original plus synthesized cells
- Updated cell-type label file in which every originally rare type contains at least its target number of cells (capped at 1,000)

## How to apply

Load the reference cell-by-gene matrix and paired cell-type labels (SELINA reference format xx_expr.txt/xx_meta.txt) with Python (pandas/numpy), and count cells per type, flagging a type as rare when its abundance falls below the rarity threshold (100, 500, or 1000, chosen as a function of Nmax). For each rare type, repeatedly draw pairs of real cells (xa, xn) of that type and synthesize a new cell as xs = xa + lambda*(xn - xa) with lambda drawn uniformly at random, continuing until the class reaches its target magnitude or is not less than 1,000 cells. Concatenate the synthesized cells with the original matrix, assign them the corresponding cell-type labels, and write the balanced matrix and updated labels to output files. The rationale is that imbalanced cell numbers cause the minority cell types to be ignored in the modeling process, so balancing is done before SELINA's MADA pre-training.

## Related tools

- **SELINA** (Annotation framework whose cell-type balancing step performs the SMOTE oversampling before pre-training) — https://github.com/SELINA-team/SELINA.py
- **Python** (Language used to load the reference expression matrix and labels and implement the pairwise weighted-vector synthesis)
- **imbalanced-learn** (Conda dependency of SELINA (v0.8.1) relevant to synthetic minority oversampling)

## Examples

```
import numpy as np; lam = np.random.uniform(); xs = xa + lam * (xn - xa)  # SMOTE synthesis of a rare-type cell from a randomly drawn pair (xa, xn)
```

## Evaluation signals

- Every originally rare cell type in the output matrix contains at least its target number of cells and none exceeds the 1,000-cell cap
- Per-type cell counts after balancing are of the same magnitude as the majority types (or >= 1,000 cells), matching the stopping rule
- Each synthesized cell is a convex combination of two real cells of the same type, so its per-gene values lie within the range spanned by its parents
- Labels of synthesized cells exactly match the rare type of the parent pair, and the total label count equals the total cell count
- Downstream SELINA annotation of minority types should not degrade relative to the Figure 3H ablation, where the balancing step was evaluated

## Limitations

- Synthesized cells are linear combinations of only two real cells of the same type, so they may not capture the full expression diversity of the rare type
- The rarity threshold and target magnitude depend on Nmax (thresholds of 100, 500, or 1000), so results vary with the majority class size
- The paper notes that cell-type abundance in each tissue is still imbalanced despite the large-scale reference, so balancing cannot fully compensate for under-represented types

## Evidence
