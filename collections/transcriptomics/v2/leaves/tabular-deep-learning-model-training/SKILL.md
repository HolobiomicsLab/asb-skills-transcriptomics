---
name: tabular-deep-learning-model-training
description: Use when training or reproducing deep tabular models (scTab/TabNet) for
  cross-tissue cell type classification on the CELLxGENE census corpus, or benchmarking
  them against linear/XGBoost/MLP baselines.
when_to_use_negative:
- When a pretrained scTab checkpoint suffices — pre-trained models are available and
  retraining on the 164GB corpus is expensive
- When labels are not ontology-consistent subtypes of 'native cell' — Cell Ontology
  child-node scoring will be unreliable, especially for rare cell types
- For single-dataset (non-cross-tissue) annotation where a linear baseline performs
  comparably and is cheaper to fit
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTab
  role: Training pipeline and cellnet model code wrapping the TabNet classifier for
    cross-tissue cell type annotation
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular deep-learning architecture with feature attention (1.5-entmax) used
    as scTab's backbone
  repo: https://github.com/theislab/scTab
- name: NVIDIA Merlin dataloader
  role: GPU-optimized on-disk dataloader feeding the >22M-cell training store to PyTorch
  repo: https://github.com/NVIDIA-Merlin/dataloader
- name: XGBoost
  role: Baseline classifier trained with multi:softprob on 256 PCA components of the
    normalized training data
- name: cellxgene-census
  role: Standardised API used to subset and download the CxG census data underlying
    the training store
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:531775bddba9553acdb972fc493692bcc86ca48fab9598e93036b31b9b5ccb91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - CIForm
  - CellTypist
  - NVIDIA Merlin dataloader
  - NumPy
  - TabNet
  - XGBoost
  - cellxgene-census
  - scGPT
  - scTab
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# tabular-deep-learning-model-training

## Summary

Train scTab, a TabNet-based classifier optimized for tabular single-cell RNA-seq data, on the 22.2M-cell CELLxGENE census corpus (19,331 protein-coding genes, 164 cell types) to produce cross-tissue cell type predictions scored by macro F1.

## When to use

Use when training or reproducing deep tabular models (scTab/TabNet) for cross-tissue cell type classification on the CELLxGENE census corpus, or benchmarking them against linear/XGBoost/MLP baselines.

## When NOT to use

- When a pretrained scTab checkpoint suffices — pre-trained models are available and retraining on the 164GB corpus is expensive
- When labels are not ontology-consistent subtypes of 'native cell' — Cell Ontology child-node scoring will be unreliable, especially for rare cell types
- For single-dataset (non-cross-tissue) annotation where a linear baseline performs comparably and is cheaper to fit

## Inputs

- Merlin-format on-disk data store of the CELLxGENE census (v2023-05-15), 22,189,056 cells x 19,331 protein-coding genes, sf-normalized to 10,000 counts and log1p-transformed
- Gene feature list (notebooks/store_creation/features.parquet)
- Donor-based train/val/test split (0.7/0.15/0.15)
- Child-node lookup matrix from the Cell Ontology for subtype-aware scoring

## Outputs

- Trained scTab/TabNet model checkpoint (class-probability outputs over 164 cell types)
- Macro F1-score table across models (e.g. scTab 0.8295±0.0007, XGBoost 0.8127±0.0005, MLP 0.7971±0.0012, linear 0.7848±0.0001)
- Per-configuration result CSVs (e.g. scTab_model_scaling_results.csv, macro_f1_table.csv)

## How to apply

Load the preprocessed Merlin-format store (22,189,056 cells x 19,331 protein-coding genes; each cell size-factor normalized to 10,000 counts then log1p-transformed) using the Nvidia Merlin dataloader, which reads data directly into GPU memory for 0-copy transfer to PyTorch via dlpack. Instantiate the TabNet architecture (feature transformer blocks with batch norm, skip connections, and GLU activations; feature attention masks computed with 1.5-entmax instead of sparsemax) via the scTab training pipeline, using donor-based train/val/test splits of 0.7/0.15/0.15 so that held-out donors prevent donor leakage. Train with cross-entropy loss weighted inversely by class frequency to handle the strong cell-type imbalance, optionally applying the scTab data augmentation scheme, which improves generalization. For the XGBoost baseline, train with the multi:softprob objective on 256 PCA components fit on training data only. Evaluate on the held-out-donor test set with macro F1-score, counting a prediction as correct if it matches the true label or any of its Cell Ontology child nodes.

## Related tools

- **scTab** (Training pipeline and cellnet model code wrapping the TabNet classifier for cross-tissue cell type annotation) — https://github.com/theislab/scTab
- **TabNet** (Tabular deep-learning architecture with feature attention (1.5-entmax) used as scTab's backbone) — https://github.com/theislab/scTab
- **NVIDIA Merlin dataloader** (GPU-optimized on-disk dataloader feeding the >22M-cell training store to PyTorch) — https://github.com/NVIDIA-Merlin/dataloader
- **XGBoost** (Baseline classifier trained with multi:softprob on 256 PCA components of the normalized training data)
- **cellxgene-census** (Standardised API used to subset and download the CxG census data underlying the training store)

## Examples

```
pip install -e . && python notebooks/training/train.py  # cellnet TabNet training on the merlin_cxg_2023_05_15_sf-log1p store
```

## Evaluation signals

- Macro F1-score on the held-out-donor test set matches reported targets, e.g. scTab with augmentation at 0.8295±0.0007 and the largest 16.2M-parameter model at 0.8323±0.0010
- scTab over linear baseline gap at full training data is 0.0447±0.0008 macro F1
- Model forward pass on a dummy batch yields class-probability outputs of correct shape (164 classes) and the inverse-frequency-weighted cross-entropy loss computes without error
- Scaling curves show strong macro F1 improvement under donor-based subsampling (minimum 2.1M→15.2M-cell difference of 0.0454±0.0083) but only weak improvement under cell-based subsampling
- Predictions are scored correct when equal to the true label or any Cell Ontology child node, not by exact-match accuracy

## Limitations

- Plain macro F1 may not reflect complexity of datasets imbalanced across cell types, tissues, and sequencing protocols
- Cell Ontology incompleteness means not all cell types (especially rare ones) match an ontology term, and its relationships are still under discussion, affecting classification metrics
- The current benchmark excludes uncommon assay technologies and finer cell subtypes due to data availability; pre-trained scTab models still contain some fairly coarsely annotated cell types
- Training the full corpus requires large-scale GPU infrastructure (164GB training store, CUDA-capable environment)

## Evidence

- [intro] scTab abstract TabNet choice: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing normalization: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [readme] readme training data availability: "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
- [readme] readme package install: "Run the following command the project folder to install the ``cellnet`` package:
``pip install -e .``"
- [readme] readme dataloader benefit: [evidence span withheld — anchor status: external_doc]
