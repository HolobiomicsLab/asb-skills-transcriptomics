---
name: scrna-seq-noise-simulation
description: 'Use when benchmarking a scRNA-Seq denoising/imputation method against
  controls: you need a noisy count matrix with known ground-truth expression and cell
  labels, or a real dataset (e.g., Tian et al. 305-cell mixture) degraded with dropout
  or negative binomial noise.'
when_to_use_negative:
- The input is already clean, imputed, or normalized expression data with no noise
  layer added — there is no ground-truth/noisy pair to evaluate recovery against
- The goal is biological discovery on real data rather than method benchmarking —
  injecting simulated noise would distort genuine signal
- You lack known cell-type labels or a true count matrix, so ASW/MSE-based recovery
  cannot be validated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: 'Generates the true simulated scRNA-Seq count matrix (Dataset 1: 1000 genes,
    500 cells, six groups) that noise types are applied to'
- name: AutoClass
  role: Method under test; AutoClassImpute is run on each noisy matrix (dropout_rate=0.3)
    to evaluate recovery
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control denoising method (zero-inflated negative binomial autoencoder) compared
    on each simulated noisy dataset
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Control imputation method compared on each simulated noisy dataset
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: scImpute
  role: Control imputation method (Gaussian-Gamma mixture model) compared on each
    simulated noisy dataset
  repo: https://github.com/Vivianstats/scImpute
- name: SAVER
  role: Control imputation method (negative binomial regression with empirical Bayes)
    compared on each simulated noisy dataset
  repo: https://github.com/mohuangx/SAVER
- name: Python 3
  role: Runtime for noise application, imputation calls, and ASW/MSE evaluation scripts
- name: TensorFlow >=2.0
  role: Dependency required to run AutoClass's neural network imputation on the noisy
    matrices
- name: numpy
  role: Array manipulation for noise injection and metric computation in Python
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:ac563b6d779dd5d5d1e5546a120a72b580519e124497ff60f6302005dd1f599a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - Python 3
  - SAVER
  - Splatter
  - TensorFlow >=2.0
  - numpy
  - scImpute
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# scrna-seq-noise-simulation

## Summary

Simulates controlled noise (dropout, uniform, Gaussian, Gamma, Poisson, negative binomial) onto scRNA-Seq count matrices — via Splatter-simulated truth or real datasets like the Tian et al. sc_mixology cell mixtures — to benchmark whether denoising methods (AutoClass vs. DCA, SAVER, scImpute, MAGIC) recover true expression and cell-type structure.

## When to use

Use when benchmarking a scRNA-Seq denoising/imputation method against controls: you need a noisy count matrix with known ground-truth expression and cell labels, or a real dataset (e.g., Tian et al. 305-cell mixture) degraded with dropout or negative binomial noise.

## When NOT to use

- The input is already clean, imputed, or normalized expression data with no noise layer added — there is no ground-truth/noisy pair to evaluate recovery against
- The goal is biological discovery on real data rather than method benchmarking — injecting simulated noise would distort genuine signal
- You lack known cell-type labels or a true count matrix, so ASW/MSE-based recovery cannot be validated

## Inputs

- True simulated count matrix from Splatter (e.g., 1000 genes x 500 cells, six groups; Dataset1_true.csv / truecounts.csv)
- Real scRNA-Seq count matrix with cell-type labels (e.g., Tian et al. sc_mixology sincell_with_class.RData, labels in colData column cell_line_demuxlet)
- Noisy count matrix CSV (e.g., Dataset1_raw.csv) with dropout or distributional noise applied

## Outputs

- Noisy count matrices per noise type (dropout, uniform, Gaussian, Gamma, Poisson, negative binomial)
- Imputed matrices from each compared method plus raw-noisy baseline
- Results table (e.g., denoising_benchmark_results.csv or asw_results.csv) with ASW recovery, MSE, and dropout-zero recovery values per method and noise condition

## How to apply

Simulate a true count matrix with Splatter (e.g., 1000 genes, 500 cells, six groups per Supplementary Table 2), then derive noisy variants by applying dropout noise and uniform, Gaussian, Gamma, Poisson, or negative binomial noise to the same truth; alternatively, take a real matrix (e.g., Tian et al. sc_mixology, 305 cells, 13,426 genes, top 1000 or 5000 highly variable genes) and inject dropout events or negative binomial noise. Run the method under test (e.g., AutoClassImpute with dropout_rate=0.3) alongside controls (DCA, MAGIC, scImpute, SAVER), keeping the raw noisy matrix as a baseline. Evaluate recovery by normalizing with take_norm, reducing with PCA to 50 components, embedding with t-SNE to 2 dimensions, and computing average silhouette width (ASW) on known group labels; for noise types other than dropout, also compute the MSE between true and imputed values, and for dropout compute the average recovered value at zero positions versus the average true value there. Success is AutoClass attaining the smallest MSE among compared methods, ASW recovery in the 0.2-0.5 range, and the highest ASW across gene subsets and noise types.

## Related tools

- **Splatter** (Generates the true simulated scRNA-Seq count matrix (Dataset 1: 1000 genes, 500 cells, six groups) that noise types are applied to)
- **AutoClass** (Method under test; AutoClassImpute is run on each noisy matrix (dropout_rate=0.3) to evaluate recovery) — https://github.com/datapplab/AutoClass
- **DCA** (Control denoising method (zero-inflated negative binomial autoencoder) compared on each simulated noisy dataset) — https://github.com/theislab/dca
- **MAGIC** (Control imputation method compared on each simulated noisy dataset) — https://github.com/KrishnaswamyLab/MAGIC
- **scImpute** (Control imputation method (Gaussian-Gamma mixture model) compared on each simulated noisy dataset) — https://github.com/Vivianstats/scImpute
- **SAVER** (Control imputation method (negative binomial regression with empirical Bayes) compared on each simulated noisy dataset) — https://github.com/mohuangx/SAVER
- **Python 3** (Runtime for noise application, imputation calls, and ASW/MSE evaluation scripts)
- **TensorFlow >=2.0** (Dependency required to run AutoClass's neural network imputation on the noisy matrices)
- **numpy** (Array manipulation for noise injection and metric computation in Python)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass attains the smallest mean squared error between true and imputed values across noise types (Datasets 3-7)
- ASW recovery values fall in the 0.2-0.5 range and reproduce the Fig. 2d-f patterns
- On the dropout dataset (Dataset 2), the average recovered value at dropout zero positions approaches the average true value at those positions
- On the Tian et al. benchmark, AutoClass achieves the highest ASW on both 1000-gene and 5000-gene subsets and both dropout and negative binomial noise conditions
- t-SNE embeddings of AutoClass-imputed data recover the known group/cell-line labels while control methods fail to recover cell-type pattern

## Limitations

- Existing methods largely focus on dropouts and assume specific distributions; benchmark conclusions depend on whether the injected noise matches each method's assumptions
- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced but not included in the provided text, so exact simulation parameter values must be obtained from the supplement
- No software version numbers are stated, so reproducibility of the simulation environment is only partially specified (Python 3, TensorFlow >=2.0)

## Evidence

- [intro] Splatter simulation with six groups and 1000 genes, 500 cells: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [abstract] AutoClass is distribution agnostic across noise types: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [abstract] Motivation: existing methods only handle one noise type: "Existing denoising and imputation methods largely focus on a single type of noise (i.e., dropouts) and have strong distribution assumptions"
- [results] Imputation call with dropout rate used in the benchmark: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [readme] Tian et al. labels available in sc_mixology colData: [evidence span withheld — anchor status: external_doc]
