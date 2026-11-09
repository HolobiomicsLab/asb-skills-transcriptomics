---
name: scrna-seq-dropout-rate-analysis
description: Use when selecting between NB and ZINB noise models for DCA denoising
  of an scRNA-seq count matrix, i.e., when you must determine whether the observed
  zeros are fully explained by the mean-dropout relationship or require an explicit
  zero-inflation component.
when_to_use_negative:
- The count matrix has not been preprocessed (cells and genes with zero expression
  not removed), so gene-wise dropout rates are unreliable
- The goal is denoising output generation rather than noise-model selection — run
  DCA directly (denoising replaces original counts with the predicted negative binomial
  mean) instead of this test
- Data are not sparse count data (e.g., normalized/log-transformed expression), since
  the NB/ZINB mean-dropout relationship and BCE fitting assume raw counts
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DCA
  role: Provides the NB/ZINB noise models and likelihood-based loss machinery whose
    configuration this skill determines
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework used to fit NB and ZINB models to the mean-dropout dependence by
    minimizing binary cross-entropy via autoencoder.loss
- name: TensorFlow
  role: Keras backend executing the NB/ZINB fits
- name: anndata
  role: Reads h5ad-formatted HDF5 count matrices used as input
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:29210bf78caa3a31d31044258204f07895db8c4f0ff80197614a83475e89e0c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# scrna-seq-dropout-rate-analysis

## Summary

Fits NB and ZINB noise models to the gene-wise mean-vs-empirical-dropout-rate dependence by minimizing binary cross-entropy, then uses a log-likelihood-ratio test to decide whether zero-inflation is required when configuring DCA's noise model for a given scRNA-seq dataset.

## When to use

Use when selecting between NB and ZINB noise models for DCA denoising of an scRNA-seq count matrix, i.e., when you must determine whether the observed zeros are fully explained by the mean-dropout relationship or require an explicit zero-inflation component.

## When NOT to use

- The count matrix has not been preprocessed (cells and genes with zero expression not removed), so gene-wise dropout rates are unreliable
- The goal is denoising output generation rather than noise-model selection — run DCA directly (denoising replaces original counts with the predicted negative binomial mean) instead of this test
- Data are not sparse count data (e.g., normalized/log-transformed expression), since the NB/ZINB mean-dropout relationship and BCE fitting assume raw counts

## Inputs

- Preprocessed scRNA-seq raw count matrix (e.g., CSV/TSV gene x cell matrix with cell and gene labels, or h5ad via anndata)
- Gene-wise mean expression values
- Gene-wise empirical dropout rates (fraction of zero counts per gene)

## Outputs

- model_selection.csv with per-dataset likelihood-ratio test statistic, p-value, and favored noise model (NB or ZINB)
- Fitted NB and ZINB models of the mean-dropout-rate dependence

## How to apply

Load the preprocessed scRNA-seq count matrix and compute, per gene, the mean expression and the empirical dropout rate (fraction of zero counts). Fit an NB model and a ZINB model to this mean-vs-dropout-rate dependence by minimizing the binary cross-entropy between predicted zero probabilities and the observed zero indicator, using DCA's Keras/TensorFlow autoencoder.loss machinery. Because ZINB is nested within NB, perform a log-likelihood-ratio test: if the statistic is significant (the zero-inflation component is required), select ZINB; otherwise select NB. Write the test statistic, p-value, and favored model per dataset to model_selection.csv. Rationale: DCA's reconstruction error is the likelihood of the noise-model distribution, so the noise model choice directly governs denoising quality.

## Related tools

- **DCA** (Provides the NB/ZINB noise models and likelihood-based loss machinery whose configuration this skill determines) — https://github.com/theislab/dca
- **Keras** (Framework used to fit NB and ZINB models to the mean-dropout dependence by minimizing binary cross-entropy via autoencoder.loss)
- **TensorFlow** (Keras backend executing the NB/ZINB fits)
- **anndata** (Reads h5ad-formatted HDF5 count matrices used as input) — https://github.com/theislab/anndata

## Examples

```
python fit_noise_models.py --counts matrix.csv --models NB,ZINB --lrt --out model_selection.csv
```

## Evaluation signals

- model_selection.csv exists and contains a test statistic, p-value, and a binary model decision (NB or ZINB) for each analyzed dataset
- The ZINB fit is nested within the NB fit, so the likelihood-ratio statistic is non-negative and its p-value is in [0, 1]
- Empirical dropout rates computed per gene are in [0, 1] and inversely track gene mean expression, consistent with the mean-dropout dependence being fit
- The binary cross-entropy loss decreases during fitting of both NB and ZINB models on the same mean-vs-dropout data
- The chosen noise model matches DCA's supported options (negative binomial with or without zero-inflation)

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, so a data-driven noise-model choice may not guarantee downstream improvement
- Evaluation of denoising is difficult because defining ground truth for real data is challenging
- Adequate hyperparameters (e.g., bottleneck dimensionality) matter; poor choices may lead to overimputation, and hyperparameters were not fine-tuned for any analysis in the manuscript

## Evidence

- [discussion] Model fitting of NB and ZINB to mean-dropout rate dependence via minimizing BCE for noise model selection: "we fit NB and ZINB models to the mean and empirical dropout rate dependence by minimizing the binary cross entropy"
- [abstract] DCA noise models with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [intro] Reconstruction error is the noise-model likelihood: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] DCA implemented in Keras with TensorFlow backend: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [readme] Input requires gene x cell raw count matrix with labels: "where `matrix.csv` is a CSV/TSV-formatted raw count matrix with genes in rows and cells in columns. Cell and gene labels are mandatory."
