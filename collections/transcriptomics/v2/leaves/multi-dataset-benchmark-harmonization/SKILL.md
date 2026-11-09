---
name: multi-dataset-benchmark-harmonization
description: Use when a cell-type classifier (e.g. Pollock) trained on merged multi-cancer
  cohorts must be benchmarked against disease-specific models on each cancer type's
  validation set to quantify generalization loss.
when_to_use_negative:
- The goal is fine-grained classification within a single cancer type, where the article
  shows the disease-specific module is the stronger choice (e.g. CESC or GBM, which
  generalize most poorly)
- Training data lacks cell type annotations (no --cell-type-key metadata column),
  so no new generalized module can be trained
- Input expression matrices are already normalized — Pollock requires raw (unnormalized)
  counts in the .X attribute or @assays$RNA@counts
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Train generalized per-modality VAE classifiers and run per-cancer-type prediction;
    installed from GitHub or Docker image
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning backend for the Pollock VAE with linear classification head
- name: scanpy
  role: Handles .h5ad AnnData inputs for merged scRNA-seq training sets and prediction
    outputs
- name: Seurat
  role: Handles .rds object inputs (raw counts in @assays$RNA@counts) for training
    and prediction
- name: Conda
  role: Environment creation for Pollock with pytorch, scanpy, umap-learn and r-base
    dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:441ddb2cb12a6f4be47e11b9d428460eb5955edeed004352e4c037a5b3fb3e7e
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-dataset-benchmark-harmonization

## Summary

Harmonize multiple disease-specific single-cell training cohorts into one generalized training set per data modality (scRNA-seq, snRNA-seq, snATAC-seq as gene activity), train Pollock's generalized pan-cancer VAE classifier, and benchmark its macro-F1 against disease-specific modules per cancer type.

## When to use

Use when a cell-type classifier (e.g. Pollock) trained on merged multi-cancer cohorts must be benchmarked against disease-specific models on each cancer type's validation set to quantify generalization loss.

## When NOT to use

- The goal is fine-grained classification within a single cancer type, where the article shows the disease-specific module is the stronger choice (e.g. CESC or GBM, which generalize most poorly)
- Training data lacks cell type annotations (no --cell-type-key metadata column), so no new generalized module can be trained
- Input expression matrices are already normalized — Pollock requires raw (unnormalized) counts in the .X attribute or @assays$RNA@counts

## Inputs

- benchmarking_datasets.tar.gz (Zenodo record 5895221): per-disease annotated single-cell training datasets
- pretrained_models.tar.gz (Zenodo record 5895221): disease-specific and generalized Pollock modules
- Seurat .rds or scanpy .h5ad objects with raw counts and a cell type label column (e.g. 'cell_type')

## Outputs

- One generalized Pollock module per modality (scRNA-seq, snRNA-seq, snATAC-seq)
- Per-cancer-type prediction tables (saved to output.rds / output.h5ad metadata or tab-separated .txt)
- generalized_vs_disease_specific_f1.csv summarizing per-cancer-type macro-F1 deltas

## How to apply

Download benchmarking_datasets.tar.gz and pretrained_models.tar.gz from Zenodo record 5895221, then for each modality merge all disease-specific training sets into a single per-modality training corpus, removing normal epithelial cells from the combined data. Train a generalized module with the Pollock CLI in train mode using default hyperparameters: 20 epochs, latent-dim 64, encoder-out-dim 128, middle-dim 512, n-per-cell-type 500, kl-scaler 1e-3. Run Pollock in predict mode on each cancer type's validation set with each generalized module, saving predictions either back into the output object (output.rds / output.h5ad) or as a tab-separated txt file. Compute macro-averaged F1 per cancer type for the generalized module and subtract the disease-specific module's F1 to get the generalization delta, writing results to generalized_vs_disease_specific_f1.csv. A successful run reproduces the reported spread: 8 of 12 cancer-type comparisons within 0.30 F1, with PDAC scRNA-seq (0.03) and BRCA snRNA-seq/snATAC-seq (0.06/0.02) generalizing best and CESC scRNA-seq (0.31) and GBM (0.52/0.43) worst.

## Related tools

- **Pollock** (Train generalized per-modality VAE classifiers and run per-cancer-type prediction; installed from GitHub or Docker image) — https://github.com/ding-lab/pollock
- **PyTorch** (Deep learning backend for the Pollock VAE with linear classification head)
- **scanpy** (Handles .h5ad AnnData inputs for merged scRNA-seq training sets and prediction outputs)
- **Seurat** (Handles .rds object inputs (raw counts in @assays$RNA@counts) for training and prediction)
- **Conda** (Environment creation for Pollock with pytorch, scanpy, umap-learn and r-base dependencies)

## Examples

```
pollock train from_scanpy --module-filepath generalized_module --scanpy-h5ad-filepath merged_pan_cancer.h5ad --cell-type-key cell_type
```

## Evaluation signals

- Reproduced F1 deltas fall within the reported spread: 8/12 cancer-type comparisons within 0.30 F1 of the disease-specific counterpart
- Best-generalizing cases reproduce: PDAC scRNA-seq delta 0.03; BRCA snRNA-seq 0.06 and snATAC-seq 0.02
- Worst-generalizing cases reproduce: CESC scRNA-seq delta 0.31; GBM snRNA-seq 0.52 and snATAC-seq 0.43
- Gene overlap check at prediction time: confirm model genes overlap after filtering (article example: 12679 overlapping, 4859 missing from dataset)
- Each generalized module trained with the documented defaults: epochs 20, latent-dim 64, encoder-out-dim 128, middle-dim 512, n-per-cell-type 500, kl-scaler 1e-3

## Limitations

- Generalized models perform somewhat worse than disease-specific counterparts, with F1 deltas up to 0.52 (GBM snRNA-seq) and 0.31 (CESC scRNA-seq)
- If fewer than N_PER_CELL_TYPE (default 500) cells exist for a cell type, cells are oversampled to balance the training dataset, which may affect benchmark fidelity
- snATAC-seq data are only represented as gene activity, limiting modality-specific signal
- No discussion section text or changelog was available from the source for additional caveats

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] There are a variety of modules available for cell type classification. The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221: "There are a variety of modules available for cell type classification. The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221"
- [readme] If increased the cell embeddings are typically more noisy, but typically moregeneralizable. If decreased the cell embeddings are typically less noisy, but typically less generalizable: "If increased the cell embeddings are typically more noisy, but typically moregeneralizable. If decreased the cell embeddings are typically less noisy, but typically less generalizable"
- [readme] N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled"
