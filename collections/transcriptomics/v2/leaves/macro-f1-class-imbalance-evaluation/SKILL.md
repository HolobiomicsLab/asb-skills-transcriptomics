---
name: macro-f1-class-imbalance-evaluation
description: Use when scoring cell type annotation predictions on heavily class-imbalanced
  single-cell datasets (e.g. 164 cell types, 22.19M cells) where accuracy would be
  dominated by abundant types and ontology subtypes should count as correct.
when_to_use_negative:
- The task calls for micro-averaged or per-cell accuracy rather than equal-weight
  per-cell-type scoring, since macro F1 deliberately up-weights rare types
- Predictions are graded against free-text annotations not mapped to Cell Ontology
  terms, so the subtype-correctness rule cannot be applied
- The dataset is small and balanced, making the imbalance-correction rationale of
  macro averaging moot
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTab
  role: TabNet-based cell type classifier whose test predictions are scored with the
    ontology-aware macro F1 metric
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Provides sf_normalize size-factor normalization used in the paper's evaluation
    code; prediction/label arrays are processed with NumPy when computing per-type
    F1
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:4da861bf39874e6e602c51d6635bcd8b261a50df2c220f76f5878340ee89334d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# macro-f1-class-imbalance-evaluation

## Summary

Evaluate scTab cell type classifiers with macro F1-score (per-cell-type F1 averaged with equal weight) combined with Cell Ontology-aware correctness rules, so that rare cell types count equally and predicting a subtype of the true label is scored as correct.

## When to use

Use when scoring cell type annotation predictions on heavily class-imbalanced single-cell datasets (e.g. 164 cell types, 22.19M cells) where accuracy would be dominated by abundant types and ontology subtypes should count as correct.

## When NOT to use

- The task calls for micro-averaged or per-cell accuracy rather than equal-weight per-cell-type scoring, since macro F1 deliberately up-weights rare types
- Predictions are graded against free-text annotations not mapped to Cell Ontology terms, so the subtype-correctness rule cannot be applied
- The dataset is small and balanced, making the imbalance-correction rationale of macro averaging moot

## Inputs

- Test-set cell type predictions from a trained scTab (TabNet) classifier on the donor hold-out split (~3,448,832 cells)
- True author cell type labels (164 fine-grained types) for each test cell
- Child-node lookup matrix derived from the Cell Ontology (CL) hierarchy
- Assay/protocol annotations per cell for per-protocol grouping (e.g. 10x 3'/5' vs STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA)
- Information content scores per ontology term for building coarse label sets

## Outputs

- Macro F1-score per cell type and the macro-averaged score per evaluation level (fine vs. coarse, or per sequencing-protocol group)
- Comparison table of coarse vs. fine macro F1 (e.g. coarse_vs_fine_f1.. , protocol_macro_f1_scores.csv, scTab_model_scaling_results.csv)

## How to apply

Load scTab test-set predictions from the donor hold-out split (~3.45M cells, 15% of the 22.19M-cell CELLxGENE census v2023-05-15 corpus) and apply the ontology-aware correctness rule using the child-node lookup matrix from `04_create_hierarchy_matrices.ipynb` (Cell Ontology CL via `cellnet/utils/cell_ontology.py`): a prediction is correct if it equals the true label or is a subtype of it, and wrong if it predicts a parent cell type or anything else. Compute F1 per cell type and average with equal weight per type (macro averaging) to handle strong class imbalance across cell types, tissues, and sequencing protocols. Compute macro F1 per subgroup when relevant — per label granularity level (164 fine vs. 31 coarse labels selected by information content ≤ 60) or per sequencing-protocol group (10X vs. non-10X assays). Validate against paper anchors: 0.897 coarse vs. 0.830 fine macro F1 (Supp. Fig. 7a), ~0.8 on holdout 10X vs. ~0.4 on non-10X protocols (Supp. Fig. 7b), and 0.7864±0.0010 / 0.8323±0.0010 for the 1.7M / 16.2M parameter models.

## Related tools

- **scTab** (TabNet-based cell type classifier whose test predictions are scored with the ontology-aware macro F1 metric) — https://github.com/theislab/scTab
- **NumPy** (Provides sf_normalize size-factor normalization used in the paper's evaluation code; prediction/label arrays are processed with NumPy when computing per-type F1)

## Examples

```
from cellnet.utils.cell_ontology import get_child_lookup; correct = (pred == y) | get_child_lookup()[y, pred]; f1_per_type = sklearn.metrics.f1_score(y[correct], pred[correct], average='macro')
```

## Evaluation signals

- Coarse macro F1 (31 labels, information content ≤ 60) must exceed fine macro F1 (164 labels) and reproduce ~0.897 vs. ~0.830 as in Supp. Fig. 7a
- Holdout 10X-protocol macro F1 should reach ~0.8 while non-10X protocol groups drop to ~0.4, matching Supp. Fig. 7b
- Model-scaling reproduction: macro F1 of 0.7864±0.0010 (1.7M params) and 0.8323±0.0010 (16.2M params) on the donor-held-out test set
- Correctness rule sanity check: predictions equal to the true label or to one of its Cell Ontology subtypes are counted correct; parent-type predictions are counted wrong
- Augmented vs. non-augmented models should show macro F1 improvement from 0.7755±0.0020 to 0.7841±0.0030 (p=0.0016)

## Limitations

- Plain macro F1-scores may not properly reflect datasets imbalanced across cell types, tissues, and sequencing protocols; more detailed evaluation metrics may be needed
- Cell Ontology relationships are still under active discussion, which can affect the classification metrics computed with the subtype-correctness rule
- Not all cell types (especially rare ones) can be correctly matched to a Cell Ontology term, biasing ontology-based scoring for those labels

## Evidence

- [methods] Ontology-aware macro F1 correctness rule: [evidence span withheld — anchor status: not_found]
- [methods] Macro averaging rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Metric caveat from discussion: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types"
- [discussion] Ontology incompleteness caveat: "not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types"
- [readme] README pointer to metric documentation: "``classification-evaluation-metrics.md``: Details about used evaluation metrics"
