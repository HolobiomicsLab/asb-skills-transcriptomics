---
name: per-class-f1-benchmark-evaluation
description: Use when benchmarking an annotation or classification model whose improvement
  is concentrated in minority classes, and aggregate metrics would hide the rare-class
  gains.
when_to_use_negative:
- When the goal is overall annotation accuracy on balanced cell types — macro aggregates
  or accuracy suffice and per-class breakdown adds little
- When no ground-truth labels exist for the query data, since per-class F1 and correct-assignment
  counts cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SELINA
  role: Annotation framework whose baseline (no SMOTE) and SMOTE-balanced variants
    are benchmarked with per-class F1; run via selina train and selina predict
  repo: https://github.com/SELINA-team/SELINA-reference
- name: Python
  role: Used to compute per-class correct-assignment counts and F1 scores and assemble
    the comparison table
- name: R
  role: SELINA is distributed as both python and R packages; R/Seurat objects carry
    the query preprocessing results used in prediction
  repo: https://github.com/SELINA-team/SELINA.py
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:5494f55919ddb4a3556dcc48519cb06dd2d9f022a2b5778ad0d31d73b5f96d9f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA-reference
  tools:
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-class-f1-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates cell-type annotation performance by computing correct-assignment counts and per-class F1 scores, specifically to quantify whether a model variant (e.g., SMOTE oversampling in SELINA) rescues rare cell-type annotation that is masked by overall accuracy.

## When to use

Use when benchmarking an annotation or classification model whose improvement is concentrated in minority classes, and aggregate metrics would hide the rare-class gains.

## When NOT to use

- When the goal is overall annotation accuracy on balanced cell types — macro aggregates or accuracy suffice and per-class breakdown adds little
- When no ground-truth labels exist for the query data, since per-class F1 and correct-assignment counts cannot be computed

## Inputs

- Per-cell predicted labels and probabilities from each model variant (query_predictions.txt, query_probability.txt from selina predict)
- Ground-truth cell-type labels for the query dataset
- List of rare/target cell types to evaluate (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE)

## Outputs

- Per-cell-type correct-assignment counts for each model condition
- Per-class F1 scores for each model condition
- Combined comparison table (smote_ablation_f1_table.csv) reproducing Figure 3A–3B trends

## How to apply

Annotate the same query dataset with each model variant — here, a baseline SELINA model trained on the 4 liver reference datasets (15,859 cells) without cell-type balancing, and a second model trained with SMOTE applied to rare cell types before pre-training — using 'selina predict' with prob-cutoff 0.9. For each condition, count correctly assigned query cells per cell type (e.g., CD4T, plasma, mast, pDC, cholangiocyte, PL, PE) and compute per-class F1 scores rather than overall accuracy, since rare classes contribute little to global metrics. Assemble the correct counts and F1 values from both runs into a single side-by-side comparison table (e.g., smote_ablation_f1_table.csv) so per-class deltas reveal the effect of the intervention. Validate by confirming the SMOTE-trained model assigns more query cells to each rare type than the baseline, consistent with the reported recovery of roughly one-third of CD4T cells and about half of mast cells.

## Related tools

- **SELINA** (Annotation framework whose baseline (no SMOTE) and SMOTE-balanced variants are benchmarked with per-class F1; run via selina train and selina predict) — https://github.com/SELINA-team/SELINA-reference
- **Python** (Used to compute per-class correct-assignment counts and F1 scores and assemble the comparison table)
- **R** (SELINA is distributed as both python and R packages; R/Seurat objects carry the query preprocessing results used in prediction) — https://github.com/SELINA-team/SELINA.py

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-train/pre-trained_params.pt --prob-cutoff 0.9 --path-out ./res/predict/
```

## Evaluation signals

- The SMOTE-trained model assigns strictly more query cells to each rare cell type (CD4T, plasma, mast, pDC, cholangiotype, PL, PE) than the baseline model
- Per-class F1 improves across most rare cell types under SMOTE, matching Figure 3A–3B trends
- Recovery magnitudes are consistent with reported values: nearly 1/3 of CD4T cells and about half of mast cells correctly annotated after SMOTE
- Comparison table rows align: each rare cell type has correct counts and F1 values populated for both baseline and SMOTE conditions
- Predictions are filtered at prob-cutoff 0.9 so per-class counts reflect high-confidence assignments only

## Limitations

- SMOTE-style cell-type balancing targets minority-class annotation; gains reported are specific to rare cell types and may not reflect overall accuracy changes
- Per-class F1 requires trustworthy ground-truth labels for the query; the validation here relies on known liver query annotations
- Annotation quality depends on reference composition — the reference atlas still suffers from imbalances in tissue-specific datasets and cell-type abundance

## Evidence
