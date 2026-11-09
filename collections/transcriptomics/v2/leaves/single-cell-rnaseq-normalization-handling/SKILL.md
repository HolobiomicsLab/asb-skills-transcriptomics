---
name: single-cell-rnaseq-normalization-handling
description: Use when raw single-cell RNA-seq count matrices must be converted to
  a comparable feature space (sf-log1p) before training or evaluating cell type classifiers
  such as scTab, TabNet, XGBoost, MLP, or linear baselines.
when_to_use_negative:
- Input is already preprocessed sf-log1p data, such as the released merlin_cxg_2023_05_15_sf-log1p
  store — re-normalizing would double-transform the values
- Analysis requires raw counts or alternative normalization (e.g. scran, SCTransform)
  or integration of non-transcriptomic modalities, which scTab does not yet cover
- Working with assays outside the corpus' sequencing protocols or gene spaces other
  than the 19,331 protein-coding gene subset
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NumPy
  role: Implements the sf_normalize size-factor normalization and log1p transformation
    on the cell-by-gene matrix
- name: scTab
  role: Training pipeline whose data-loading infrastructure consumes the sf-log1p
    normalized on-disk store
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: API used to download the raw CELLxGENE census data prior to normalization
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- name: NVIDIA-Merlin/dataloader
  role: Reads the normalized sf-log1p data from an on-disk Merlin-format store for
    GPU training
  repo: https://github.com/NVIDIA-Merlin/dataloader
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:e06e042d5b5290e7476478c5bda6992800ee70087c13d946da2ff49336efd2f1
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
  - cellxgene-census
  - scGPT
  - scTab
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rnaseq-normalization-handling

## Summary

Size-factor normalize each single-cell RNA-seq cell to 10,000 counts followed by log1p transformation, as used to preprocess the 22,189,056-cell CELLxGENE corpus before training and benchmarking scTab and all reference classifiers.

## When to use

Use when raw single-cell RNA-seq count matrices must be converted to a comparable feature space (sf-log1p) before training or evaluating cell type classifiers such as scTab, TabNet, XGBoost, MLP, or linear baselines.

## When NOT to use

- Input is already preprocessed sf-log1p data, such as the released merlin_cxg_2023_05_15_sf-log1p store — re-normalizing would double-transform the values
- Analysis requires raw counts or alternative normalization (e.g. scran, SCTransform) or integration of non-transcriptomic modalities, which scTab does not yet cover
- Working with assays outside the corpus' sequencing protocols or gene spaces other than the 19,331 protein-coding gene subset

## Inputs

- Raw cell-by-gene count matrix from CELLxGENE census version 2023-05-15 (22,189,056 cells x 19,331 protein-coding genes)
- Gene feature list (features.parquet) defining the 19,331 protein-coding gene space
- Donor-based train/val/test split assignments (0.7/0.15/0.15)

## Outputs

- Size-factor normalized (10,000 counts) and log1p-transformed cell-by-gene matrix
- On-disk Merlin-format data store compatible with the Nvidia Merlin dataloader (e.g. merlin_cxg_2023_05_15_sf-log1p.tar.gz, 164GB)
- Normalized feature matrix suitable for PCA (256 components) and classifier training

## How to apply

Apply two sequential steps to every cell in the corpus: (1) size factor normalization so each cell sums to exactly 10,000 counts, and (2) a log1p transformation of the normalized values. In the scTab pipeline this is implemented with NumPy via an sf_normalize function on the cell-by-gene matrix (19,331 protein-coding genes), and the normalized data are then written to an on-disk Merlin-format store (merlin_cxg_2023_05_15_sf-log1p) readable by the Nvidia Merlin dataloader. Perform normalization before any PCA computation (256 components for XGBoost training, 50 for visualization), fitting PCA only on normalized training data, and apply the identical transform to train, validation, and donor-held-out test splits so feature scales remain consistent across models.

## Related tools

- **NumPy** (Implements the sf_normalize size-factor normalization and log1p transformation on the cell-by-gene matrix)
- **scTab** (Training pipeline whose data-loading infrastructure consumes the sf-log1p normalized on-disk store) — https://github.com/theislab/scTab
- **cellxgene-census** (API used to download the raw CELLxGENE census data prior to normalization) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- **NVIDIA-Merlin/dataloader** (Reads the normalized sf-log1p data from an on-disk Merlin-format store for GPU training) — https://github.com/NVIDIA-Merlin/dataloader

## Examples

```
import numpy as np; X_norm = np.log1p(X / X.sum(axis=1, keepdims=True) * 10000)
```

## Evaluation signals

- Each normalized cell's counts sum to exactly 10,000 before the log1p step
- The output store is loadable by merlin.io.Dataset / Nvidia Merlin dataloader with the expected 22,189,056 cells x 19,331 genes dimensions
- Downstream benchmarks reproduce the published macro F1-scores on the sf-log1p features (scTab 0.8295 ± 0.0007, linear 0.7848 ± 0.0001)
- PCA fitted on normalized training data yields 256 components that support XGBoost macro F1 of 0.8127 ± 0.0005
- Identical transform is verified for train, validation, and test splits (no split-specific scaling)

## Limitations

- Normalization is applied only to transcriptomic counts; other modalities are not incorporated at scale in the scTab framework
- Plain normalization does not fix annotation-granularity inconsistencies or coarse cell type labels present in CELLxGENE datasets
- The sf-log1p scheme was validated on the filtered corpus (primary data, ≥5000 cells and ≥30 donors per cell type); behavior on less stringent filters or uncommon assay technologies is untested

## Evidence

- [methods] Size factor normalization to 10000 counts: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] Log1p transformation: "2. After size factor normalization the data is Log1p transformed"
- [methods] NumPy sf_normalize implementation: "import numpy as np 
 def sf_normalize(X):"
- [readme] On-disk Merlin store for normalized data: "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
- [readme] Merlin dataloader purpose: "Save data into on-disk format that can be used by Nvidia Merlin dataloader"
