---
name: ontology-aware-prediction-scoring
description: Use when evaluating cell type classifiers on CELLxGENE-style annotations
  where predictions of a more specific Cell Ontology subtype should count as correct
  against a parent true label.
when_to_use_negative:
- The label space has no ontology structure or you need strict exact-match accuracy
  — the ontology-aware rule deliberately credits subtype predictions, inflating scores
  relative to plain accuracy.
- Evaluating on cell types not mappable to Cell Ontology terms (e.g., rare or coarsely
  annotated labels) — the article notes not all cell types can be matched to an ontology
  term.
- You need metrics reflecting tissue/protocol imbalance — the article warns plain
  macro F1 may not properly reflect the complexity of unbalanced datasets.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: scTab
  role: Cell type classifier whose TabNet-based predictions are scored with the ontology-aware
    rule
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: Provides the CELLxGENE census corpus (v2023-05-15) with author cell type annotations
    used as ground truth
- name: NumPy
  role: sf_normalize preprocessing and lookup-matrix arithmetic for ontology-aware
    scoring
- name: INCATools/ubergraph
  role: Integrated OBO ontology triplestore including Cell Ontology, usable to retrieve
    subtype relations
  repo: https://github.com/INCATools/ubergraph
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:5a298b1d0d80abeca28ea02f07f33ae5f2fba13391f46620b124a0e525da6ed7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - CIForm
  - CellTypist
  - INCATools/ubergraph
  - NumPy
  - TabNet
  - cellxgene-census
  - scGPT
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ontology-aware-prediction-scoring

## Summary

Scores cell type classifier predictions using the Cell Ontology child-node lookup so a prediction is counted correct if it matches the annotator label exactly or predicts a subtype of it. This is the evaluation rule used across all scTab benchmark tasks (macro F1 and test loss) to credit biologically consistent subtype predictions.

## When to use

Use when evaluating cell type classifiers on CELLxGENE-style annotations where predictions of a more specific Cell Ontology subtype should count as correct against a parent true label.

## When NOT to use

- The label space has no ontology structure or you need strict exact-match accuracy — the ontology-aware rule deliberately credits subtype predictions, inflating scores relative to plain accuracy.
- Evaluating on cell types not mappable to Cell Ontology terms (e.g., rare or coarsely annotated labels) — the article notes not all cell types can be matched to an ontology term.
- You need metrics reflecting tissue/protocol imbalance — the article warns plain macro F1 may not properly reflect the complexity of unbalanced datasets.

## Inputs

- Predicted cell type labels or class probabilities per test cell (e.g., from TabnetClassifier inference)
- True author/annotator cell type labels on the donor-held-out test split (test fraction 0.15, ~3.45M cells)
- Cell Ontology (CL) hierarchy / child-node lookup matrix (04_create_hierarchy_matrices.ipynb, Cell Ontology from https://www.ebi.ac.uk/ols/ontologies/cl)

## Outputs

- Per-cell correctness flags (exact match or subtype-of-true-label)
- Macro F1-score per model or per protocol group (e.g., macro_f1_table.csv, protocol_macro_f1_scores.csv, augmentation_ablation_metrics.csv)

## How to apply

Build (or load) the child-node lookup matrix from the Cell Ontology (CL) using notebooks/04_create_hierarchy_matrices.ipynb, mapping each cell type term to its ontology subtypes. For each held-out test cell, fetch the predicted label and the true author label; mark the prediction correct if it equals the true label or appears among the true label's child nodes in the lookup matrix — predicting a parent type or an unrelated type is wrong. Compute per-cell-type F1 and average equally across cell types (macro F1) to handle strong class imbalance, as in classification-tabnet.ipynb. Apply this rule identically to all compared models (scTab/TabNet, linear, XGBoost, MLP, CellTypist, scGPT, CIForm) so macro F1 values like scTab 0.8295±0.0007 vs linear 0.7848±0.0001 remain comparable.

## Related tools

- **scTab** (Cell type classifier whose TabNet-based predictions are scored with the ontology-aware rule) — https://github.com/theislab/scTab
- **cellxgene-census** (Provides the CELLxGENE census corpus (v2023-05-15) with author cell type annotations used as ground truth)
- **NumPy** (sf_normalize preprocessing and lookup-matrix arithmetic for ontology-aware scoring)
- **INCATools/ubergraph** (Integrated OBO ontology triplestore including Cell Ontology, usable to retrieve subtype relations) — https://github.com/INCATools/ubergraph

## Examples

```
pred_ok = (y_pred == y_true) | child_lookup[y_true, y_pred].astype(bool); macro_f1 = f1_score(y_true, y_pred, labels=label_ids, average='macro')
```

## Evaluation signals

- Reproduced macro F1 falls within reported uncertainty for each model: scTab 0.8295±0.0007, linear 0.7848±0.0001, XGBoost 0.8127±0.0005, MLP 0.7971±0.0012, CellTypist 0.7304±0.0015, scGPT zero-shot 0.7301±0.0035, CIForm 0.766.
- Spot-check correctness flags: a prediction equal to a Cell Ontology child node of the true label must be marked correct; a parent-type prediction must be marked wrong.
- Holdout 10X protocol group reaches macro F1 ~0.8 while non-10X groups (STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA) score ~0.4 under the same scoring rule (Supp. Fig. 7b).
- Augmentation ablation reproduces macro F1 increase from 0.7755±0.0020 to 0.7841±0.0030 (p=0.0016) with the same scoring rule.
- Macro F1 is computed as the mean of per-cell-type F1 with equal weight, so every one of the 164 cell types contributes equally regardless of cell count.

## Limitations

- Cell Ontology incompleteness: not all cell types (especially rare ones) match a corresponding ontology term, which distorts scoring.
- Cell Ontology cell type relationships are still under active discussion, which can affect the classification metrics computed with this rule.
- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets imbalanced across cell types, tissues, and sequencing protocols.
- Annotation granularity inconsistency across CELLxGENE datasets cannot be fixed by the scoring rule or model architecture; needs data curation.

## Evidence

- [methods] Create child node lookup matrix based on cell type ontology: "4. `04_create_hierarchy_matrices.ipynb`: Create child node lookup matrix to find subtypes based on cell type 
  ontology"
- [other] Ontology-aware correctness rule: [evidence span withheld — anchor status: not_found]
- [discussion] Macro F1 for imbalance: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types"
- [discussion] Ontology incompleteness: "not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types"
- [abstract] Benchmark values under the rule: [evidence span withheld — anchor status: not_found]
