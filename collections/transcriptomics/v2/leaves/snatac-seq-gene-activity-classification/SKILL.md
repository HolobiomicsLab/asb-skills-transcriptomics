---
name: snatac-seq-gene-activity-classification
description: Use when the input is snATAC-seq data expressed as gene activity (not
  peak calls) and the goal is cell type classification or benchmarking against Pollock's
  snATACseq pretrained modules.
when_to_use_negative:
- Input is single-cell peak-level accessibility (fragment files, peak matrices) rather
  than gene activity — the snATACseq modules are trained with snATAC-seq data represented
  as gene activity
- Input expression matrix is already normalized/log-transformed — the CLI requires
  raw, unnormalized counts in .X or @assays$RNA@counts
- Input is scRNA-seq or snRNA-seq — use the scRNAseq_* or snRNAseq_* pretrained modules
  instead of the snATACseq gene activity modules
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: VAE with a linear classification head run in train/predict mode on snATAC-seq
    gene activity inputs
  repo: https://github.com/ding-lab/pollock
- name: scanpy
  role: loads snATAC-seq gene activity data as .h5ad (from_scanpy source type) for
    training and prediction
- name: PyTorch
  role: backend for Pollock's deep learning training and prediction (installed via
    conda with the pollock environment)
- name: Conda
  role: environment setup to install Pollock and its dependencies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:5c76ce0a48d002377fdd2d69d394a8a5e8958ad32513b6c6ec17215fb0ea2a28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# snATAC-seq gene activity classification

## Summary

Train or apply Pollock's VAE-based cell type classifiers on snATAC-seq data represented as gene activity, using the disease-specific snATACseq modules and the snATAC-seq gene activity benchmarking datasets from Zenodo record 5895221.

## When to use

Use when the input is snATAC-seq data expressed as gene activity (not peak calls) and the goal is cell type classification or benchmarking against Pollock's snATACseq pretrained modules.

## When NOT to use

- Input is single-cell peak-level accessibility (fragment files, peak matrices) rather than gene activity — the snATACseq modules are trained with snATAC-seq data represented as gene activity
- Input expression matrix is already normalized/log-transformed — the CLI requires raw, unnormalized counts in .X or @assays$RNA@counts
- Input is scRNA-seq or snRNA-seq — use the scRNAseq_* or snRNAseq_* pretrained modules instead of the snATACseq gene activity modules

## Inputs

- snATAC-seq gene activity matrix as a scanpy .h5ad (.X = raw, unnormalized gene activity) or Seurat .RDS object with counts in @assays$RNA@counts
- Cell type label column (e.g. 'cell_type' in metadata/.obs) for train mode
- benchmarking_datasets.tar.gz and pretrained_models.tar.gz from Zenodo record 5895221 (snATACseq_brca, snATACseq_ccrcc, snATACseq_generalized modules)

## Outputs

- Trained Pollock snATACseq module directory (--module-filepath)
- Per-cell predicted cell type labels written back into the input object's metadata or a tab-separated .txt file via --txt-output
- UMAP-transformed VAE embeddings incorporated into the predicted object (unless --no-umap)
- Per-cell-type and macro-averaged F1 scores for the method-by-dataset comparison table (benchmark_f1_comparison.csv)

## How to apply

Download snATAC-seq gene activity datasets with annotated cell type labels and train/validation splits from benchmarking_datasets.tar.gz on Zenodo record 5895221, and optionally the matching pretrained modules from pretrained_models.tar.gz (e.g. snATACseq_brca, snATACseq_ccrcc, snATACseq_generalized). Supply the gene activity matrix as the raw (unnormalized) expression source — in scanpy mode the .X attribute of the .h5ad must contain raw counts — via 'pollock train from_scanpy' or 'pollock predict from_scanpy' with --cell-type-key set to the label column. Train with the paper hyperparameters (epochs=20, latent-dim=64, encoder-out-dim=128, middle-dim=512, kl-scaler=1e-3, zinb-scaler=0.5, clf-scaler=1.0, n-per-cell-type=500 validation split) and predict on held-out validation cells. Evaluate per-cell-type and macro-averaged F1 against the baseline classifiers on the 3 snATAC-seq gene activity benchmarking datasets, checking Pollock is within 0.02 F1 of the top-performing tool. Raw gene activity (not normalized) input is essential because the VAE models unnormalized counts with a ZINB reconstruction loss.

## Related tools

- **Pollock** (VAE with a linear classification head run in train/predict mode on snATAC-seq gene activity inputs) — https://github.com/ding-lab/pollock
- **scanpy** (loads snATAC-seq gene activity data as .h5ad (from_scanpy source type) for training and prediction)
- **PyTorch** (backend for Pollock's deep learning training and prediction (installed via conda with the pollock environment))
- **Conda** (environment setup to install Pollock and its dependencies)

## Examples

```
pollock train from_scanpy --module-filepath output_snatac_module --scanpy-h5ad-filepath gene_activity.h5ad --cell-type-key cell_type --epochs 20 --n-per-cell-type 500
```

## Evaluation signals

- The gene activity .h5ad/.RDS supplied to Pollock contains raw, unnormalized values (article logs show raw counts_adata is copied before sc.pp.normalize_total/log1p are applied to visualization copies)
- Prediction log reports gene overlap with the model (e.g. '12679 genes overlap with model after filtering'; '4859 genes missing from dataset after filtering')
- Macro-averaged F1 on each of the 3 snATAC-seq gene activity benchmarking datasets is within 0.02 of the best-performing classifier, matching the benchmark finding
- Predicted object includes VAE-embedding-derived UMAP columns unless --no-umap is passed
- Trained module directory contains a usable model loadable for subsequent --module-filepath prediction runs

## Limitations

- snATAC-seq data must be represented as gene activity; the pretrained snATACseq modules were trained on gene activity representations, so peak-level inputs are incompatible
- Validation split default n-per-cell-type=500 oversamples cells when a cell type has fewer than N_PER_CELL_TYPE cells, which can bias small cell types
- The README's snATACseq module list contains naming inconsistencies (snATACseq_brca listed twice, and the glioblastoma entry labeled 'snATACseq_brca'), so verify module contents against downloaded tarballs
- Explain mode has excessive runtimes for large cell numbers; downsample to roughly 100 explained cells with ~100 background cells (~15 minutes runtime)

## Evidence

- [readme] snATAC-seq data is represented as gene activity: "snATACseq_brca (trained with disease-specific breast cancer dataset, snATAC-seq data is represented as gene activity)"
- [readme] raw counts requirement for scanpy input: "scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
- [methods] Zenodo archive locations for pretrained models and benchmarking datasets: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [results] gene overlap filtering log during prediction: "2022-01-24 09:47:13,449 12679 genes overlap with model after filtering
2022-01-24 09:47:13,450 4859 genes missing from dataset after filtering"
- [methods] Pollock architecture: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
