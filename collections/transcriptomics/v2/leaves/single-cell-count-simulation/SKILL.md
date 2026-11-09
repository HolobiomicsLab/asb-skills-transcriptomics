---
name: single-cell-count-simulation
description: Use when you need ground-truth scRNA-seq count data with known celltype
  labels and an applied dropout model to quantify whether a denoiser (DCA vs. an MSE
  autoencoder) recovers the true cell structure.
when_to_use_negative:
- Real scRNA-seq data where the true expression state (ground truth) is unknown —
  the article notes 'evaluation of denoising is difficult because the definition of
  a ground truth can be challenging for real data'.
- Benchmarks requiring hyperparameter-optimized denoisers — the authors state hyperparameters
  were not fine-tuned for any of the manuscript analyses, so default settings are
  the reproducible baseline, not a tuned configuration.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Generates the two-group and six-group simulated scRNA-seq count data (200
    genes, 2000 cells) with ground-truth celltype labels
- name: DCA
  role: Denoises the dropout simulations using the ZINB loss and default 64-32-64
    autoencoder, providing the count-loss-based arm of the comparison
  repo: https://github.com/theislab/dca
- name: anndata
  role: Stores simulated count matrices, ground truth, and celltype labels in h5ad
    format for loading via the autoencoder.io module
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Provides the Python API for running DCA preprocessing and downstream visualization
    (PCA) of simulated and denoised data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:9f2adcbd8a2e3eda358b4d492ac091f08de67dff179c940f5c29e2ae04f97671
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - Splatter
  - anndata
  - numpy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-count-simulation

## Summary

Benchmarking skill that generates synthetic scRNA-seq count matrices with controlled dropout (e.g. two-group and six-group Splatter simulations of 200 genes x 2000 cells) so that denoisers such as DCA can be evaluated against a known ground truth.

## When to use

Use when you need ground-truth scRNA-seq count data with known celltype labels and an applied dropout model to quantify whether a denoiser (DCA vs. an MSE autoencoder) recovers the true cell structure.

## When NOT to use

- Real scRNA-seq data where the true expression state (ground truth) is unknown — the article notes 'evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data'.
- Benchmarks requiring hyperparameter-optimized denoisers — the authors state hyperparameters were not fine-tuned for any of the manuscript analyses, so default settings are the reproducible baseline, not a tuned configuration.

## Inputs

- Splatter simulation parameters (two or six celltype groups, 200 genes, 2000 cells)
- Noisy count matrices produced by the Methods dropout model
- No-dropout ground-truth count matrices and ground-truth celltype labels
- AnnData h5ad objects holding simulated counts and labels

## Outputs

- Dropout-perturbed count matrices paired with their ground-truth versions
- DCA (ZINB mean, e.g. mean.tsv) and MSE-autoencoder denoised count matrices
- PCA plots of ground truth vs. denoised matrices colored by simulated celltype (e.g. simulation_pca.png)
- Comparison table of silhouette scores and clustering agreement (e.g. simulation_denoising_metrics.csv)

## How to apply

Simulate two-group and six-group scRNA-seq count data with Splatter (200 genes, 2000 cells) and apply the paper's Methods dropout model to produce noisy count matrices alongside the no-dropout ground truth. Load the simulated counts and ground-truth celltype labels into AnnData h5ad objects via the autoencoder.io module. Denoise each dropout simulation with DCA using the zero-inflated negative binomial (ZINB) loss and the default network (three hidden layers with 64, 32, 64 neurons) trained with RMSProp at learning rate 0.001, reduced by 0.1 after 20 epochs without validation-loss improvement, with early stopping after 25 epochs; train a plain MSE-reconstruction autoencoder on the same data as the baseline. Run PCA on the ground-truth, DCA-denoised, and MSE-denoised matrices, color cells by simulated celltype, and quantify recovery with silhouette scores and clustering agreement (e.g. ARI) against the ground-truth labels.

## Related tools

- **Splatter** (Generates the two-group and six-group simulated scRNA-seq count data (200 genes, 2000 cells) with ground-truth celltype labels)
- **DCA** (Denoises the dropout simulations using the ZINB loss and default 64-32-64 autoencoder, providing the count-loss-based arm of the comparison) — https://github.com/theislab/dca
- **anndata** (Stores simulated count matrices, ground truth, and celltype labels in h5ad format for loading via the autoencoder.io module) — https://github.com/theislab/anndata
- **Scanpy** (Provides the Python API for running DCA preprocessing and downstream visualization (PCA) of simulated and denoised data)

## Examples

```
dca matrix.csv results
```

## Evaluation signals

- The comparison table contains silhouette and clustering-agreement (ARI) values for both group counts (two and six celltypes), both denoisers, and the ground truth.
- DCA's celltype-recovery metrics (silhouette, ARI) are at least as high as the MSE autoencoder's on the dropout simulations.
- PCA of the DCA-denoised matrix shows cells separated by simulated celltype matching the ground-truth PCA, while the MSE-autoencoder PCA fails to separate celltypes.
- The DCA output preserves input dimensions (genes x cells), excluding only zero-expression genes/cells as described in the README.
- Training logs confirm the expected regime: RMSProp, learning rate 0.001, 0.1 reduction after 20 epochs without validation-loss improvement, early stopping after 25 epochs.

## Limitations

- Denoising evaluation is difficult because defining a ground truth for real (non-simulated) data is challenging — simulation-based benchmarks sidestep this but may not transfer.
- Overimputation can occur with inadequate hyperparameter choices, such as a too low-dimensional bottleneck layer and hence data manifold.
- It may be difficult to determine in general when denoising improves scRNA-seq data, so positive benchmark results are not a universal guarantee.

## Evidence

- [abstract] We demonstrate that DCA denoising improves a diverse set of typical scRNA-seq data analyses using simulated and real datasets.: "We demonstrate that DCA denoising improves a diverse set of typical scRNA-seq data analyses using simulated and real datasets."
- [discussion] Default training regime: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [intro] Default network size: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [discussion] Ground-truth difficulty: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
- [readme] Output format of DCA denoised matrix: "mean.tsv is the main output of the method which represents the mean parameter of the ZINB distribution."
