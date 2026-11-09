---
name: immune-cell-state-annotation-handling
description: Use when the input is annotated single-cell RNA-seq with fine-grained
  immune cell state labels (e.g. exhausted CD8 T cell, NKT, Treg) and you need to
  train a Pollock classifier and report per-cell-state confusion/accuracy.
when_to_use_negative:
- Input cells are unlabeled and the goal is prediction only — use a pretrained module
  (e.g. scRNAseq_brca_panimmune) with 'pollock predict' rather than training
- The annotation column contains only coarse lineages (e.g. 'B cell', 'Monocyte',
  'Malignant') rather than fine-grained immune cell states — the pan-immune 17-state
  module and its accuracy patterns do not apply
- Input is not raw counts (the .h5ad .X or Seurat counts slot must be unnormalized
  counts for training and prediction)
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Pollock
  role: Trains a VAE-with-linear-classification-head module on the immune cell state
    key and predicts withheld validation cell state labels
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning backend used in the pollock conda environment for training the
    classification module
- name: scanpy
  role: Loads and passes the annotated .h5ad object (raw counts in .X) to 'pollock
    train from_scanpy'
- name: Conda
  role: Creates the pollock environment with pytorch, captum, and scanpy before pip-installing
    the cloned repo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:284b4dccb2621f33048ddc411b758e8797ddfa6fc38ce64bce49ea40eb2b2b77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# immune-cell-state-annotation-handling

## Summary

Handles fine-grained immune cell state annotations (e.g. the 17-state BRCA pan-immune scRNA-seq labels) when training and predicting with Pollock, including oversampling of rare states for training splits and per-cell-state accuracy/confusion evaluation on withheld validation cells.

## When to use

Use when the input is annotated single-cell RNA-seq with fine-grained immune cell state labels (e.g. exhausted CD8 T cell, NKT, Treg) and you need to train a Pollock classifier and report per-cell-state confusion/accuracy.

## When NOT to use

- Input cells are unlabeled and the goal is prediction only — use a pretrained module (e.g. scRNAseq_brca_panimmune) with 'pollock predict' rather than training
- The annotation column contains only coarse lineages (e.g. 'B cell', 'Monocyte', 'Malignant') rather than fine-grained immune cell states — the pan-immune 17-state module and its accuracy patterns do not apply
- Input is not raw counts (the .h5ad .X or Seurat counts slot must be unnormalized counts for training and prediction)

## Inputs

- Annotated scRNA-seq dataset with fine-grained immune cell state labels in a metadata/.obs column (BRCA pan-immune dataset from benchmarking_datasets.tar.gz, Zenodo record 5895221)
- Raw (unnormalized) counts in the Seurat @assays$RNA@counts slot or the scanpy .X attribute of an .h5ad object
- Withheld validation set of cells (or a train/validation split generated via --n-per-cell-type)

## Outputs

- Trained Pollock cell type classification module saved to --module-filepath
- Per-cell predicted cell state labels as a tab-separated .txt file (--txt-output)
- Per-cell-state accuracy values and a confusion matrix over the 17 immune cell states

## How to apply

Load the pan-immune scRNA-seq data (benchmarking_datasets.tar.gz from Zenodo record 5895221) into scanpy/Seurat and pass the immune cell state annotation column via --cell-type-key to 'pollock train from_scanpy' (or from_seurat). Train with default hyperparameters from the methods — epochs 20, latent-dim 64, encoder-out-dim 128, middle-dim 512, kl-scaler 1e-3, n-per-cell-type 500 — so that fewer cells than N_PER_CELL_TYPE for a state are oversampled to balance the training set. Save the trained module to --module-filepath, then run 'pollock predict' on the withheld validation cells with --txt-output to write per-cell predicted cell state labels. Compute per-cell-state accuracy and a confusion matrix from the output; on the BRCA validation data the hardest states are exhausted CD8 T cells and NKT cells, which are confused mainly with other T and NK cell states (Fig. 4A pattern).

## Related tools

- **Pollock** (Trains a VAE-with-linear-classification-head module on the immune cell state key and predicts withheld validation cell state labels) — https://github.com/ding-lab/pollock
- **PyTorch** (Deep learning backend used in the pollock conda environment for training the classification module)
- **scanpy** (Loads and passes the annotated .h5ad object (raw counts in .X) to 'pollock train from_scanpy')
- **Conda** (Creates the pollock environment with pytorch, captum, and scanpy before pip-installing the cloned repo)

## Examples

```
pollock train from_scanpy --scanpy-h5ad-filepath brca_panimmune.h5ad --cell-type-key cell_type --module-filepath output_module
```

## Evaluation signals

- Withheld-validation per-cell-state accuracy shows exhausted CD8 T cells and NKT cells as the lowest-accuracy states, matching the Fig. 4A pattern
- Confusion matrix shows the misclassified exhausted CD8 T cell and NKT cells are predicted as other T and NK cell states, not unrelated lineages
- Training split used n-per-cell-type 500 with oversampling of rarer cell states (fewer than 500 cells for a state triggers oversampling)
- Trained module directory is written to --module-filepath and 'pollock predict' runs on the withheld validation cells without error
- With --txt-output, a tab-separated per-cell prediction file is produced for every validation cell

## Limitations

- Rare immune cell states have few cells; the --n-per-cell-type split oversamples them to balance training, which can affect classification of the rarest states (exhausted CD8 T cells and NKT cells showed the lowest accuracy)
- By default only N_PER_CELL_TYPE (500) cells per cell type are used for training; --use-all-cells must be passed to override this
- Expression matrix must be raw counts (not normalized) — normalized .h5ad .X or Seurat counts slots will violate the input expectation

## Evidence

- [readme] If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled to balance the training dataset. Default is 500.: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled"
- [readme] At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] The key must be a sring representing a column in the metadata of the input seurat object or .obs attribute of the scanpy anndata object.: "The key must be a sring representing a column in the metadata of the input seurat object or .obs attribute of the scanpy anndata object."
- [results] On the withheld BRCA breast validation dataset, the cell types with the lowest classification accuracy were exhausted CD8 T cells and NKT cells.: [evidence span withheld — anchor status: not_found]
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
