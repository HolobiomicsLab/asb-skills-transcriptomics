---
name: hierarchical-cell-ontology-label-aggregation
description: Use when comparing cell-label predictors (e.g. zero-shot scPRINT vs CellTypist
  vs supervised baselines) on labels whose granularity differs across methods or annotation
  schemes, and you need ontology-consistent, coarser-grained evaluation.
when_to_use_negative:
- All methods are already evaluated on the same coarse labels, or the label set has
  no ontology-defined parent-child structure to roll up
- The goal is zero-shot label prediction itself rather than post-hoc re-evaluation
  of existing predictions at matched granularity
tools:
- name: scPRINT
  role: Source of zero-shot cell type predictions whose hierarchical labels are aggregated
    for multi-granularity evaluation
  repo: https://github.com/cantinilab/scPRINT
- name: CellTypist
  role: Marker-based classifier baseline run with the 'Adult_Human_PancreaticIslet'
    database (14 cell types); its label space motivates ontology-level aggregation
    and the noted 4/14-type ceiling
- name: scikit-learn
  role: Used to compute accuracy and macro-F1 (average F1 across classes) for each
    method at each label granularity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:4ca905b08d6ee9172ade6b10d4d51bd459343a18244d7dbb503f05f444297da5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - CellTypist
  - Python
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hierarchical-cell-ontology-label-aggregation

## Summary

Rolls fine-grained predicted and ground-truth cell type labels up to coarser ontology terms (e.g. aggregating pancreatic endocrine subtypes to 'endocrine pancreatic cell') so classifier performance can be evaluated at multiple label granularities using accuracy and macro-F1.

## When to use

Use when comparing cell-label predictors (e.g. zero-shot scPRINT vs CellTypist vs supervised baselines) on labels whose granularity differs across methods or annotation schemes, and you need ontology-consistent, coarser-grained evaluation.

## When NOT to use

- All methods are already evaluated on the same coarse labels, or the label set has no ontology-defined parent-child structure to roll up
- The goal is zero-shot label prediction itself rather than post-hoc re-evaluation of existing predictions at matched granularity

## Inputs

- Per-cell fine-grained ground-truth cell type labels (e.g. open-problems multi-batch human pancreas annotations)
- Per-cell predicted labels from each method (scPRINT zero-shot predictions, CellTypist with 'Adult_Human_PancreaticIslet' database, openproblems logistic regression and xgboost on 25 PCs)
- Cell ontology / label-hierarchy mapping defining parent terms such as 'endocrine pancreatic cell'

## Outputs

- Per-method, per-granularity classification metrics (accuracy and macro-F1) compiled into classification_benchmark_results.csv
- Aggregated label vectors at the coarser ontology level (e.g. 'endocrine pancreatic cell')

## How to apply

After each classifier produces predictions on the test split, define the parent ontology term(s) covering the fine-grained labels (in the open-problems pancreas benchmark, fine labels are aggregated under 'endocrine pancreatic cell') and map both predicted and true labels to their ancestors. Recompute accuracy and macro-F1 (average F1 across classes) at both granularities so per-method results are comparable. The rationale is grounded in scPRINT's own treatment of labels as hierarchical: the model uses a hierarchical loss during its classification pretraining task, reflecting that cell type labels have a tree structure. Compile per-method, per-granularity metrics into a single results table (e.g. classification_benchmark_results.csv). Note known ceiling effects, e.g. CellTypist's 'Adult_Human_PancreaticIslet' database covers only 4 of 14 database cell types over ~70% of cells, which coarse aggregation partially mitigates.

## Related tools

- **scPRINT** (Source of zero-shot cell type predictions whose hierarchical labels are aggregated for multi-granularity evaluation) — https://github.com/cantinilab/scPRINT
- **CellTypist** (Marker-based classifier baseline run with the 'Adult_Human_PancreaticIslet' database (14 cell types); its label space motivates ontology-level aggregation and the noted 4/14-type ceiling)
- **scikit-learn** (Used to compute accuracy and macro-F1 (average F1 across classes) for each method at each label granularity)

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, y_pred); macro_f1 = f1_score(y_true, y_pred, average='macro')
```

## Evaluation signals

- The results table contains accuracy and macro-F1 for every method at both label granularities (fine-grained and ontology-aggregated)
- CellTypist's ceiling (4 of 14 database cell types covering ~70% of cells) is reflected in its accuracy relative to scPRINT's zero-shot predictions
- Every mapped prediction/ground-truth pair is assigned to a valid ontology parent (no unmapped or orphan labels after aggregation)
- Macro-F1 is the average F1 across classes, so class imbalance in the coarser aggregation is handled consistently across methods
- scPRINT's zero-shot accuracy on the pancreas benchmark (62%) is reproducible at the fine-grained level before aggregation

## Limitations

- Aggregation to coarser ontology terms can mask differences between fine subtypes, so coarse-grained accuracy should be reported alongside fine-grained metrics
- Marker-based baselines like CellTypist are limited by database coverage (only 4 of 14 cell types covering ~70% of cells here), which biases cross-method comparisons at any granularity
- scPRINT's zero-shot predictions depend on its random sampling of genes at a context of 4000 genes, so results can vary with sampling and context size

## Evidence

- [methods] scPRINT hierarchical label structure: "Due to the hierarchical structure of the prediction, we also created a hierarchical loss."
- [abstract] zero-shot cell label prediction ability: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [abstract] pre-training scale and purpose: "we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells"
