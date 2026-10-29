---
name: sequencing-protocol-stratified-benchmarking
description: Use when a cell type classifier trained on 10x 3'/5' scRNA-seq must be
  evaluated on cells from other sequencing protocols (STRT-seq, Smart-seq2, BD Rhapsody
  Targeted mRNA) to quantify protocol-specific generalization loss.
when_to_use_negative:
- When the goal is in-distribution benchmarking on holdout 10X data only (the donor-based
  split), not cross-protocol transfer
- When author cell type labels are free-text and not mappable to Cell Ontology terms,
  since the ontology-aware correctness rule (exact match or subtype via child-node
  lookup) cannot be applied
- When a protocol group has too few cells to yield a stable macro F1 estimate — the
  paper's benchmark excluded uncommon assay technologies due to data availability
edam_operation: http://edamontology.org/operation_0389
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scTab
  role: Provides the trained TabNet cross-tissue cell type classifier and the model
    evaluation notebook infrastructure (classification-tabnet.ipynb) being benchmarked
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Architecture behind the TabnetClassifier checkpoint loaded from cellnet/models.py
    for inference on test cells
- name: NumPy
  role: Implements the sf_normalize routine used for size-factor normalization to
    10000 counts before log1p transformation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:1940328d4eb515dc959114805896b4b3d2547c786a2d6871c6d62927d87ca391
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - scTab
  edam_operation: http://edamontology.org/operation_0389
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequencing-protocol-stratified-benchmarking

## Summary

Benchmark out-of-distribution protocol transfer of a cross-tissue cell type annotation model (scTab/TabNet) by grouping donor-holdout test cells by sequencing assay and computing macro F1 per protocol group. It exposes degradation from 10X-trained inference (~0.8 macro F1) to non-10X protocols (~0.4) as reported in Supp. Fig. 7b.

## When to use

Use when a cell type classifier trained on 10x 3'/5' scRNA-seq must be evaluated on cells from other sequencing protocols (STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA) to quantify protocol-specific generalization loss.

## When NOT to use

- When the goal is in-distribution benchmarking on holdout 10X data only (the donor-based split), not cross-protocol transfer
- When author cell type labels are free-text and not mappable to Cell Ontology terms, since the ontology-aware correctness rule (exact match or subtype via child-node lookup) cannot be applied
- When a protocol group has too few cells to yield a stable macro F1 estimate — the paper's benchmark excluded uncommon assay technologies due to data availability

## Inputs

- Trained TabNet/scTab model checkpoint (scTab-checkpoints.tar.gz)
- Donor-holdout test split (~3.45 million cells, test fraction 0.15) with assay annotations
- Cell Ontology child-node lookup matrix (04_create_hierarchy_matrices.ipynb output)
- Gene list of 19331 protein-coding genes (notebooks/store_creation/features.parquet)

## Outputs

- protocol_macro_f1_scores.csv with one macro F1-score per sequencing protocol group
- Per-cell predicted cell type labels and class probabilities for the test split

## How to apply

Load the trained TabNet classifier checkpoint via cellnet/models.py (TabnetClassifier) and the donor-holdout test split (test fraction 0.15, ~3.45 million cells). Preprocess cells identically to training: NumPy sf_normalize to 10000 counts per cell, log1p transformation, and restrict to the 19331 protein-coding gene space (features.parquet). Run inference to get class probabilities and labels, then group cells by their assay annotation into 10x 3'/5' versus non-10X protocols. Apply the ontology-aware correctness rule with the Cell Ontology child-node lookup (cellnet/utils/cell_ontology.py): a prediction is correct only if it exactly matches the author label or predicts a subtype; a parent type or anything else is wrong. Compute macro F1 per protocol group (per-cell-type F1 averaged with equal weight to handle class imbalance), mirroring the classification-tabnet.ipynb notebook, and write grouped scores to protocol_macro_f1_scores.csv.

## Related tools

- **scTab** (Provides the trained TabNet cross-tissue cell type classifier and the model evaluation notebook infrastructure (classification-tabnet.ipynb) being benchmarked) — https://github.com/theislab/scTab
- **TabNet** (Architecture behind the TabnetClassifier checkpoint loaded from cellnet/models.py for inference on test cells)
- **NumPy** (Implements the sf_normalize routine used for size-factor normalization to 10000 counts before log1p transformation)

## Examples

```
from cellnet.models import TabnetClassifier; model = TabnetClassifier.load_from_checkpoint('scTab-checkpoints.tar.gz'); y_pred = model.predict(X_sf_log1p)
```

## Evaluation signals

- Holdout 10x 3'/5' protocol group reaches macro F1 around 0.8, while non-10X protocol groups fall to around 0.4, matching Supp. Fig. 7b
- STRT-seq, Smart-seq2, and BD Rhapsody Targeted mRNA are the worst-generalizing groups and should show the lowest per-protocol macro F1 scores
- Ontology-aware scoring: predictions equal to the author label or one of its Cell Ontology child nodes count as correct; parent types and unrelated labels count as wrong
- protocol_macro_f1_scores.csv contains exactly one macro F1 entry per protocol group, computed with equal weight per cell type
- Preprocessing invariants: every test cell is normalized to 10000 counts, log1p transformed, and restricted to the 19331 protein-coding gene space before inference

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets that are imbalanced not only across cell types but also across tissues and sequencing protocols
- Cell Ontology incompleteness: not all cell types (especially rare ones) match an ontology term, and ontology relationships are still under active discussion, which affects the correctness metric
- The benchmark is restricted to common sequencing protocols; cells from less common assay technologies were filtered out, so transfer performance on those technologies is unknown

## Evidence

- [results] a macro F1-Score of ~0.4 (vs ~0.8 on holdout 10X data), but generalizes poorly to STRT-seq, Smart-seq2, and BD Rhapsody Targeted mRNA: [evidence span withheld — anchor status: not_found]
- [methods] 1. Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [readme] Model checkpoints: https://pklab.med.harvard.edu/felix/data/scTab-checkpoints.tar.gz (8.1GB): "Model checkpoints: https://pklab.med.harvard.edu/felix/data/scTab-checkpoints.tar.gz (8.1GB)"
- [discussion] plain macro F1-scores may not properly reflect the complexity of unbalanced datasets: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets"
