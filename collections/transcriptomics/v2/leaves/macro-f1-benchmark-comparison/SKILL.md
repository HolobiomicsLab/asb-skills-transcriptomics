---
name: macro-f1-benchmark-comparison
description: Use when benchmarking cell type classifiers on a large cross-tissue scRNA-seq
  corpus and you need macro F1-scores comparable to scTab (0.8295±0.0007), linear
  (0.7848±0.0001), XGBoost (0.8127±0.0005), and MLP (0.7971±0.0012).
when_to_use_negative:
- Your goal is organ-specific (tissue-restricted) annotation rather than cross-tissue
  generalization — the article notes cross-tissue vs. organ-specific comparison is
  a distinct question
- Your dataset violates the benchmark filters (non-primary data, <5000 cells per cell
  type, cell types observed in <30 donors, or cell types with <7 ontology parent nodes),
  making macro F1-scores incomparable
- You need per-cell-type accuracy or balanced metrics tailored to heavily imbalanced
  tissue/protocol distributions — the article warns plain macro F1 may not reflect
  such complexity
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scTab
  role: TabNet-based cell type classifier trained with the proposed data augmentation
    scheme; the top-performing model in the comparison
  repo: https://github.com/theislab/scTab
- name: XGBoost
  role: Reference classifier trained with multi:softprob objective on 256 PCA components
    of the normalized training data
- name: CellTypist
  role: External baseline classifier benchmarked on the same corpus (macro F1 0.7304±0.0015)
- name: scGPT
  role: Transformer baseline evaluated zero-shot (0.7301±0.0035) and fine-tuned (0.749)
- name: CIForm
  role: Transformer-based annotation baseline benchmarked for comparison (macro F1
    0.766)
- name: cellxgene-census
  role: Python API used to query and subset the CELLxGENE census (version 2023-05-15)
    for the benchmark corpus
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- name: NVIDIA-Merlin/dataloader
  role: Provides the GPU-optimized on-disk data loading infrastructure used to feed
    normalized training data to scTab
  repo: https://github.com/NVIDIA-Merlin/dataloader
- name: NumPy
  role: Implements the size-factor normalization (sf_normalize) step of preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:13d81d11e6259296ee116fe3a4234e4cc652c6447d81069efb0508dd8246b768
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - CIForm
  - CellTypist
  - NVIDIA-Merlin/dataloader
  - NumPy
  - TabNet
  - XGBoost
  - cellxgene-census
  - scGPT
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# macro-f1-benchmark-comparison

## Summary

Reproduces the cross-tissue cell type classification benchmark from scTab, training and evaluating scTab, a linear model, MLP, XGBoost, CellTypist, scGPT, and CIForm on the CELLxGENE census corpus (22,189,056 cells, 164 cell types, 19,331 protein-coding genes) and compiling macro F1-scores into a comparison table.

## When to use

Use when benchmarking cell type classifiers on a large cross-tissue scRNA-seq corpus and you need macro F1-scores comparable to scTab (0.8295±0.0007), linear (0.7848±0.0001), XGBoost (0.8127±0.0005), and MLP (0.7971±0.0012).

## When NOT to use

- Your goal is organ-specific (tissue-restricted) annotation rather than cross-tissue generalization — the article notes cross-tissue vs. organ-specific comparison is a distinct question
- Your dataset violates the benchmark filters (non-primary data, <5000 cells per cell type, cell types observed in <30 donors, or cell types with <7 ontology parent nodes), making macro F1-scores incomparable
- You need per-cell-type accuracy or balanced metrics tailored to heavily imbalanced tissue/protocol distributions — the article warns plain macro F1 may not reflect such complexity

## Inputs

- CELLxGENE census (version 2023-05-15) Cell-by-Gene matrix subset to 19,331 protein-coding genes and 164 cell types (22,189,056 cells)
- Donor-based train/val/test split file (fractions 0.7/0.15/0.15)
- Cell Ontology child node lookup matrix (from notebooks/04_create_hierarchy_matrices.ipynb)
- Normalized count matrix (size-factor 10,000 + log1p) or on-disk Merlin dataloader store

## Outputs

- macro_f1_table.csv with mean ± std macro F1-score per model (scTab, linear, XGBoost, MLP, CellTypist, scGPT, CIForm)
- Trained classifier checkpoints (e.g., scTab TabNet weights)

## How to apply

