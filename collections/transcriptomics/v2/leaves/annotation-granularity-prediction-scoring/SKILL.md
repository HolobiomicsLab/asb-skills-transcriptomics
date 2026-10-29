---
name: annotation-granularity-prediction-scoring
description: Use when evaluating whether a cell type classifier's predictions are
  acceptable at coarser annotation granularity, e.g. reproducing Supp. Fig. 7a of
  scTab on the donor hold-out test split.
when_to_use_negative:
- Predictions or labels are not mapped to Cell Ontology (CL) terms, so subtype/parent
  relationships cannot be resolved
- Test set lacks donor-based splitting, so granularity scores would not measure generalization
  to new donors
- Cell types are extremely rare with no ontology term match — the paper notes not
  all cell types can be correctly matched to an ontology term
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: scTab
  role: Trained cell type prediction model whose test-set predictions are scored at
    coarse and fine granularity
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: Standardised API used to query the CELLxGENE census (version 2023-05-15) that
    supplies the annotated single-cell corpus
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- name: NumPy
  role: Used for array computations such as sf_normalize during data preparation feeding
    the evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:3075a2114a4bbb4bc8a82ed355d8324a944a9b517743dd7294b3179c06677d72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# annotation-granularity-prediction-scoring

## Summary

Reproduces scTab's coarse (31 labels) vs. fine (164 labels) cell type granularity evaluation by mapping fine-grained CELLxGENE labels to coarse classes via Cell Ontology information content and scoring donor hold-out test predictions with granularity-aware macro F1 (coarse 0.897 vs. fine 0.830).

## When to use

Use when evaluating whether a cell type classifier's predictions are acceptable at coarser annotation granularity, e.g. reproducing Supp. Fig. 7a of scTab on the donor hold-out test split.

## When NOT to use

- Predictions or labels are not mapped to Cell Ontology (CL) terms, so subtype/parent relationships cannot be resolved
- Test set lacks donor-based splitting, so granularity scores would not measure generalization to new donors
- Cell types are extremely rare with no ontology term match — the paper notes not all cell types can be correctly matched to an ontology term

## Inputs

- Child node lookup matrix from 04_create_hierarchy_matrices.ipynb built from the Cell Ontology (CL)
- scTab test-set predictions on the donor hold-out test split (~3.4M cells, 15% of 22.19M-cell dataset)
- True fine-grained cell type labels (164 classes) for test cells

## Outputs

- Coarse-vs-fine macro F1 comparison table written to coarse_vs_fine_f1
- Per-cell-type macro F1 scores at fine (164-label) and coarse (31-label) granularity

## How to apply

Load the child node lookup matrix built from the Cell Ontology (CL) via cellnet/utils/cell_ontology.py in notebook 04_create_hierarchy_matrices.ipynb, and compute the information content of each of the 164 fine-grained labels; select labels with information content ≤ 60 to form the 31 coarse classes. Load scTab test-set predictions from the donor hold-out test split (~3.4M cells, 15% of the 22.19M-cell corpus). Apply the granularity rules: a prediction counts as right if it equals the true label or is a subtype of it (via the child node lookup matrix), and as wrong if it predicts a parent cell type or anything else. Compute macro F1-score per cell type at both fine and coarse granularity and average per level to account for class imbalance, then compare against the reported 0.897 (coarse) vs. 0.830 (fine).

## Related tools

- **scTab** (Trained cell type prediction model whose test-set predictions are scored at coarse and fine granularity) — https://github.com/theislab/scTab
- **cellxgene-census** (Standardised API used to query the CELLxGENE census (version 2023-05-15) that supplies the annotated single-cell corpus) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- **NumPy** (Used for array computations such as sf_normalize during data preparation feeding the evaluation)

## Examples

```
ic = compute_information_content(hierarchy_matrix); coarse_labels = [l for l in fine_164 if ic[l] <= 60]; f1_coarse, f1_fine = granularity_f1(preds, truth, child_lookup)
```

## Evaluation signals

- Computed coarse macro F1 (0.897) exceeds computed fine macro F1 (0.830), matching Supp. Fig. 7a within reasonable tolerance
- Exactly 31 coarse label classes are produced when filtering fine labels by information content ≤ 60
- Prediction counted as correct only when it equals the true label or is a subtype per the child node lookup matrix; parent-type predictions are wrong
- Test split is the donor hold-out set covering ~3.4M cells (15% of the 22.19M-cell corpus)

## Limitations

- Cell Ontology incompleteness: not all cell types (especially rare ones) match an ontology term
- Cell Ontology cell type relationships are still under active discussion, which can affect classification metrics
- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets imbalanced across cell types, tissues, and sequencing protocols
- Current pre-trained scTab models still contain some fairly coarsely annotated cell type labels despite filtering efforts

## Evidence

- [methods] Coarse label selection by information content: "4. `04_create_hierarchy_matrices.ipynb`: Create child node lookup matrix to find subtypes based on cell type ontology"
- [supplementary] Granularity rule and reported scores: [evidence span withheld — anchor status: not_found]
- [supplementary] Reported coarse vs fine macro F1: [evidence span withheld — anchor status: not_found]
- [discussion] Ontology incompleteness caveat: "not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types"
- [discussion] Ontology relationship instability caveat: "relationships between cell types given by the Cell Ontology are still a topic of active discussion, which can affect the classification metrics"
- [readme] Repository supplies model checkpoints and data: "Model checkpoints and traning data"
