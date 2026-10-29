---
name: neural-network-early-stopping-training
description: Use when training the DCA autoencoder (64-32-64 hidden layers, 32-dim
  bottleneck, ZINB loss) on raw scRNA-seq counts and you need a validation-driven
  early-stopping and learning-rate-reduction schedule.
when_to_use_negative:
- Input is not raw counts (e.g., log-normalized or batch-corrected data), since the
  ZINB/NB noise-model likelihood assumes a sparse count structure
- You only need a fixed, pre-trained transformation rather than fitting an autoencoder
  to a new dataset
- Denoising with simpler methods is sufficient and the count structure/overdispersion
  does not need modeling
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: DCA
  role: Deep count autoencoder whose training loop implements the early-stopping and
    learning-rate-reduction schedule
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework used to compile and train the autoencoder with early-stopping and
    ReduceLROnPlateau-style callbacks
- name: TensorFlow
  role: Backend behind Keras for the DCA training
- name: kopt
  role: Hyper-parameter optimization library (TPE via hyperopt) used to search 1000
    DCA configurations and select lowest reconstruction error
  repo: https://github.com/Avsecz/kopt
- name: anndata
  role: Reader for h5ad-formatted HDF5 count matrices fed into DCA training
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:1e97fdbb544ba407296dd7129cf52fdd6196c519f087b588456ead7eb7303d2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  - kopt
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# neural-network-early-stopping-training

## Summary

Training-protocol skill for the DCA deep count autoencoder: train the ZINB/NB-loss autoencoder with RMSProp (learning rate 0.001), halve the learning rate by 0.1 after 20 stagnant epochs, and terminate via early stopping after 25 epochs without validation-loss improvement to prevent overfitting/overimputation of scRNA-seq data.

## When to use

Use when training the DCA autoencoder (64-32-64 hidden layers, 32-dim bottleneck, ZINB loss) on raw scRNA-seq counts and you need a validation-driven early-stopping and learning-rate-reduction schedule.

## When NOT to use

- Input is not raw counts (e.g., log-normalized or batch-corrected data), since the ZINB/NB noise-model likelihood assumes a sparse count structure
- You only need a fixed, pre-trained transformation rather than fitting an autoencoder to a new dataset
- Denoising with simpler methods is sufficient and the count structure/overdispersion does not need modeling

## Inputs

- Raw scRNA-seq count matrix (CSV/TSV, genes in rows and cells in columns, with mandatory cell and gene labels, or h5ad-formatted HDF5 via anndata)
- Zero-expression cell/gene filtered and optionally HVG-restricted count matrix (e.g., top 1000 highly variable genes via Scanpy filter_genes_dispersion)

## Outputs

- Early-stopped trained DCA model with converged ZINB parameters
- mean.tsv: denoised matrix (mean parameter of the ZINB distribution), same gene x cell dimensions as input (minus zero-expression features)
- pi.tsv and dispersion.tsv (dropout probability and dispersion per cell and gene); reduced.tsv (32-dimensional bottleneck activations); mean_norm.tsv (library-size-normalized output)

## How to apply

Compile the DCA network (three hidden layers of 64, 32, 64 neurons with a 32-dimensional bottleneck and ZINB output heads: mu via exponential, pi via sigmoid, theta dispersion) with the ZINB negative-log-likelihood reconstruction loss plus a ridge prior on pi. Use RMSProp for optimization with learning rate 0.001, implemented in Python 3 via Keras with a TensorFlow backend. If validation loss does not improve for 20 epochs, multiply the learning rate by 0.1; stop training entirely after 25 epochs without validation-loss improvement (early stopping). Optionally run hyperparameter search over 1000 configurations sampled from specified ranges and optimized via TPE, selecting the configuration with the lowest reconstruction error; the DCA README exposes this through the --hyper option.

## Related tools

- **DCA** (Deep count autoencoder whose training loop implements the early-stopping and learning-rate-reduction schedule) — https://github.com/theislab/dca
- **Keras** (Framework used to compile and train the autoencoder with early-stopping and ReduceLROnPlateau-style callbacks)
- **TensorFlow** (Backend behind Keras for the DCA training)
- **kopt** (Hyper-parameter optimization library (TPE via hyperopt) used to search 1000 DCA configurations and select lowest reconstruction error) — https://github.com/Avsecz/kopt
- **anndata** (Reader for h5ad-formatted HDF5 count matrices fed into DCA training) — https://github.com/theislab/anndata

## Examples

```
dca matrix.csv results --hyper
```

## Evaluation signals

- Training terminates via the early-stopping criterion (25 epochs without validation-loss improvement), not by exhausting a fixed epoch budget
- Learning rate is multiplied by 0.1 only after 20 epochs without validation-loss improvement
- Resulting mean.tsv has the same gene x cell dimensions as the (filtered) input count matrix
- Validation loss reflects ZINB negative-log-likelihood reconstruction error (the noise-model likelihood), and the hyperparameter-selected run has the lowest reconstruction error among 1000 TPE-sampled configurations
- Pipeline completes on the raw count matrix with cells and genes with zero expression removed beforehand

## Limitations

- Hyperparameters were not fine-tuned for any analysis described in the manuscript; default schedule (RMSProp, lr 0.001, 20/25-epoch patience) may be suboptimal for new datasets
- Inadequate hyperparameter choices such as a too low-dimensional bottleneck layer can lead to overimputation
- It may be difficult to determine when denoising improves scRNA-seq data, since defining a ground truth for real data is challenging

## Evidence

- [discussion] RMSProp schedule with LR reduction on stagnation: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [intro] Loss defined as noise-model likelihood: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Denoised output is the NB/ZINB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [readme] README: mean.tsv as main output: "mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution. This file has the same dimensions as the input file"
- [readme] README: hyperparameter search entry point: [evidence span withheld — anchor status: external_doc]