Load the benchmark corpus derived from CELLxGENE census version 2023-05-15 using the donor-based train/val/test split with fractions 0.7/0.15/0.15. Preprocess each cell by size-factor normalization to 10,000 counts followed by log1p transformation. Train scTab (TabNet architecture with the proposed data augmentation scheme) using cross-entropy loss; train the linear (single fully connected layer) and MLP reference classifiers under the same normalization; train XGBoost with the multi:softprob objective on 256 PCA components computed from normalized training data only (PCA fit on training data to avoid leakage). Evaluate all classifiers on the held-out-donor test set using macro F1-score, scoring a prediction as correct if it matches the true label or any of its Cell Ontology child nodes via the child node lookup matrix. Compile the macro F1 table (targets: scTab 0.8295±0.0007, linear 0.7848±0.0001, XGBoost 0.8127±0.0005, MLP 0.7971±0.0012, CellTypist 0.7304±0.0015, scGPT zero-shot 0.7301±0.0035, scGPT fine-tuned 0.749, CIForm 0.766) and save as macro_f1_table.csv.

## Related tools

- **scTab** (TabNet-based cell type classifier trained with the proposed data augmentation scheme; the top-performing model in the comparison) — https://github.com/theislab/scTab
- **XGBoost** (Reference classifier trained with multi:softprob objective on 256 PCA components of the normalized training data)
- **CellTypist** (External baseline classifier benchmarked on the same corpus (macro F1 0.7304±0.0015))
- **scGPT** (Transformer baseline evaluated zero-shot (0.7301±0.0035) and fine-tuned (0.749))
- **CIForm** (Transformer-based annotation baseline benchmarked for comparison (macro F1 0.766))
- **cellxgene-census** (Python API used to query and subset the CELLxGENE census (version 2023-05-15) for the benchmark corpus) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- **NVIDIA-Merlin/dataloader** (Provides the GPU-optimized on-disk data loading infrastructure used to feed normalized training data to scTab) — https://github.com/NVIDIA-Merlin/dataloader
- **NumPy** (Implements the size-factor normalization (sf_normalize) step of preprocessing)

## Examples

```
pip install -e . && python notebooks/store_creation/01_download_data.ipynb  # then train and evaluate scTab/linear/MLP/XGBoost on the census corpus and write macro_f1_table.csv
```

## Evaluation signals

- Reproduced macro F1-scores fall within the reported confidence intervals: scTab 0.8295±0.0007, linear 0.7848±0.0001, XGBoost 0.8127±0.0005, MLP 0.7971±0.0012
- Nonlinear models (scTab, XGBoost, MLP) each exceed the optimized linear model's 0.7848, confirming the paper's finding that cross-tissue annotation requires nonlinear models
- PCA for XGBoost is fit on training data only; any donor-level label leakage between train and test sets inflates scores (corpus filters enforce is_primary_data == True)
- Scoring counts a prediction as correct if it equals the true label or any Cell Ontology child node — a plain exact-match macro F1 will systematically read lower
- Test-set evaluation is restricted to held-out donors, not merely held-out cells

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of datasets imbalanced across cell types, tissues, and sequencing protocols; more detailed evaluation metrics may be needed
- Cell Ontology incompleteness means not all cell types (especially rare ones) match an ontology term, and ontology relationships are still under active discussion, which can affect classification metrics
- Annotation granularity inconsistency across CELLxGENE datasets cannot be fixed by model architecture and requires further data curation
- The benchmark excludes uncommon assay technologies and finer cell subtypes due to data availability filters

## Evidence

- [other] scTab fitted with data augmentation achieves a macro F1-score of 0.8295 ± 0.0007, XGBoost 0.8127 ± 0.0005, and MLP 0.7971 ± 0.0012, versus 0.7848 ± 0.0001 for the optimized linear model: [evidence span withheld — anchor status: not_found]
- [methods] Normalize each cell to have 10000 counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] Create child node lookup matrix to find subtypes based on cell type ontology: "4. `04_create_hierarchy_matrices.ipynb`: Create child node lookup matrix to find subtypes based on cell type 
  ontology"
- [methods] Each cell type has to be observed in at least 30 donors to reliably quantify whether the classifier can generalize to new donors: "5. Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can 
  generalize to new donors."
- [discussion] plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types"
- [abstract] we propose scTab, an automated cell type prediction model specific to tabular data, and train it using a novel data augmentation scheme: "we propose scTab, an automated cell type prediction model specific to tabular data, and train it using a novel data augmentation scheme"
- [intro] an XGBoost model that reported robust performances on classification tasks for tabular data: "an XGBoost model that reported robust performances on classification tasks for tabular data"
