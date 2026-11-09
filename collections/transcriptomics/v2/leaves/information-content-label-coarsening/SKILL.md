---
name: information-content-label-coarsening
description: Use when evaluating a cell type classifier on a label hierarchy — e.g.
  mapping ~164 Cell Ontology fine labels to ~31 coarse classes via information content
  and comparing macro F1 across granularity levels.
when_to_use_negative:
- Cell type labels are free-text annotations not resolvable to Cell Ontology terms
  — the subtype/parent grading requires a valid CL hierarchy
- The analysis goal is fine-grained subtype discrimination only and granularity relaxation
  would mask real errors (e.g. coarse labels like 'native cell' being predicted)
- Predictions and ground truth come from a flat label set with no ontology-defined
  parent–child relations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTab
  role: Provides the trained cross-tissue TabNet-based classifier and its donor hold-out
    test-set predictions that are re-scored at both label granularities
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Used in the scTab codebase for the label-matrix normalization and information-content
    computations over the ontology lookup matrices
- name: cellxgene-census
  role: Source API for querying the CELLxGENE CxG census (version 2023-05-15) that
    supplies the cells, donors, and cell type annotations being evaluated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:7046588e4d7df966e5a27c05fffc2825226d9d930c3cb5d402da13d4dfc80036
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Information-content-based label coarsening

## Summary

A granularity-aware evaluation method from the scTab paper that maps 164 fine-grained Cell Ontology cell type labels to 31 coarse classes using information content (IC ≤ 60) and computes ontology-aware macro F1 at both levels. It quantifies how much a cross-tissue classifier (scTab) gains when annotation tolerance is relaxed from fine subtypes to parent cell types.

## When to use

Use when evaluating a cell type classifier on a label hierarchy — e.g. mapping ~164 Cell Ontology fine labels to ~31 coarse classes via information content and comparing macro F1 across granularity levels.

## When NOT to use

- Cell type labels are free-text annotations not resolvable to Cell Ontology terms — the subtype/parent grading requires a valid CL hierarchy
- The analysis goal is fine-grained subtype discrimination only and granularity relaxation would mask real errors (e.g. coarse labels like 'native cell' being predicted)
- Predictions and ground truth come from a flat label set with no ontology-defined parent–child relations

## Inputs

- Child node lookup matrix from Cell Ontology CL hierarchy (`04_create_hierarchy_matrices.ipynb` output)
- scTab test-set predictions from the donor hold-out split (~3.4M cells) on the 22.19M-cell CELLxGENE census (2023-05-15)
- Fine-grained ground-truth cell type labels (164 Cell Ontology terms)

## Outputs

- Mapping of 164 fine labels to 31 coarse classes (labels with IC ≤ 60)
- Per-cell-type macro F1 at fine and coarse granularity
- Comparison table of coarse vs. fine macro F1 (e.g. `coarse_vs_fine_f1`)

## How to apply

Load the child node lookup matrix produced by `04_create_hierarchy_matrices.ipynb`, which encodes Cell Ontology (CL) subtyping via `cellnet/utils/cell_ontology.py`, and compute the information content of each of the 164 fine labels. Select labels with information content score ≤ 60 to build the 31 coarse label classes, mapping each fine label to its coarse parent. On the scTab donor hold-out test split (~3.4M cells, 15% of the 22.19M-cell CELLxGENE corpus), grade each prediction ontology-aware: correct if it equals the true label or is a subtype of it, wrong if it predicts a parent cell type or anything else. Compute macro F1 per cell type at both fine and coarse granularity and average per level to account for class imbalance, then report coarse (0.897) vs. fine (0.830) macro F1 as in Supp. Fig. 7a.

## Related tools

- **scTab** (Provides the trained cross-tissue TabNet-based classifier and its donor hold-out test-set predictions that are re-scored at both label granularities) — https://github.com/theislab/scTab
- **NumPy** (Used in the scTab codebase for the label-matrix normalization and information-content computations over the ontology lookup matrices)
- **cellxgene-census** (Source API for querying the CELLxGENE CxG census (version 2023-05-15) that supplies the cells, donors, and cell type annotations being evaluated)

## Examples

```
jupyter nbconvert --to notebook --execute notebooks/store_creation/04_create_hierarchy_matrices.ipynb
```

## Evaluation signals

- The coarse label set contains exactly 31 classes derived from the 164 fine labels under the IC ≤ 60 cutoff
- Computed coarse macro F1 (~0.897) exceeds the fine macro F1 (~0.830), and both reproduce Supp. Fig. 7a within reasonable tolerance
- Ontology-aware grading is correct: a prediction equal to the true label or a subtype counts as right; a parent-cell-type prediction counts as wrong
- Per-level averaging over cell types is applied so class imbalance across the 164/31 label levels does not skew the macro F1

## Limitations

- Cell Ontology incompleteness: not all cell types (especially rare ones) match an ontology term, which distorts the subtype-based grading
- Cell Ontology cell type relationships are still under active discussion, which can affect the classification metrics
- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets (imbalanced across cell types, tissues, and sequencing protocols)

## Evidence

- [methods] hierarchy notebook: "4. `04_create_hierarchy_matrices.ipynb`: Create child node lookup matrix to find subtypes based on cell type ontology"
- [discussion] ontology incompleteness: "As the Cell Ontology is still a work in progress, not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types."
- [discussion] ontology relationships under discussion: "relationships between cell types given by the Cell Ontology are still a topic of active discussion, which can affect the classification metrics discussed here"
- [readme] metric docs: "``classification-evaluation-metrics.md``: Details about used evaluation metrics"
