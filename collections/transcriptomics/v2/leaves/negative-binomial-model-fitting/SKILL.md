---
name: negative-binomial-model-fitting
description: Decision support for choosing between NB and ZINB noise models before
  denoising an scRNA-seq count matrix with DCA, when you need to test whether the
  zero-inflation component is required for your data.
when_to_use_negative:
- The input is not raw count data (e.g. already normalized or log-transformed), since
  dropout-rate/mean dependence and count-distribution likelihoods assume raw counts
- The dataset's zero-expression genes and cells have not been removed, which the DCA
  preprocessing requires before fitting
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DCA
  role: Deep count autoencoder whose NB/ZINB loss functions define the noise models
    being compared for denoising
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework in which the NB and ZINB fits minimize binary cross-entropy via
    the autoencoder.loss machinery
- name: TensorFlow
  role: Backend used by Keras for the NB/ZINB model fitting
- name: anndata
  role: I/O of h5ad-formatted count matrices used as model-fitting input
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:739aa46c1375e433538c2935c1c3e63d2bc72cde82345cc2e336c1533f07099e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# negative-binomial-model-fitting

## Summary

Fit negative binomial (NB) and zero-inflated NB (ZINB) noise models to the gene-wise mean-vs-empirical-dropout-rate dependence by minimizing binary cross-entropy, then use a log-likelihood-ratio test between the nested NB and ZINB fits to decide whether zero inflation is present and which noise model DCA should use for a given scRNA-seq dataset.

## When to use

Decision support for choosing between NB and ZINB noise models before denoising an scRNA-seq count matrix with DCA, when you need to test whether the zero-inflation component is required for your data.

## When NOT to use

- The input is not raw count data (e.g. already normalized or log-transformed), since dropout-rate/mean dependence and count-distribution likelihoods assume raw counts
- The dataset's zero-expression genes and cells have not been removed, which the DCA preprocessing requires before fitting

## Inputs

- Preprocessed scRNA-seq count matrix (genes x cells; CSV/TSV or h5ad, with cell and gene labels)
- Gene-wise mean expression values
- Gene-wise empirical dropout rates (fraction of zero counts per gene)

## Outputs

- model_selection.csv containing the likelihood-ratio test statistic, p-value, and favored noise model (NB or ZINB) per dataset
- Selected DCA noise model (NB or ZINB) to use for denoising

## How to apply

From the preprocessed scRNA-seq count matrix, compute for each gene the mean expression and the empirical dropout rate (fraction of zero counts). Fit an NB model to the mean-vs-dropout-rate dependence by minimizing the binary cross-entropy between predicted zero probabilities and the observed zero indicator, using the autoencoder.loss machinery in Keras with the TensorFlow backend; then fit the ZINB model to the same relationship the same way. Because ZINB nests NB, perform a log-likelihood-ratio test comparing the NB fit against the ZINB fit to test whether the zero-inflation component is required. Write the test statistic, p-value, and the favored noise model (NB or ZINB) per dataset to model_selection.csv, and pass the favored distribution as the DCA noise model.

## Related tools

- **DCA** (Deep count autoencoder whose NB/ZINB loss functions define the noise models being compared for denoising) — https://github.com/theislab/dca
- **Keras** (Framework in which the NB and ZINB fits minimize binary cross-entropy via the autoencoder.loss machinery)
- **TensorFlow** (Backend used by Keras for the NB/ZINB model fitting)
- **anndata** (I/O of h5ad-formatted count matrices used as model-fitting input) — https://github.com/theislab/anndata

## Examples

```
from dca import DCALoss; loss = DCALoss(zinb=False)  # fit NB vs ZINB to mean-vs-dropout via autoencoder.loss in Keras, then LRT to pick the model
```

## Evaluation signals

- model_selection.csv exists with one row per dataset and columns for the likelihood-ratio test statistic, p-value, and favored model (NB or ZINB)
- The NB fit must be nested within the ZINB fit, so the likelihood-ratio statistic is non-negative and asymptotically chi-square distributed under the null of no zero inflation
- Binary cross-entropy between predicted zero probabilities and the observed zero indicator decreases during both the NB and ZINB fits
- The chosen noise model reproduces the observed gene-wise mean-vs-empirical-dropout-rate dependence in the count matrix
- The downstream DCA run uses the selected distribution and its mean.tsv output retains the input dimensions (excluding zero-expression genes/cells)

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data in general, and defining ground truth for real data is challenging
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript, so default settings may not be optimal for the model fits
- Overimputation can occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer

## Evidence

- [discussion] NB/ZINB fitting procedure: "we fit NB and ZINB models to the mean and empirical dropout rate dependence by minimizing the binary cross entropy"
- [abstract] Noise model choice in DCA: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [discussion] Implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [intro] Reconstruction error definition: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Preprocessing prerequisite: "First, cells and genes with zero expression are removed from the count matrix."
