---
name: likelihood-ratio-model-selection
description: Decision support for choosing between NB and ZINB noise models before
  running DCA on a scRNA-seq count matrix, when you need to test whether a zero-inflation
  component is statistically justified.
when_to_use_negative:
- The count matrix has not been preprocessed (zero-expression cells and genes not
  yet removed), since the mean/dropout dependence would be distorted
- You are not deciding between nested NB and ZINB noise models — a likelihood-ratio
  test is only valid for nested model comparison
- The dataset is so small or sparse that gene-wise mean and empirical dropout-rate
  estimates are too unstable to fit a mean-dropout dependence
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: DCA
  role: Deep count autoencoder whose NB/ZINB noise models are being selected between;
    the fitted distributions define the likelihood used in the test
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework in which the NB and ZINB fits to the mean-dropout dependence are
    trained by minimizing binary cross-entropy
- name: TensorFlow
  role: Backend for the Keras autoencoder.loss machinery used to fit the noise models
- name: anndata
  role: Reads h5ad-formatted HDF5 count matrices that can serve as input to the preprocessing
    and fitting steps
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:a13dfb307f498209ef2f7a1f53c873f168fef05bc0eccc8eae9618ea5913cc07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# likelihood-ratio-model-selection

## Summary

A statistical model-selection procedure for DCA denoising that fits both negative binomial (NB) and zero-inflated NB (ZINB) noise models to the gene-wise mean-vs-dropout-rate dependence and uses a log-likelihood-ratio test on the nested fits to decide whether zero-inflation is required for a given scRNA-seq dataset.

## When to use

Decision support for choosing between NB and ZINB noise models before running DCA on a scRNA-seq count matrix, when you need to test whether a zero-inflation component is statistically justified.

## When NOT to use

- The count matrix has not been preprocessed (zero-expression cells and genes not yet removed), since the mean/dropout dependence would be distorted
- You are not deciding between nested NB and ZINB noise models — a likelihood-ratio test is only valid for nested model comparison
- The dataset is so small or sparse that gene-wise mean and empirical dropout-rate estimates are too unstable to fit a mean-dropout dependence

## Inputs

- Preprocessed scRNA-seq raw count matrix (CSV/TSV, genes x cells, with cell and gene labels)
- Gene-wise mean expression values
- Gene-wise empirical dropout rates (fraction of zero counts per gene)

## Outputs

- model_selection.csv with per-dataset likelihood-ratio test statistic, p-value, and favored noise model (NB or ZINB)
- Fitted NB and ZINB models to the mean-dropout-rate dependence

## How to apply

Load the preprocessed scRNA-seq count matrix (cells and genes with zero expression removed first) and compute, per gene, the mean expression and the empirical dropout rate (fraction of zero counts). Fit an NB model to the mean-vs-dropout-rate dependence by minimizing the binary cross-entropy between predicted zero probabilities and the observed zero indicator, using the autoencoder.loss machinery in Keras with the TensorFlow backend. Fit the ZINB model to the same relationship in the same way. Because ZINB nests NB, perform a log-likelihood-ratio test comparing the NB fit against the ZINB fit; a significant statistic (e.g. p < 0.05) indicates the zero-inflation component is required. Write the test statistic, p-value, and favored noise model (NB or ZINB) per dataset to model_selection.csv, then use the favored model as DCA's noise model.

## Related tools

- **DCA** (Deep count autoencoder whose NB/ZINB noise models are being selected between; the fitted distributions define the likelihood used in the test) — https://github.com/theislab/dca
- **Keras** (Framework in which the NB and ZINB fits to the mean-dropout dependence are trained by minimizing binary cross-entropy)
- **TensorFlow** (Backend for the Keras autoencoder.loss machinery used to fit the noise models)
- **anndata** (Reads h5ad-formatted HDF5 count matrices that can serve as input to the preprocessing and fitting steps) — https://github.com/theislab/anndata

## Examples

```
dca matrix.csv results  # after fitting NB and ZINB to mean-vs-dropout via Keras binary cross-entropy and running the LRT, inspect model_selection.csv
```

## Evaluation signals

- model_selection.csv exists and contains, for each dataset, a likelihood-ratio test statistic, a p-value, and a discrete noise-model label restricted to {'NB', 'ZINB'}
- The ZINB fit achieves lower (or equal) binary cross-entropy on the mean-vs-dropout relationship than the NB fit, consistent with the ZINB model being nested and more flexible
- The likelihood-ratio statistic is non-negative, as expected when comparing a nested (NB) against a more general (ZINB) fit
- The noise model finally used for DCA denoising matches the favored model recorded in model_selection.csv

## Limitations

- Hyperparameters were not fine-tuned for any of the analyses in the manuscript, so model selection inherits default settings that may not suit every dataset
- It may be difficult to determine when denoising improves scRNA-seq data, so a favorable test outcome does not guarantee downstream benefit
- The test compares fits to the gene-wise mean vs empirical dropout-rate dependence, not to the full count distribution, so it is a proxy for zero-inflation rather than a direct test on counts

## Evidence

- [discussion] NB-vs-ZINB noise-model selection via LRT: "we fit NB and ZINB models to the mean and empirical dropout rate dependence by minimizing the binary cross entropy"
- [abstract] DCA noise-model basis: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [discussion] Implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [discussion] Preprocessing prerequisite: "First, cells and genes with zero expression are removed from the count matrix."
- [discussion] Denoising output depends on the chosen noise model: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
