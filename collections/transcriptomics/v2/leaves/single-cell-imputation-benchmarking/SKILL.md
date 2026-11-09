---
name: single-cell-imputation-benchmarking
description: Use when you need to compare scRNA-Seq imputation/denoising methods quantitatively
  — e.g. simulated counts with known true values and group labels across noise types
  such as dropout, uniform, Gaussian, Gamma, Poisson, negative binomial.
when_to_use_negative:
- Input is real scRNA-Seq data with no simulated truth counts or known group labels
  — MSE against true values and ASW recovery of known labels cannot be computed
- Input is a single noise type only (e.g. dropouts alone) when the goal is to test
  distribution-agnostic cleaning across uniform, Gaussian, Gamma, Poisson, and negative
  binomial noise
- The matrix is not a gene x cell count matrix from a scRNA-Seq experiment (e.g. bulk
  RNA-Seq or precomputed feature tables)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: AutoClass
  role: Imputes each noisy simulated dataset via AutoClassImpute with dropout_rate=0.3;
    the method under test
  repo: https://github.com/datapplab/AutoClass
- name: Splatter
  role: Simulates the true count matrix (1000 genes, 500 cells, six groups) used to
    derive all noisy benchmark datasets
- name: DCA
  role: Control imputation method (zero-inflated negative binomial autoencoder) benchmarked
    against AutoClass
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Control imputation method benchmarked against AutoClass on simulated noisy
    data
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: scImpute
  role: Control imputation method (Gaussian-Gamma mixture model) benchmarked against
    AutoClass
  repo: https://github.com/Vivianstats/scImpute
- name: SAVER
  role: Control imputation method (negative binomial regression with empirical Bayes)
    benchmarked against AutoClass
  repo: https://github.com/mohuangx/SAVER
- name: TensorFlow
  role: Runtime dependency for AutoClassImpute (>=2.0) in the Python environment
- name: numpy
  role: Python dependency for count-matrix handling during imputation and metric computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:d47347109e878edea79a998bfc8d7696498c511b21c8e515bff7eca95acbec38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - SAVER
  - Splatter
  - TensorFlow
  - numpy
  - scImpute
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-imputation-benchmarking

## Summary

Benchmark scRNA-Seq denoising/imputation methods by simulating noisy count matrices with Splatter, imputing them with AutoClassImpute and controls (DCA, MAGIC, scImpute, SAVER), and quantifying recovery via MSE against true counts and average silhouette width (ASW) against known group labels.

## When to use

Use when you need to compare scRNA-Seq imputation/denoising methods quantitatively — e.g. simulated counts with known true values and group labels across noise types such as dropout, uniform, Gaussian, Gamma, Poisson, negative binomial.

## When NOT to use

- Input is real scRNA-Seq data with no simulated truth counts or known group labels — MSE against true values and ASW recovery of known labels cannot be computed
- Input is a single noise type only (e.g. dropouts alone) when the goal is to test distribution-agnostic cleaning across uniform, Gaussian, Gamma, Poisson, and negative binomial noise
- The matrix is not a gene x cell count matrix from a scRNA-Seq experiment (e.g. bulk RNA-Seq or precomputed feature tables)

## Inputs

- Simulated true scRNA-Seq count matrix from Splatter (1000 genes x 500 cells, six groups per Supplementary Table 2)
- Noisy derivative count matrices (dropout, uniform, Gaussian, Gamma, Poisson, negative binomial noise) sharing the same true data
- Known group/cell-type labels for simulated cells

## Outputs

- denoising_benchmark_results.csv with ASW recovery, MSE, and dropout zero-recovery values per method per dataset
- t-SNE (2-D) embeddings from PCA (n_components=50) of raw, noisy, and imputed matrices
- MSE between true and imputed expression values for Datasets 3-7 and average recovered value at dropout zero positions for Dataset 2

## How to apply

Simulate a true count matrix with Splatter (1000 genes, 500 cells, six groups per Supplementary Table 2), then derive noisy datasets by applying dropout noise (Dataset 2) and uniform, Gaussian, Gamma, Poisson, and negative binomial noise (Datasets 3-7) to the same true data. Impute each noisy dataset with AutoClassImpute (dropout_rate=0.3) and the control methods DCA, MAGIC, scImpute, and SAVER, keeping the raw noisy data as baseline. For each imputed dataset, normalize with take_norm, reduce with PCA (n_components=50) followed by t-SNE (n_components=2), and compute average silhouette width (ASW) recovery of the known group labels; for Datasets 3-7 compute the mean squared error (MSE) between true and imputed expression values, and for Dataset 2 compute the average recovered value at dropout zero positions versus the average true value at those positions. Assemble ASW recovery, MSE, and zero-recovery values into a single table (denoising_benchmark_results.csv); the design is grounded in AutoClass being distribution agnostic, so it should be evaluated across noise distributions rather than dropouts alone.

## Related tools

- **AutoClass** (Imputes each noisy simulated dataset via AutoClassImpute with dropout_rate=0.3; the method under test) — https://github.com/datapplab/AutoClass
- **Splatter** (Simulates the true count matrix (1000 genes, 500 cells, six groups) used to derive all noisy benchmark datasets)
- **DCA** (Control imputation method (zero-inflated negative binomial autoencoder) benchmarked against AutoClass) — https://github.com/theislab/dca
- **MAGIC** (Control imputation method benchmarked against AutoClass on simulated noisy data) — https://github.com/KrishnaswamyLab/MAGIC
- **scImpute** (Control imputation method (Gaussian-Gamma mixture model) benchmarked against AutoClass) — https://github.com/Vivianstats/scImpute
- **SAVER** (Control imputation method (negative binomial regression with empirical Bayes) benchmarked against AutoClass) — https://github.com/mohuangx/SAVER
- **TensorFlow** (Runtime dependency for AutoClassImpute (>=2.0) in the Python environment)
- **numpy** (Python dependency for count-matrix handling during imputation and metric computation)

## Examples

```
res = AutoClassImpute(X,dropout_rate=0.3)
```

## Evaluation signals

- Denoising_benchmark_results.csv reproduces the Fig. 2d-f patterns: ASW recovery values fall in the 0.2-0.5 range
- AutoClass attains the smallest MSE between true and imputed expression values among the five compared methods on Datasets 3-7
- AutoClass recovers the six-group cell-type pattern that DCA, MAGIC, scImpute, and SAVER fail to recover in the t-SNE embeddings
- For Dataset 2, the average recovered value at dropout zero positions is compared against the average true value at those positions to quantify dropout recovery
- Each method was run on the same simulated true matrix (1000 genes, 500 cells, six groups) with identical noise derivations, so per-dataset results are directly comparable

## Limitations

- Existing denoising/imputation methods benchmarked here largely focus on a single noise type (dropouts) and have strong distribution assumptions, so their results are conditioned on those assumptions
- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced but not included in the provided text, so exact simulation parameterization may need to be retrieved separately
- ASW is computed on t-SNE embeddings derived from PCA (n_components=50), so clustering-quality recovery values depend on that reduction chain

## Evidence

- [abstract] AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [intro] we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] res = AutoClassImpute(X,dropout_rate=0.3): "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] PCA_raw = PCA(n_components=50).fit_transform(X_norm): "PCA_raw = PCA(n_components=50).fit_transform(X_norm)"
- [readme] AutoClass integrates two deep neural network components, an autoencoder and a classifier, as to maximize both noise removal and signal retention.: "AutoClass integrates two deep neural network components, an autoencoder and a classifier, as to maximize both noise removal and signal retention."
