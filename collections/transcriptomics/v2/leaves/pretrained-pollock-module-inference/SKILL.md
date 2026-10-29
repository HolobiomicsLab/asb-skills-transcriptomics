---
name: pretrained-pollock-module-inference
description: Use when cell type labels must be predicted for raw scRNA-seq count matrices
  (e.g. artificially dropout-masked validation datasets) using a pretrained disease/tissue-specific
  Pollock module, so F1 can be computed against annotated cell types.
when_to_use_negative:
- The input matrix is normalized rather than raw counts — Pollock expects raw expression
  counts (.X for scanpy, @assays$RNA@counts for Seurat)
- You need to train a new classifier on your own annotated data instead of applying
  an existing pretrained module (use train mode, not this skill)
- The tissue/disease context has no matching pretrained module (modules are disease-specific,
  e.g. breast cancer, melanoma, pancreatic cancer)
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Pollock
  role: Deep learning (VAE with linear classification head) cell type predictor run
    in predict mode on pretrained modules
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning backend installed via conda for running Pollock modules
- name: scanpy
  role: Loads raw count matrices (e.g. read_10x_mtx) and provides .h5ad anndata input
    for prediction; used to generate dropout-masked copies
- name: Docker
  role: Alternative containerized way to run the Pollock CLI with mounted input/output
    volumes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:9e3de023cc4000fba9cce4e123b441c0ff39e9ba577419dd1a6c3ea84943e4a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Docker
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# pretrained-pollock-module-inference

## Summary

Runs a pretrained Pollock VAE-plus-linear-classifier module in predict mode ('pollock predict') on scRNA-seq count data (Seurat .rds, scanpy .h5ad, or 10x cellranger input) to assign cell type labels with per-cell probabilities. This skill powers the dropout-robustness benchmark where masked copies of validation datasets are classified and scored by macro-averaged F1 against annotated labels.

## When to use

Use when cell type labels must be predicted for raw scRNA-seq count matrices (e.g. artificially dropout-masked validation datasets) using a pretrained disease/tissue-specific Pollock module, so F1 can be computed against annotated cell types.

## When NOT to use

- The input matrix is normalized rather than raw counts — Pollock expects raw expression counts (.X for scanpy, @assays$RNA@counts for Seurat)
- You need to train a new classifier on your own annotated data instead of applying an existing pretrained module (use train mode, not this skill)
- The tissue/disease context has no matching pretrained module (modules are disease-specific, e.g. breast cancer, melanoma, pancreatic cancer)

## Inputs

- Raw count matrices from scRNA-seq validation datasets (benchmarking_datasets.tar.gz, Zenodo record 5895221)
- Pretrained Pollock modules (pretrained_models.tar.gz, e.g. scRNAseq_generalized, scRNAseq_brca)
- Scanpy .h5ad, Seurat .rds, or 10x cellranger counts (mtx directory or .h5) with unnormalized counts

## Outputs

- Per-cell cell type predictions written back into the input object metadata or a tab-separated .txt file (--txt-output, --output-prefix output)
- Macro-averaged F1 versus annotated cell type labels per dropout level, collated in dropout_f1_scores.csv

## How to apply

Download pretrained_models.tar.gz and benchmarking_datasets.tar.gz from Zenodo record 5895221, then invoke the Pollock CLI in predict mode: 'pollock predict from_scanpy --module-filepath <module> --scanpy-h5ad-filepath <h5ad> --output-prefix output'. Inputs must contain raw (unnormalized) counts in the .X attribute of the anndata object (or @assays$RNA@counts for Seurat). For the dropout-robustness workflow, generate masked copies of each validation count matrix with 0%, 20%, 40%, 60%, 80%, and 90% of positions zeroed (uniform random dropout), predict cell types on each copy, and compute macro-averaged F1 of predictions versus annotated 'cell_type' labels. Use --no-umap to skip UMAP embedding of VAE latents and speed up repeated predictions. Validation: Pollock's F1 holds up to an additional 60% dropout and declines more steadily beyond that level.

## Related tools

- **Pollock** (Deep learning (VAE with linear classification head) cell type predictor run in predict mode on pretrained modules) — https://github.com/ding-lab/pollock
- **PyTorch** (Deep learning backend installed via conda for running Pollock modules)
- **scanpy** (Loads raw count matrices (e.g. read_10x_mtx) and provides .h5ad anndata input for prediction; used to generate dropout-masked copies)
- **Docker** (Alternative containerized way to run the Pollock CLI with mounted input/output volumes)

## Examples

```
pollock predict from_scanpy --module-filepath pretrained_models/scRNAseq_generalized --scanpy-h5ad-filepath validation_60pct_dropout.h5ad --output-prefix output --no-umap
```

## Evaluation signals

- Prediction log reports genes overlapping the model after filtering (e.g. '12679 genes overlap with model after filtering'); a large overlap confirms gene matching worked
- Predicted labels fall within the module's trained cell type classes (17 classes such as 'B cell', 'CD4 T cell', 'Monocyte', 'NK', 'Malignant')
- Macro-averaged F1 at 0% additional dropout is high and F1 remains stable through 60% added dropout, declining beyond 60%
- All six masked copies (0/20/40/60/80/90% dropout) produce predictions for the full cell set (e.g. 2638 cells) with no dropped cells
- Output file matches the requested format: metadata-embedded predictions by default, or a tab-separated .txt when --txt-output is set

## Limitations

- Input must be raw, unnormalized counts; normalized matrices will give invalid results
- Some model genes may be missing from the dataset (e.g. 4859 genes missing after filtering in the PBMC example), which can affect predictions
- UMAP embedding of VAE latents adds runtime; use --no-umap to skip it
- In 10x mode, cells below --min-genes-per-cell (default 10) are not classified

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] An example of cell type prediction on a scanpy .h5ad object: "An example of cell type prediction on a scanpy .h5ad object"
- [results] 2022-01-24 09:47:13,449 12679 genes overlap with model after filtering: "2022-01-24 09:47:13,449 12679 genes overlap with model after filtering"
- [other] Pollock's classification performance is robust to computationally added dropout: F1 holds up to an additional 60% dropout: [evidence span withheld — anchor status: not_found]
