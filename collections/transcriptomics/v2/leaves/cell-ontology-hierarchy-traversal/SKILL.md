---
name: cell-ontology-hierarchy-traversal
description: Use when evaluating a Cell Ontology-based cell type classifier at multiple
  annotation granularities, or when fine labels must be collapsed into coarser parent
  classes (e.g. 164 → 31 labels) using ontology-derived information content.
when_to_use_negative:
- Cell type labels are not drawn from the Cell Ontology (e.g. free-text or non-CL
  nomenclature), so subtype/parent relations cannot be resolved
- Evaluation goal is strict exact-match accuracy only, where ontology-derived partial
  credit would change the metric definition
- The cell types of interest are rare types that may not be correctly matched to a
  corresponding ontology term, making the hierarchy unreliable
tools:
- name: scTab
  role: Classifier whose 164 CL-based labels are traversed and whose test predictions
    are scored at coarse vs fine granularity
  repo: https://github.com/theislab/scTab
- name: Ubergraph
  role: Integrated OBO ontology triplestore containing the Cell Ontology with precomputed
    information content scores per class, useful for ontology hierarchy queries
  repo: https://github.com/INCATools/ubergraph
- name: NumPy
  role: Array operations used for normalization and hierarchy-matrix computation in
    the cellnet codebase
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:ff2196cf73f96d706125125d34d81024cba4eb02813ca061a727e33691ba7318
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - Ubergraph
  - cellxgene-census
  - scTab
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cell-ontology-hierarchy-traversal

## Summary

Traverses the Cell Ontology (CL) child/parent structure — via the child node lookup matrix built in `04_create_hierarchy_matrices.ipynb` — to map the 164 fine-grained cell type labels of scTab to 31 coarse classes using information-content scores, enabling coarse-vs-fine granularity evaluation of macro F1 (0.897 vs 0.830).

## When to use

Use when evaluating a Cell Ontology-based cell type classifier at multiple annotation granularities, or when fine labels must be collapsed into coarser parent classes (e.g. 164 → 31 labels) using ontology-derived information content.

## When NOT to use

- Cell type labels are not drawn from the Cell Ontology (e.g. free-text or non-CL nomenclature), so subtype/parent relations cannot be resolved
- Evaluation goal is strict exact-match accuracy only, where ontology-derived partial credit would change the metric definition
- The cell types of interest are rare types that may not be correctly matched to a corresponding ontology term, making the hierarchy unreliable

## Inputs

- Child node lookup matrix built from the Cell Ontology (CL) in cellnet/utils/cell_ontology.py
- scTab test-set predictions from the donor hold-out split (~3.4M cells) with 164 fine-grained cell type labels
- Ground-truth cell type annotations from the CELLxGENE census (2023-05-15) dataset

## Outputs

- Mapping of 164 fine-grained labels to 31 coarse label classes (labels with information content ≤ 60)
- Per-cell-type macro F1-scores at fine and coarse granularity, averaged per level
- Comparison table (e.g. coarse_vs_fine_f1) reproducing Supp. Fig. 7a (0.897 coarse vs 0.830 fine)

## How to apply

Load the child node lookup matrix produced by `04_create_hierarchy_matrices.ipynb`, which was built from the Cell Ontology code in `cellnet/utils/cell_ontology.py`. Compute the information content of each of the 164 fine-grained labels and select labels with an information content score ≤ 60 to form the 31 coarse label classes. Then, when scoring predictions from the donor hold-out test split (~3.4M cells, 15% of the 22.19M-cell corpus), count a prediction as right if it equals the true label or is an ontology subtype of it, and wrong if it predicts a parent cell type or anything else. Compute macro F1-score per cell type at both granularities and average per level to mitigate class imbalance; the rationale is that the CELLxGENE corpus is ontology-based, so subtype relationships carry partial credit information.

## Related tools

- **scTab** (Classifier whose 164 CL-based labels are traversed and whose test predictions are scored at coarse vs fine granularity) — https://github.com/theislab/scTab
- **Ubergraph** (Integrated OBO ontology triplestore containing the Cell Ontology with precomputed information content scores per class, useful for ontology hierarchy queries) — https://github.com/INCATools/ubergraph
- **NumPy** (Array operations used for normalization and hierarchy-matrix computation in the cellnet codebase)

## Examples

```
from cellnet.utils import cell_ontology as co; child = co.load_child_node_matrix(); ic = co.information_content(child); coarse_labels = [l for l in ic if ic[l] <= 60]
```

## Evaluation signals

- Computed coarse macro F1 (0.897) exceeds the fine macro F1 (0.830), matching Supp. Fig. 7a within reasonable tolerance
- The coarse label set contains exactly 31 classes derived from labels with information content ≤ 60
- Granularity rules applied: a prediction is correct if it equals the true label or is an ontology subtype; parent-type predictions count as wrong
- Test-set evaluation uses the donor hold-out split (~3.4M cells, 15% of 22.19M cells) so per-level macro F1 is computed over held-out donors
- Child node lookup matrix successfully resolves subtype relations for all 164 fine labels via cellnet/utils/cell_ontology.py

## Limitations

- The Cell Ontology is still a work in progress: not all cell types, especially rare cell types, can be correctly matched to a corresponding ontology term
- Relationships between cell types given by the Cell Ontology are still a topic of active discussion, which can affect the classification metrics
- Pre-trained scTab models still contain fairly coarsely annotated cell type labels despite filtering; current ontologies also lack resolution especially for brain tissue

## Evidence

- [methods] Create child node lookup matrix to find subtypes based on cell type ontology: "4. `04_create_hierarchy_matrices.ipynb`: Create child node lookup matrix to find subtypes based on cell type ontology"
- [discussion] The CELLxGENE data corpus used here is based on the cell ontology: "The CELLxGENE data corpus used here is based on the cell ontology."
- [discussion] not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types: "not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types."
- [readme] Precomputed information content score for each ontology class, based on the count of terms related to a given term via rdfs:subClassOf: [evidence span withheld — anchor status: external_doc]
