---
name: zero-inflated-count-model-fitting
description: Decision support for selecting between NB and ZINB noise models before
  denoising a scRNA-seq count matrix, when sparsity/dropout structure suggests possible
  zero-inflation.
when_to_use_negative:
- Input is not a raw count matrix (e.g. already normalized, log-transformed, or imputed
  data), since dropout-rate and count-distribution assumptions no longer hold
- The dataset has already been denoised or imputed, so the empirical zero counts no
  longer reflect the dropout process being modeled
- Only denoised mean output is needed without a principled model-choice decision —
  DCA's default noise model may suffice
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: DCA
  role: Provides the NB/ZINB noise-model framework whose loss is selected via the
    likelihood-ratio test; implemented in Python 3 using Keras with a TensorFlow backend
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework whose autoencoder.loss machinery is used to fit NB and ZINB models
    to the mean-vs-dropout-rate dependence by minimizing binary cross-entropy
- name: TensorFlow
  role: Backend used with Keras for fitting the noise models
- name: anndata
  role: Supported h5ad-formatted HDF5 input format for the DCA pipeline surrounding
    the model-selection step
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:fb833e886dce9ba145be55eb6dcd49a6a679a55eb4e923ecd79b5273e7d3679e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - TensorFlow
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# zero-inflated-count-model-fitting

## Summary

Fits negative binomial (NB) and zero-inflated negative binomial (ZINB) noise models to the gene-wise mean vs empirical dropout-rate dependence and uses a likelihood-ratio test to decide whether zero-inflation is required, as done in DCA for scRNA-seq count denoising.

## When to use

Decision support for selecting between NB and ZINB noise models before denoising a scRNA-seq count matrix, when sparsity/dropout structure suggests possible zero-inflation.

## When NOT to use

- Input is not a raw count matrix (e.g. already normalized, log-transformed, or imputed data), since dropout-rate and count-distribution assumptions no longer hold
- The dataset has already been denoised or imputed, so the empirical zero counts no longer reflect the dropout process being modeled
- Only denoised mean output is needed without a principled model-choice decision — DCA's default noise model may suffice

## Inputs

- Preprocessed scRNA-seq raw count matrix (CSV/TSV, genes x cells, with cell and gene labels; zero-expression cells and genes removed)
- Gene-wise mean expression values
- Gene-wise empirical dropout rates (fraction of zero counts)

## Outputs

- model_selection.csv with likelihood-ratio test statistic, p-value, and favored noise model (NB or ZINB) per dataset
- Selected noise model (NB or ZINB) to configure the DCA loss function

## How to apply

Load the preprocessed scRNA-seq count matrix and compute, for each gene, the mean expression and the empirical dropout rate (fraction of zero counts). Fit an NB model to the mean-vs-dropout-rate dependence by minimizing binary cross-entropy between predicted zero probabilities and the observed zero indicator, using the autoencoder.loss machinery in Keras with the TensorFlow backend; then fit the ZINB model to the same relationship the same way. Because the NB model is nested within the ZINB model, perform a log-likelihood-ratio test comparing the two fits to decide whether the zero-inflation component is required. Record the test statistic, p-value, and the favored noise model (NB or ZINB) per dataset, then use the favored model as the DCA noise model for denoising. This is justified because DCA defines the reconstruction error as the likelihood of the noise-model distribution, so the choice of NB vs ZINB directly determines the loss used for training.

## Related tools

- **DCA** (Provides the NB/ZINB noise-model framework whose loss is selected via the likelihood-ratio test; implemented in Python 3 using Keras with a TensorFlow backend) — https://github.com/theislab/dca
- **Keras** (Framework whose autoencoder.loss machinery is used to fit NB and ZINB models to the mean-vs-dropout-rate dependence by minimizing binary cross-entropy)
- **TensorFlow** (Backend used with Keras for fitting the noise models)
- **anndata** (Supported h5ad-formatted HDF5 input format for the DCA pipeline surrounding the model-selection step) — https://github.com/theislab/anndata

## Examples

```
python -c "import pandas as pd, numpy as np; X = pd.read_csv('matrix.csv', index_col=0); mu = X.mean(1); drop = (X == 0).mean(1); print(pd.DataFrame({'mean': mu, 'dropout': drop}).to_csv('mean_dropout.csv'))"
```

## Evaluation signals

- The NB fit must be strictly nested within the ZINB fit so that the log-likelihood-ratio test statistic is valid (non-negative, asymptotically chi-square distributed under the null)
- model_selection.csv exists and contains one row per dataset with a test statistic, a p-value, and a discrete model label limited to {NB, ZINB}
- Fitted zero probabilities from the NB/ZINB models should track the empirical dropout-rate vs mean curve in the data (low binary cross-entropy)
- The downstream DCA training loss must be the model favored by the test (NB loss or ZINB loss), consistent with model_selection.csv
- A non-significant p-value (e.g. p > 0.05) should map to NB; a significant p-value indicating a zero-inflation component should map to ZINB

## Limitations

- In general it may be difficult to determine when denoising improves scRNA-seq data; evaluation is difficult because defining a ground truth for real data is challenging
- Overimputation may occur with inadequate hyperparameter choices such as too low-dimensional a bottleneck layer, affecting the fitted noise model's usefulness
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript, so default settings may not transfer to every dataset

## Evidence

- [discussion] NB/ZINB fitting via BCE on mean-dropout dependence: "we fit NB and ZINB models to the mean and empirical dropout rate dependence by minimizing the binary cross entropy"
- [abstract] DCA uses NB noise model with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [intro] Reconstruction error is noise-model likelihood: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] DCA implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [readme] Input format for DCA runs: "where `matrix.csv` is a CSV/TSV-formatted raw count matrix with genes in rows and cells in columns. Cell and gene labels are mandatory."
