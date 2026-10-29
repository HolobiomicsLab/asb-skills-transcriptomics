---
name: single-cell-benchmarking-f1-evaluation
description: Use when you need to quantitatively compare a single-cell (scRNA-seq,
  snRNA-seq, or snATAC-seq gene activity) cell type classifier against established
  baselines using held-out validation F1 on annotated, pre-split benchmark datasets.
when_to_use_negative:
- The goal is cell type annotation of a new dataset with a pretrained model rather
  than head-to-head classifier benchmarking — use Pollock predict mode instead
- The datasets lack annotated cell type labels or predefined train/validation splits,
  since supervised training and F1 evaluation both require ground-truth labels
- The data are not single-cell expression matrices (e.g., bulk RNA-seq, proteomics,
  or already-aggregated feature tables), which are outside the scope of these single-cell
  classifiers
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Deep learning classifier (VAE with linear classification head) trained in
    train mode on each benchmark dataset and evaluated on validation splits
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning backend installed via conda on which Pollock training and prediction
    run
- name: scanpy
  role: Used as a baseline classifier for benchmarking and as one of Pollock's input
    source types (from_scanpy, .h5ad with raw counts in .X)
- name: Conda
  role: Environment management for installing Pollock, PyTorch, and scanpy dependencies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:8a447018bd437df5ebe051e717ef039035eb64db97bc20b739306cbc20eae0a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-benchmarking-f1-evaluation

## Summary

Benchmark a cell type classification method (e.g., Pollock, a VAE with a linear classification head) against established classifiers (Seurat, Scanpy, SingleCellNet, ACTINN, SVM-linear, MLP) by training each on shared train/validation splits of annotated single-cell datasets and comparing per-cell-type and macro-averaged F1 scores.

## When to use

Use when you need to quantitatively compare a single-cell (scRNA-seq, snRNA-seq, or snATAC-seq gene activity) cell type classifier against established baselines using held-out validation F1 on annotated, pre-split benchmark datasets.

## When NOT to use

- The goal is cell type annotation of a new dataset with a pretrained model rather than head-to-head classifier benchmarking — use Pollock predict mode instead
- The datasets lack annotated cell type labels or predefined train/validation splits, since supervised training and F1 evaluation both require ground-truth labels
- The data are not single-cell expression matrices (e.g., bulk RNA-seq, proteomics, or already-aggregated feature tables), which are outside the scope of these single-cell classifiers

## Inputs

- benchmarking_datasets.tar.gz from Zenodo record 5895221: 12 annotated single-cell datasets (6 scRNA-seq, 3 snRNA-seq, 3 snATAC-seq gene activity) with cell type labels and train/validation splits
- Raw (unnormalized) expression counts in .h5ad (scanpy) or Seurat .RDS format for Pollock training/prediction
- Trained model outputs from each of the seven classifiers (Pollock plus six baselines) on each validation split

## Outputs

- Per-cell-type and macro-averaged F1 scores for each method on each of the 12 datasets
- Method-by-dataset F1 comparison table (benchmark_f1_comparison.csv)
- Trained Pollock modules saved to the specified --module-filepath directory

## How to apply

Download benchmarking_datasets.tar.gz from Zenodo record 5895221, which contains 12 annotated datasets (6 scRNA-seq, 3 snRNA-seq, 3 snATAC-seq gene activity) with defined train/validation splits. Train Pollock on each dataset in train mode using the paper hyperparameters (epochs=20, batch size=64, latent dim=64, encoder-out dim=128, middle dim=512, lr=1e-4, kl-scaler=1e-3, zinb-scaler=0.5, clf-scaler=1.0, validation split via n-per-cell-type=500), supplying raw unnormalized counts as input. Train the six baseline classifiers (Seurat, Scanpy, SingleCellNet, ACTINN, linear SVM, MLP) on the identical splits of each dataset. Predict labels on each validation split with every trained model, then compute per-cell-type and macro-averaged F1 scores per method per dataset and assemble the method-by-dataset comparison table (benchmark_f1_comparison.csv). Correct application is judged by Pollock being within 0.02 F1 of the top-performing tool on the expected number of datasets (11 of 12 in the paper).

## Related tools

- **Pollock** (Deep learning classifier (VAE with linear classification head) trained in train mode on each benchmark dataset and evaluated on validation splits) — https://github.com/ding-lab/pollock
- **PyTorch** (Deep learning backend installed via conda on which Pollock training and prediction run)
- **scanpy** (Used as a baseline classifier for benchmarking and as one of Pollock's input source types (from_scanpy, .h5ad with raw counts in .X))
- **Conda** (Environment management for installing Pollock, PyTorch, and scanpy dependencies)

## Examples

```
pollock train from_scanpy --module-filepath <output_module_dir> --scanpy-h5ad-filepath <benchmark_dataset.h5ad> --cell-type-key cell_type --epochs 20 --n-per-cell-type 500
```

## Evaluation signals

- Pollock is within 0.02 F1 of the top-performing tool in 11 out of 12 benchmarking datasets (the paper's reported result)
- Each of the seven methods is trained and evaluated on identical train/validation splits of each dataset — no leakage between splits
- Pollock inputs are raw unnormalized counts (the README requires raw counts in .X for scanpy or @assays$RNA@counts for Seurat)
- Per-cell-type and macro-averaged F1 are both computed for every method-by-dataset pair before assembling benchmark_f1_comparison.csv
- Validation split behavior matches the documented default: n-per-cell-type=500 partitioned per cell type, with oversampling when a cell type has fewer than 500 cells

## Limitations

- Pollock requires raw (unnormalized) counts as input; supplying normalized data (e.g., after normalize_total to 1e4 and log1p) violates its input contract
- Default validation splitting via n-per-cell-type=500 oversamples cell types with fewer than 500 cells, which may affect comparability for rare cell types
- snATAC-seq benchmarks rely on gene activity representations rather than peak data, so results do not directly assess peak-level classification
- Explain mode and prediction on very large cell numbers incur long runtimes; benchmarking on 12 datasets requires batched, reproducible training runs

## Evidence

- [full_text] Pollock performs comparably to existing classification tools, being within an F1-score of 0.02 of the top-performing tool in 11 out of 12 datasets.: [evidence span withheld — anchor status: not_found]
- [methods] Pollock is a VAE with linear classification head: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [methods] Models and training datasets on Zenodo in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] README: scanpy .h5ad input must be raw expression counts: "scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
- [readme] README: n-per-cell-type determines validation/training split, default 500: [evidence span withheld — anchor status: not_found]
- [readme] README: default epochs is 20: "Number of epochs to train the neural net for. Default is 20."
