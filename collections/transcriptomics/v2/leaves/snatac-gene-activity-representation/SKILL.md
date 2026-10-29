---
name: snatac-gene-activity-representation
description: 'Use when classifying snATAC-seq nuclei with Pollock: convert accessibility
  to gene activity, then train or apply the snATACseq_generalized or disease-specific
  BRCA/CCRCC/GBM modules.'
when_to_use_negative:
- Input is a peak-by-cell or fragment-file snATAC-seq matrix that has not been converted
  to gene activity — Pollock's snATAC-seq modules expect the gene activity representation
- Input is scRNA-seq or snRNA-seq — use the corresponding scRNAseq_* or snRNAseq_*
  pretrained modules instead of the snATAC-seq gene activity modules
- The goal is a disease-specific model for BRCA, CCRCC or GBM with no cross-cancer
  generalization benchmark — training a merged generalized module adds unnecessary
  complexity
tools:
- name: Pollock
  role: Variational autoencoder with linear classification head used in train and
    predict modes on snATAC-seq gene activity data
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Backend for training the VAE classification module (installed via conda in
    the pollock environment)
- name: scanpy
  role: Handles .h5ad gene activity inputs and outputs for from_scanpy predict/train
    runs
- name: Conda
  role: Environment manager for installing Pollock and its PyTorch/scanpy dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:ac733f15aaf5780c1d41a50be255a97c7dd462a6d5a1945f69360ef67868d6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - Seurat
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# snatac-gene-activity-representation

## Summary

Represents single-nucleus ATAC-seq data as a gene activity matrix so Pollock's VAE classifiers can predict cell types in the same feature space as scRNA-seq, enabling training of generalized pan-cancer snATAC-seq modules (BRCA, CCRCC, GBM) and benchmarking them against disease-specific models.

## When to use

Use when classifying snATAC-seq nuclei with Pollock: convert accessibility to gene activity, then train or apply the snATACseq_generalized or disease-specific BRCA/CCRCC/GBM modules.

## When NOT to use

- Input is a peak-by-cell or fragment-file snATAC-seq matrix that has not been converted to gene activity — Pollock's snATAC-seq modules expect the gene activity representation
- Input is scRNA-seq or snRNA-seq — use the corresponding scRNAseq_* or snRNAseq_* pretrained modules instead of the snATAC-seq gene activity modules
- The goal is a disease-specific model for BRCA, CCRCC or GBM with no cross-cancer generalization benchmark — training a merged generalized module adds unnecessary complexity

## Inputs

- snATAC-seq gene activity matrix (gene-by-cell), provided as Seurat .RDS or scanpy .h5ad with raw counts
- Cell type labels under a metadata/.obs key (e.g. 'cell_type') for train mode
- pretrained_models.tar.gz and benchmarking_datasets.tar.gz (Zenodo record 5895221)

## Outputs

- Trained snATACseq_generalized module directory
- output.rds / output.h5ad with predicted cell types and UMAP of VAE embeddings, or a tab-separated txt file via --txt-output
- generalized_vs_disease_specific_f1.csv of per-cancer-type macro-F1 deltas

## How to apply

First represent snATAC-seq as gene activity — Pollock's pretrained snATAC-seq modules expect gene activity, not a peak-by-cell matrix, as noted in the README's module list. To train a generalized module, download benchmarking_datasets.tar.gz and pretrained_models.tar.gz from Zenodo record 5895221, merge all disease-specific snATAC-seq training datasets into one training set, and remove normal epithelial cells from the combined data. Train with the Pollock CLI in train mode using the default hyperparameters: 20 epochs, latent-dim 64, encoder-out-dim 128, middle-dim 512, n-per-cell-type 500, and kl-scaler 1e-3 (higher kl-scaler gives noisier but more generalizable embeddings). Run predict mode on each cancer type's validation set, writing predictions either back into the input object or via --txt-output, then compute macro-averaged F1 per cancer type and subtract the disease-specific module's F1 to obtain the generalization delta.

## Related tools

- **Pollock** (Variational autoencoder with linear classification head used in train and predict modes on snATAC-seq gene activity data) — https://github.com/ding-lab/pollock
- **PyTorch** (Backend for training the VAE classification module (installed via conda in the pollock environment))
- **scanpy** (Handles .h5ad gene activity inputs and outputs for from_scanpy predict/train runs)
- **Conda** (Environment manager for installing Pollock and its PyTorch/scanpy dependencies)

## Examples

```
pollock predict from_scanpy --module-filepath snATACseq_generalized --scanpy-h5ad-filepath gene_activity.h5ad --output-prefix output --txt-output
```

## Evaluation signals

- SnATAC-seq validation matrices must overlap the model's genes after filtering (logged as 'N genes overlap with model after filtering', e.g. 12679 overlapping genes in the PBMC example)
- Reproduced F1 deltas fall within the reported spread: 8 of 12 cancer-type comparisons within 0.30 F1; for snATAC-seq, BRCA delta 0.02 vs GBM delta 0.43
- Predictions are written back into the output object (output.rds/output.h5ad) or a tab-separated txt file with one prediction per cell
- Training completes with the default hyperparameters confirmed (epochs=20, latent-dim=64, encoder-out-dim=128, middle-dim=512, n-per-cell-type=500, kl-scaler=1e-3)
- Combined training set contains no normal epithelial cells before fitting the generalized module

## Limitations

- The GBM snATAC-seq comparison generalizes most poorly, with an F1 difference of 0.43 (the largest of any modality alongside its 0.52 snRNA-seq delta), so generalized snATAC-seq predictions on GBM should be treated cautiously
- The README's pretrained module list appears inconsistent for snATAC-seq, listing 'snATACseq_brca' twice, the second entry described as 'trained with disease-specific glioblastoma cancer dataset' — verify module identities from the downloaded archive
- kl-scaler trades off noise versus generalizability: increasing it yields noisier cell embeddings that are typically more generalizable, so default 1e-3 may under- or over-regularize for other ATAC-seq datasets

## Evidence

- [readme] snATACseq_brca (trained with disease-specific breast cancer dataset, snATAC-seq data is represented as gene activity): "* snATACseq_brca (trained with disease-specific breast cancer dataset, snATAC-seq data is represented as gene activity)"
- [readme] This parameter controls how regularized the VAE is... typically more generalizable: "This parameter controls how regularized the VAE is. 1e-3 is the default. If increased the cell embeddings are typically more noisy, but typically moregeneralizable."
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [results] 12679 genes overlap with model after filtering: "2022-01-24 09:47:13,449 12679 genes overlap with model after filtering
2022-01-24 09:47:13,450 4859 genes missing from dataset after filtering"
