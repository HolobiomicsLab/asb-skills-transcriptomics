---
name: donor-based-holdout-evaluation
description: Use when benchmarking cell type classifiers on scRNA-seq data where generalization
  to unseen donors must be quantified and donor-level label leakage between train
  and test sets must be prevented.
when_to_use_negative:
- Cell types with fewer than 30 donors in the corpus — donor-level generalization
  cannot be reliably quantified per the paper's filter criteria
- Datasets where cells from the same donor must appear in multiple splits (e.g., within-donor
  stratification is the goal rather than unseen-donor generalization)
- Datasets lacking is_primary_data == True filtering, where label leakage between
  train and test set cannot be ruled out
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: TabNet-based cell type classifier evaluated on the donor-based holdout test
    set under both augmented and non-augmented training conditions
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: Standardized API used to query and download the CELLxGENE census (version
    2023-05-15) from which donor-subsampled train/val/test stores are built
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- name: NumPy
  role: Used for size-factor normalization (sf_normalize) to 10,000 counts and log1p
    transformation prior to evaluation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:fd0c80d8b743244ccca580ae96ed3b83ec180dd69f156d3ccaa6aa7b0f3b2511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - CIForm
  - CellTypist
  - NumPy
  - TabNet
  - cellxgene-census
  - scGPT
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# donor-based-holdout-evaluation

## Summary

Evaluates cell type classifiers on a held-out-donor test set with donor-based 70/15/15 train/val/test splits, ensuring each donor appears in exactly one split, and scores predictions as correct if they match the annotator label or any of its Cell Ontology child nodes using macro F1-score.

## When to use

Use when benchmarking cell type classifiers on scRNA-seq data where generalization to unseen donors must be quantified and donor-level label leakage between train and test sets must be prevented.

## When NOT to use

- Cell types with fewer than 30 donors in the corpus — donor-level generalization cannot be reliably quantified per the paper's filter criteria
- Datasets where cells from the same donor must appear in multiple splits (e.g., within-donor stratification is the goal rather than unseen-donor generalization)
- Datasets lacking is_primary_data == True filtering, where label leakage between train and test set cannot be ruled out

## Inputs

- CELLxGENE census version 2023-05-15 Cell-by-Gene (CxG) store subset to 19331 protein-coding genes
- Donor-based train/val/test split metadata (fractions 0.7/0.15/0.15)
- Cell Ontology child-node lookup matrix (hierarchy matrices from cell type ontology)
- Trained classifier predictions on the held-out test set

## Outputs

- Macro F1-score table across models (macro_f1_table.csv)
- Per-seed and aggregated test loss and macro F1 metrics for each condition (augmentation_ablation_metrics.csv)

## How to apply

Load the donor-subsampled CELLxGENE census (version 2023-05-15; 22,189,056 cells, 164 cell types, 19,331 protein-coding genes) store and split it into train, val, and test sets with donor-based fractions 0.7/0.15/0.15 so each donor appears entirely in exactly one split, preventing donor-level leakage. Preprocess each cell with size-factor normalization to 10,000 counts followed by log1p transformation. Evaluate classifiers on the unseen-donor holdout test set, scoring a prediction as correct when it matches the annotator label or predicts a subtype of it according to the Cell Ontology child-node lookup matrix. Compute macro F1-score (per-cell-type F1 averaged) and holdout test loss, repeating training over multiple random seeds to report mean ± standard deviation. For the scTab benchmark, confirm expected values such as scTab with augmentation at 0.8295 ± 0.0007 versus linear at 0.7848 ± 0.0001.

## Related tools

- **scTab** (TabNet-based cell type classifier evaluated on the donor-based holdout test set under both augmented and non-augmented training conditions) — https://github.com/theislab/scTab
- **cellxgene-census** (Standardized API used to query and download the CELLxGENE census (version 2023-05-15) from which donor-subsampled train/val/test stores are built) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- **NumPy** (Used for size-factor normalization (sf_normalize) to 10,000 counts and log1p transformation prior to evaluation)

## Examples

```
pip install -e . && python notebooks/model_evaluation/<eval_notebook>.ipynb --test-set merlin_cxg_2023_05_15_sf-log1p/test --hierarchy child_node_lookup.npz
```

## Evaluation signals

- Verify each donor appears entirely in exactly one of the train/val/test splits (no donor-level leakage)
- Check macro F1-score falls within the reported ranges, e.g. scTab with augmentation 0.8295 ± 0.0007 and linear model 0.7848 ± 0.0001
- Confirm holdout test loss for augmented vs. non-augmented scTab reproduces 0.659 ± 0.04 vs. 0.797 ± 0.05 with p=0.0039
- Confirm predictions are scored as correct when they match the true label or any of its Cell Ontology child nodes via the lookup matrix
- Validate the output CSV contains per-seed and aggregated metrics for both conditions (loss and macro F1 with standard deviations)

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are imbalanced concerning the distribution of cell types, tissues, and sequencing protocols
- Cell Ontology relationships are still under active discussion, which can affect the classification metrics; not all cell types (especially rare ones) match an ontology term
- The benchmark corpus is restricted to relatively common assay technologies and cell types with ≥5000 cells observed in ≥30 donors, excluding uncommon assays and finer subtypes

## Evidence

- [methods] Donor-based split prevents leakage: [evidence span withheld — anchor status: not_found]
- [methods] Child-node correct prediction scoring: [evidence span withheld — anchor status: not_found]
- [methods] Filter criteria for donor reliability: "Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors."
- [results] Benchmark target metrics: [evidence span withheld — anchor status: not_found]
- [discussion] Metric limitation caveat: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types but also concerning the distribution of"
