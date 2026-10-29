---
name: autoencoder-based-expression-denoising
description: Use when a sparse scRNA-seq expression matrix (cells x genes) with simulated
  dropout or low sampling efficiency (e.g., 5–10%) needs a denoised reconstruction
  before imputation or downstream clustering.
when_to_use_negative:
- 'Small genomic datasets: SERM learns the model from a small fraction of the data,
  so the estimated distribution may diverge from the actual distribution and yield
  inaccurate imputation values'
- Datasets where ROI coverage is too small for the captured data distribution to be
  representative of the entire expression matrix
- 'Data requiring explicit batch-effect correction: SERM''s formulation does not consider
  batch effects explicitly (only a z-score integration step is applied post-hoc)'
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Primary autoencoder framework whose encoder-decoder step denoises the noisy
    expression matrix before distribution fitting and histogram-equalization imputation;
    pip-installable as 'serm'
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: SAUCIE
  role: Alternative TensorFlow sparse autoencoder for imputing and embedding single-cell
    data, comparable to the denoising role SERM's autoencoder plays
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: SciPy
  role: Loads the .mat input files (scipy.io.loadmat) that feed the autoencoder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:7564ce9e13dc1c28d8e04bef51231dee4a404433a66b712a1e21b5bd8e5651a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - SAUCIE
  - SciPy
  - self-consistent expression recovery machine (SERM)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# autoencoder-based-expression-denoising

## Summary

Train a deep autoencoder (as in SERM's first pipeline step) to extract a latent representation from sparse, dropout-affected scRNA-seq data and reconstruct denoised expression values, which then serve as the basis for distribution learning and histogram-equalization imputation.

## When to use

Use when a sparse scRNA-seq expression matrix (cells x genes) with simulated dropout or low sampling efficiency (e.g., 5–10%) needs a denoised reconstruction before imputation or downstream clustering.

## When NOT to use

- Small genomic datasets: SERM learns the model from a small fraction of the data, so the estimated distribution may diverge from the actual distribution and yield inaccurate imputation values
- Datasets where ROI coverage is too small for the captured data distribution to be representative of the entire expression matrix
- Data requiring explicit batch-effect correction: SERM's formulation does not consider batch effects explicitly (only a z-score integration step is applied post-hoc)

## Inputs

- Sparse observed scRNA-seq expression matrix (.mat, cells x genes), e.g. data_observed_10perc.mat
- Optional reference expression matrix (.mat) for evaluation, e.g. data_reference.mat (12,162 cells x 2,422 genes)

## Outputs

- Denoised/reconstructed expression matrix (same cells x genes dimensions) used for downstream SERM imputation
- Latent representation of the data (latent dimension P=2 in SERM's configuration)

## How to apply

Load the observed expression matrix (e.g., data_observed_10perc.mat for the 12,162-cell x 2,422-gene bone-marrow-stroma cellular taxonomy reference) as a cells-by-genes matrix. Train the SERM autoencoder — a three-layer encoder-decoder with latent dimension P=2, learning rate 1e-5, max epoch 20, batch size 64, adam optimizer — on a subset of the noisy data so the network learns the underlying data distribution and reconstructs denoised expression values. These denoised values feed the next SERM steps (curve fitting of a parametric PDF, adaptive histogram equalization over ROIs of e.g. 6100 x 1250 with 50% overlap). Ablation checks (empirical-distribution or quantile-normalization substitutions) verify that the learned reconstruction, not a generic smoothing, drives imputation gains. The rationale is enforcing data self-consistency: a low-dimensional latent code filters dropout noise so reconstructed values are similarly distributed across matrix regions.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Primary autoencoder framework whose encoder-decoder step denoises the noisy expression matrix before distribution fitting and histogram-equalization imputation; pip-installable as 'serm') — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **SAUCIE** (Alternative TensorFlow sparse autoencoder for imputing and embedding single-cell data, comparable to the denoising role SERM's autoencoder plays) — https://github.com/KrishnaswamyLab/SAUCIE
- **SciPy** (Loads the .mat input files (scipy.io.loadmat) that feed the autoencoder)

## Examples

```
import scipy.io as sio, serm as srm; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; out_SERM = srm.serm(dataObs, [6100, 1250], 0.5, randomize=False)
```

## Evaluation signals

- Pearson correlation coefficient between reference and imputed values per cell lies in [-1, 1], with SERM-imputed data showing percent improvement over observed data (e.g., >20% improvement over state-of-the-art for the cellular taxonomy dataset)
- Number of zeros in the SERM-imputed data is similar to the number in the dropout-free reference data
- At 5% sampling efficiency the reconstruction yields Pearson coefficients above 0.9 for most genes, versus <=0.4 for most other methods
- Autoencoder training uses the documented configuration (three-layer encoder-decoder, latent dimension P=2, lr 1e-5, max epoch 20, batch 64, adam) and converges within max epoch budget
- Ablation substitutions (empirical-distribution target, quantile-normalization instead of adaptive histogram equalization) degrade Pearson performance relative to standard SERM, confirming the learned reconstruction matters

## Limitations

- SERM learns the gene expression model from a small fraction of the data, which is a challenge for small genomic datasets where the estimated distribution may diverge from the actual distribution
- Reference evaluation data were created by choosing only cells and genes with high expression, and dropouts were simulated rather than observed experimentally
- Performance depends on the implicit assumption that ROI coverage is sufficiently large so the captured distribution is representative of the entire matrix
- Batch effects are not considered explicitly in the formulation (mitigated by a post-hoc z-score integration of batches)

## Evidence

- [intro] autoencoder learns latent representation and reconstructs denoised values: "The approach leverages a deep learning optimization to extract the latent representation of the data and reconstruct the denoised expression values"
- [methods] autoencoder architecture definition: "The deep learning architecture (autoencoder) used in SERM consists of two parts, an encoder and a decoder."
- [abstract] neural network learns distribution from noisy subset: "Using a neural network, the technique first learns the underlying data distribution from a subset of the noisy data."
- [readme] README description of deep learning step: "Using deep learning, SERM first learns from a subset of the noisy gene expression data to estimate the underlying data distribution."
- [readme] reference dataset construction from raw data: "we choose 12,162 cells and 2,422 genes with high expression to create a reference dataset."
