---
name: autoencoder-denoising-of-scrna-seq-expression
description: Use when a sparse scRNA-seq expression matrix with simulated dropout
  (low sampling efficiency, e.g. 0.1%–10%) must be denoised before downstream distribution
  learning or imputation, and you need a latent representation of the noisy data.
when_to_use_negative:
- Input is already a clean, dropout-free reference matrix (e.g. reference data created
  by choosing cells and genes with high expression) — denoising targets sparse observed
  data
- Small genomic datasets, where the distribution learned from a small data subset
  may diverge from the true data distribution and yield inaccurate imputations
- Analyses that explicitly require batch-effect modeling — batch effects are not considered
  explicitly in the SERM formulation
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Framework whose first stage ('Denoising Via Auto-Encoding') trains the three-layer
    autoencoder to reconstruct denoised expression values that feed downstream histogram
    computation and PDF curve fitting
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:76d9a13c7de7f3e0fa902a4f0e22b1542dc395c5e0bf5ac0446031c98b4e618c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# autoencoder-denoising-of-scrna-seq-expression

## Summary

Trains a three-layer autoencoder (logistic sigmoid encoder, linear decoder, latent dimension P=2) on a small random subset of a sparse scRNA-seq count matrix (e.g. data_observed_10perc.mat) to reconstruct denoised expression values (matrix So), the first modeling stage of SERM's gene expression recovery pipeline.

## When to use

Use when a sparse scRNA-seq expression matrix with simulated dropout (low sampling efficiency, e.g. 0.1%–10%) must be denoised before downstream distribution learning or imputation, and you need a latent representation of the noisy data.

## When NOT to use

- Input is already a clean, dropout-free reference matrix (e.g. reference data created by choosing cells and genes with high expression) — denoising targets sparse observed data
- Small genomic datasets, where the distribution learned from a small data subset may diverge from the true data distribution and yield inaccurate imputations
- Analyses that explicitly require batch-effect modeling — batch effects are not considered explicitly in the SERM formulation

## Inputs

- Sparse dropout-affected scRNA-seq expression matrix loaded from a .mat file (e.g. data_observed_10perc.mat, key 'data_observed'), rows = cells, columns = genes
- Randomly selected ~2000-element subset of the observed expression values used to train the autoencoder

## Outputs

- Denoised reconstructed expression matrix So (same shape as the observed matrix, with zeros recovered)
- Latent (P=2) representation of the noisy data produced by the sigmoid encoder

## How to apply

Load the observed dropout-affected matrix (e.g. via scipy.io.loadmat on data_observed_10perc.mat, with a['data_observed']) and randomly select a 2000-element subset as the autoencoder's training input. Build a three-layer network (input, latent, output) with a logistic sigmoid encoder and linear decoder, latent dimension P=2, and train with the adam optimizer for at most 20 epochs at learning rate 1e-5 and batch size 64, using L2 weight decay of 0.05 and a sparse regularizer of 0.9 — the small subset and shallow latent space are what let SERM learn the underlying distribution cheaply and scalably (divide-and-conquer). The trained network reconstructs the denoised expression matrix So, whose value distribution (e.g. via a 100-bin normalized histogram) is then characterized by curve fitting to Gaussian, Rayleigh, or exponential PDFs. Judge the denoising stage by checking that So has the same cell-by-gene shape as the input, that the histogram of So values is smooth enough for stable PDF fitting, and that the best-fit lambda lands within its fitted bounds (0.01–1 for Gaussian/Rayleigh, 5–20 for exponential).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Framework whose first stage ('Denoising Via Auto-Encoding') trains the three-layer autoencoder to reconstruct denoised expression values that feed downstream histogram computation and PDF curve fitting) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import scipy.io as sio, serm as srm; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; out_SERM = srm.serm(dataObs, [6100, 1250], 0.5, randomize=False)
```

## Evaluation signals

- Output matrix So exists and matches the observed matrix's cell-by-gene dimensions (e.g. 12,162 cells × 2,422 genes for the Baryawno bone marrow stroma example)
- Training uses exactly the specified configuration: sigmoid encoder + linear decoder, latent dimension P=2, L2 regularizer 0.05, sparse regularizer 0.9, lr 1e-5, ≤20 epochs, batch 64, adam optimizer
- Subsequent histogram computation on So (100 bins) yields a smooth normalized histogram to which Gaussian, Rayleigh, and exponential PDFs can be fitted via Trust-Region with MSE cost
- The selected fitted lambda falls within its fitted bounds (0.01–1 for Gaussian/Rayleigh, 5–20 for exponential) and the chosen PDF has the minimum root mean square error
- End-to-end check: SERM-imputed data shows Pearson coefficient better than 0.9 for most genes at 5% sampling efficiency, far above baselines

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so on small genomic datasets the estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation values
- An implicit assumption is that ROI/data-subset coverage is sufficiently large so the captured distribution is representative of the entire matrix
- Batch effects are not considered explicitly in the SERM formulation, though it is generally less susceptible to data from different technologies or protocols

## Evidence

- [methods] The deep learning architecture (autoencoder) used in SERM consists of two parts, an encoder and a decoder.: "The deep learning architecture (autoencoder) used in SERM consists of two parts, an encoder and a decoder."
- [intro] The approach leverages a deep learning optimization to extract the latent representation of the data and reconstruct the denoised expression values: "The approach leverages a deep learning optimization to extract the latent representation of the data and reconstruct the denoised expression values (Fig. 1-step 1)."
- [abstract] Using a neural network, the technique first learns the underlying data distribution from a subset of the noisy data.: "Using a neural network, the technique first learns the underlying data distribution from a subset of the noisy data."
- [readme] Using deep learning, SERM first learns from a subset of the noisy gene expression data to estimate the underlying data distribution.: "Using deep learning, SERM first learns from a subset of the noisy gene expression data to estimate the underlying data distribution."
- [intro] Because of this divide-and-conquer strategy, SERM is scalable to large datasets and can impute the datasets much faster: "Because of this divide-and-conquer strategy, SERM is scalable to large datasets and can impute the datasets much faster than other analytical or deep learning-based methods."
