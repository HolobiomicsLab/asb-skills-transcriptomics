---
name: cell-type-classification-macro-f1-evaluation
description: Use when comparing cell type annotation models on a held-out test set
  and you need donor-generalizing performance metrics such as macro F1-score and cross-entropy
  loss.
when_to_use_negative:
- Evaluating on a random (cell-level) split where the same donors appear in train
  and test — this allows donor-level label leakage and inflates scores relative to
  the unseen-donor protocol
- Datasets with cell types present in fewer than 30 donors or fewer than 5000 cells,
  where per-cell-type F1 and donor-generalization cannot be reliably quantified
- Single-tissue, single-donor benchmarks where cross-tissue macro F1 is not the target
  of comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Automated cell type prediction model producing class probabilities on tabular
    scRNA-seq data; its predictions are scored with macro F1 and cross-entropy loss
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular-data architecture underlying scTab; trained with cross-entropy loss
    and evaluated via class-probability outputs
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Used in the sf_normalize routine for size-factor normalization (10,000 counts
    + log1p) that prepares cells before scoring
- name: cellxgene-census
  role: Provides the standardized CELLxGENE census API used to assemble the labeled
    corpus whose test split is evaluated
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:c2ff959cb699446ef300cfba56d3103b3e035f599f0e08c2b47fd835b4368587
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-classification-macro-f1-evaluation

## Summary

Evaluates cross-tissue cell type classifiers (e.g., scTab/TabNet vs. a linear reference) on an unseen-donor test set using macro F1-score (per-cell-type F1 averaged over cell types) and cross-entropy loss, following the scTab benchmark protocol on the CELLxGENE-derived corpus.

## When to use

Use when comparing cell type annotation models on a held-out test set and you need donor-generalizing performance metrics such as macro F1-score and cross-entropy loss.

## When NOT to use

- Evaluating on a random (cell-level) split where the same donors appear in train and test — this allows donor-level label leakage and inflates scores relative to the unseen-donor protocol
- Datasets with cell types present in fewer than 30 donors or fewer than 5000 cells, where per-cell-type F1 and donor-generalization cannot be reliably quantified
- Single-tissue, single-donor benchmarks where cross-tissue macro F1 is not the target of comparison

## Inputs

- Trained classifier (e.g., scTab TabNet with 19,331 input genes and class-probability output)
- Donor-held-out test split of the scTab corpus (Merlin-format store, 22,189,056 cells x 19,331 protein-coding genes, sf-normalized to 10,000 counts + log1p)
- Ground-truth cell type labels for test cells (Cell Ontology-mapped)
- Trained linear reference model predictions for baseline comparison

## Outputs

- Macro F1-score per model/training-size condition (F1 per cell type averaged over cell types)
- Cross-entropy loss per model/training-size condition
- Scaling tables of macro F1 and loss vs. training set size under donor-based vs. cell-based subsampling
- scTab-over-linear reference performance gap (expected ~0.0447 ± 0.0008 macro F1 at full data)

## How to apply

Hold out the donor-disjoint val (0.15) and test (0.15) splits fixed while the train split is 0.7 of the full 22.2M-cell corpus, so every model is scored on cells from unseen donors. Feed the test cells (size-factor normalized to 10,000 counts and log1p-transformed, 19,331 protein-coding genes) through the trained classifier to obtain class probabilities, then compute cross-entropy loss against the true cell type labels and the macro F1-score, i.e., the F1 per cell type averaged over cell types. Tabulate these metrics versus training set size or model size when running scaling comparisons, and benchmark against a well-tuned linear reference model trained on the same data. Correctness is judged by reproducing expected gaps: at full training data scTab outperformed the linear reference by 0.0447 ± 0.0008 macro F1, and donor-based scaling from 2.1M to 15.2M cells yielded a minimum organ-system macro F1 difference of 0.0454 ± 0.0083 (median 0.1219 ± 0.0212). Note that the corpus is filtered so each cell type has ≥5000 cells and is observed in ≥30 donors, which is what makes donor-level generalization measurable.

## Related tools

- **scTab** (Automated cell type prediction model producing class probabilities on tabular scRNA-seq data; its predictions are scored with macro F1 and cross-entropy loss) — https://github.com/theislab/scTab
- **TabNet** (Tabular-data architecture underlying scTab; trained with cross-entropy loss and evaluated via class-probability outputs) — https://github.com/theislab/scTab
- **NumPy** (Used in the sf_normalize routine for size-factor normalization (10,000 counts + log1p) that prepares cells before scoring)
- **cellxgene-census** (Provides the standardized CELLxGENE census API used to assemble the labeled corpus whose test split is evaluated) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html

## Examples

```
from cellnet.utils.evaluation import compute_metrics; scores = compute_metrics(y_true=test_labels, y_pred=model.predict(merlin_loader))
```

## Evaluation signals

- At full training data, scTab exceeds the linear reference model by 0.0447 ± 0.0008 macro F1-score
- Donor-based scaling from 2.1M to 15.2M training cells yields a minimum organ-system macro F1 difference of 0.0454 ± 0.0083 (median 0.1219 ± 0.0212)
- Macro F1 improves strongly with training size under donor-based subsampling but only weakly under cell-based subsampling
- Test-set cell types each have ≥5000 cells and are observed in ≥30 donors in the filtered corpus
- Loss curves and macro F1 tabulated per training fraction (15%, 30%, 50%, 70%, 100% of donors) on the fixed unseen-donor test split

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets, which are imbalanced not only by cell type but also by tissue and sequencing protocol distribution
- Cell Ontology incompleteness means not all cell types (especially rare ones) match an ontology term, and ongoing discussion of CL relationships affects the classification metrics
- Current benchmark excludes uncommon assay technologies and finer cell subtypes due to data availability

## Evidence

- [results] macro F1 definition and donor scaling gap: [evidence span withheld — anchor status: not_found]
- [methods] unseen-donor splits and training fraction: [evidence span withheld — anchor status: not_found]
- [methods] donor count requirement enabling generalization measurement: "Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors."
- [discussion] limitation of plain macro F1 on imbalanced data: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types"
- [methods] preprocessing before scoring: "Normalize each cell to have `10000` counts (size factor normalization)"
