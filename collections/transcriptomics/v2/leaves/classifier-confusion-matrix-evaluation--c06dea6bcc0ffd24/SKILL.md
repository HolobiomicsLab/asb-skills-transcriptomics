---
name: classifier-confusion-matrix-evaluation
description: Use when a Pollock model has been trained on annotated scRNA-seq data
  (e.g. BRCA pan-immune states) and you must quantify which cell states a withheld
  validation partition classifies poorly.
when_to_use_negative:
- You only need overall (global) model accuracy and do not care which specific cell
  states are confused — per-state metrics and the confusion matrix would be superfluous.
- The input is unannotated data with no validation labels available, so per-state
  accuracy against ground truth cannot be computed.
- The task is cell type discovery/clustering rather than benchmarking an existing
  trained classifier against known labels.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Trains the VAE classifier in train mode and produces per-cell predicted labels
    in predict mode with --txt-output; the core engine being evaluated
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Backend framework for the Pollock VAE model training and prediction
- name: scanpy
  role: Loads the BRCA pan-immune .h5ad data and supplies the cell state annotation
    column via --cell-type-key; predictions can be written back to the anndata object
- name: Conda
  role: Creates the pollock environment with pytorch, captum, and scanpy needed to
    train and predict
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:003b5e469582148ec3143041ef04b379372dbf8fb1882498d6471c7559bfe10a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# classifier-confusion-matrix-evaluation

## Summary

Evaluates a trained Pollock single-cell classifier on withheld validation data by computing per-cell-type accuracy and a confusion matrix, exposing which fine-grained cell states (e.g. exhausted CD8 T cells, NKT cells) are most often misclassified.

## When to use

Use when a Pollock model has been trained on annotated scRNA-seq data (e.g. BRCA pan-immune states) and you must quantify which cell states a withheld validation partition classifies poorly.

## When NOT to use

- You only need overall (global) model accuracy and do not care which specific cell states are confused — per-state metrics and the confusion matrix would be superfluous.
- The input is unannotated data with no validation labels available, so per-state accuracy against ground truth cannot be computed.
- The task is cell type discovery/clustering rather than benchmarking an existing trained classifier against known labels.

## Inputs

- benchmarking_datasets.tar.gz (Zenodo record 5895221) containing the BRCA pan-immune scRNA-seq dataset with 17 immune cell state annotations
- Trained Pollock module directory saved via --module-filepath
- Withheld validation cells in a Seurat .rds, scanpy .h5ad, or 10x cellranger matrix with raw counts
- Ground-truth cell state labels for validation cells (metadata column matching --cell-type-key)

## Outputs

- Per-cell predicted cell state labels as a tab-separated .txt file (--txt-output)
- Per-cell-state accuracy table
- Cell state x cell state confusion matrix identifying confusable T/NK states

## How to apply

Train the Pollock VAE-plus-linear-head module with 'pollock train from_scanpy' (or from_seurat), setting --cell-type-key to the immune cell state annotation column and keeping the default hyperparameters (epochs 20, latent-dim 64, encoder-out-dim 128, middle-dim 512, kl-scaler 1e-3, n-per-cell-type 500 with oversampling of rarer states) so a validation partition is held out. Then run 'pollock predict' on the withheld cells with --txt-output to write per-cell predicted labels as a tab-separated .txt file. Parse the per-cell predictions against the withheld ground-truth labels to compute per-cell-state accuracy and a confusion matrix. Judge success by whether the confusion pattern reproduces the paper's Fig. 4A: exhausted CD8 T cells and NKT cells have the lowest accuracy and are primarily confused with other T and NK cell states. Rationale: oversampling via --n-per-cell-type balances rare states during training, so failures on rare, transcriptionally similar states surface as off-diagonal mass within T/NK lineages in the confusion matrix.

## Related tools

- **Pollock** (Trains the VAE classifier in train mode and produces per-cell predicted labels in predict mode with --txt-output; the core engine being evaluated) — https://github.com/ding-lab/pollock
- **PyTorch** (Backend framework for the Pollock VAE model training and prediction)
- **scanpy** (Loads the BRCA pan-immune .h5ad data and supplies the cell state annotation column via --cell-type-key; predictions can be written back to the anndata object)
- **Conda** (Creates the pollock environment with pytorch, captum, and scanpy needed to train and predict)

## Examples

```
pollock predict from_scanpy --module-filepath brca_panimmune_module --scanpy-h5ad-filepath brca_validation.h5ad --output-prefix output --txt-output
```

## Evaluation signals

- Per-cell-state accuracy on the withheld validation set reproduces the Fig. 4A pattern: exhausted CD8 T cells and NKT cells are the two lowest-accuracy states
- The confusion matrix shows the misclassified exhausted CD8 T cells and NKT cells are primarily assigned to other T and NK cell states (off-diagonal mass stays within the T/NK lineage)
- The --txt-output file is written as a tab-separated file with one predicted label per validation cell, and row count equals the number of validation cells
- Training logs/splits confirm rarer cell states were oversampled to --n-per-cell-type (default 500) rather than dropped
- Predicted labels were generated from raw counts only (input .h5ad/.rds .X or counts slot not normalized), as Pollock requires

## Limitations

- Rare or transcriptionally subtle states (exhausted CD8 T cells, NKT cells) are inherently hard to classify and may remain confused with related T/NK states even with a well-trained model
- The train/validation split is driven by --n-per-cell-type (default 500 per type, with oversampling when a state has fewer cells), so the split composition affects reported per-state accuracy
- Running ~100 cells through explain mode takes ~15 minutes with a background sample size of ~100, so downstream interpretation of confusions is runtime-limited for large cell counts

## Evidence

- [other] lowest-accuracy cell states on withheld BRCA validation: [evidence span withheld — anchor status: not_found]
- [methods] Pollock model architecture: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] txt output format for predictions: "If included output will be written to a tab-seperated .txt file. Otherwise output will be saved in the metadata of the input seurat object"
- [readme] validation split and oversampling behavior: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled"
- [readme] pan-immune BRCA pretrained module availability: "scRNAseq_brca_panimmune (trained with specific immune cell state breast cancer dataset)"
