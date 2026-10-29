---
name: dropout-and-distributional-noise-injection
description: 'Use when benchmarking imputation/denoising methods against known ground
  truth: you need noisy variants of a simulated true count matrix (1000 genes, 500
  cells, six groups) spanning dropout and five noise distributions to compute MSE
  and ASW recovery.'
when_to_use_negative:
- Input is real experimental scRNA-Seq data with no ground truth — MSE against true
  values cannot be computed; use clustering/batch-effect benchmarks instead
- The method under test is explicitly designed for a single noise type and the evaluation
  goal is limited to that noise (e.g., pure dropout correction only)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3524
tools:
- name: Splatter
  role: Simulates the true count matrix (1000 genes, 500 cells, six groups) used as
    ground truth before noise injection
- name: AutoClass
  role: Primary imputation method under benchmark; run as AutoClassImpute with dropout_rate=0.3
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control denoising method assuming zero-inflated negative binomial distribution
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Control imputation method
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: scImpute
  role: Control imputation method using a Gaussian-Gamma mixture model
  repo: https://github.com/Vivianstats/scImpute
- name: SAVER
  role: Control imputation method assuming negative binomial distribution
  repo: https://github.com/mohuangx/SAVER
- name: Python 3 / TensorFlow
  role: Runtime for AutoClassImpute and downstream normalization/PCA/t-SNE evaluation
    code
  repo: https://github.com/datapplab/AutoClass
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:93b3cfd07faa4fbf62c551ed8bc877234a4b991201a97cffc452468b03199ee1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - Python 3 / TensorFlow
  - SAVER
  - Splatter
  - numpy
  - scImpute
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# dropout-and-distributional-noise-injection

## Summary

Generate benchmark scRNA-Seq datasets by applying dropout noise plus five distributional noise types (uniform, Gaussian, Gamma, Poisson, negative binomial) to a Splatter-simulated true count matrix, enabling distribution-agnostic evaluation of denoising/imputation methods such as AutoClass against DCA, MAGIC, scImpute, and SAVER.

## When to use

Use when benchmarking imputation/denoising methods against known ground truth: you need noisy variants of a simulated true count matrix (1000 genes, 500 cells, six groups) spanning dropout and five noise distributions to compute MSE and ASW recovery.

## When NOT to use

- Input is real experimental scRNA-Seq data with no ground truth — MSE against true values cannot be computed; use clustering/batch-effect benchmarks instead
- The method under test is explicitly designed for a single noise type and the evaluation goal is limited to that noise (e.g., pure dropout correction only)

## Inputs

- Splatter-simulated true count matrix (1000 genes x 500 cells, six groups, Supplementary Table 2 parameters)
- Noise type specification: dropout, uniform, Gaussian, Gamma, Poisson, negative binomial

## Outputs

- Datasets 2-7: noisy count matrices derived from one true matrix
- Imputed matrices from AutoClass, DCA, MAGIC, scImpute, SAVER plus raw noisy baseline
- denoising_benchmark_results.csv with ASW recovery, MSE, and dropout zero-recovery values

## How to apply

Simulate a ground-truth count matrix with Splatter (1000 genes, 500 cells, six groups, parameters per Supplementary Table 2). Derive Dataset 2 by applying dropout noise to the true data, and Datasets 3-7 by applying uniform, Gaussian, Gamma, Poisson, and negative binomial noise respectively to the same true matrix so all noisy variants share one ground truth. Impute each noisy dataset with AutoClassImpute (dropout_rate=0.3) and controls DCA, MAGIC, scImpute, SAVER, keeping the raw noisy data as baseline. Rationale: existing methods assume single noise types (e.g., DCA's zero-inflated negative binomial, scImpute's Gaussian-Gamma mixture), so covering five noise distributions tests whether a method is distribution agnostic. Evaluate by MSE between true and imputed values (Datasets 3-7), average silhouette width recovery of known group labels after take_norm/PCA(50)/t-SNE(2), and average recovered value at dropout zero positions versus average true value (Dataset 2).

## Related tools

- **Splatter** (Simulates the true count matrix (1000 genes, 500 cells, six groups) used as ground truth before noise injection)
- **AutoClass** (Primary imputation method under benchmark; run as AutoClassImpute with dropout_rate=0.3) — https://github.com/datapplab/AutoClass
- **DCA** (Control denoising method assuming zero-inflated negative binomial distribution) — https://github.com/theislab/dca
- **MAGIC** (Control imputation method) — https://github.com/KrishnaswamyLab/MAGIC
- **scImpute** (Control imputation method using a Gaussian-Gamma mixture model) — https://github.com/Vivianstats/scImpute
- **SAVER** (Control imputation method assuming negative binomial distribution) — https://github.com/mohuangx/SAVER
- **Python 3 / TensorFlow** (Runtime for AutoClassImpute and downstream normalization/PCA/t-SNE evaluation code) — https://github.com/datapplab/AutoClass

## Examples

```
res = AutoClassImpute(X, dropout_rate=0.3); X_norm = take_norm(res); PCA_raw = PCA(n_components=50).fit_transform(X_norm); tsne = TSNE(n_components=2).fit_transform(PCA_raw)
```

## Evaluation signals

- AutoClass attains the smallest mean squared error between true and imputed data among the five tested methods on Datasets 3-7 (Fig. 2 patterns)
- ASW recovery values for imputed data fall in the 0.2-0.5 range on the six-group simulated datasets
- On Dataset 2, average recovered value at dropout zero positions approximates the average true value at those positions
- ASW recovery is computed on known group labels after take_norm, PCA (n_components=50), and t-SNE (n_components=2)
- All noisy variants must derive from the same true matrix so MSE comparisons across noise types share one ground truth

## Limitations

- Supplementary Tables 1-3 containing simulation parameters are referenced but not included in the provided text, so exact noise injection parameters must be sourced from the SI
- The paper's discussion reports no explicit limitations; generalization beyond the tested noise types and distributions is unverified in the text

## Evidence

- [methods] Simulate with Splatter then inject five noise types: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [abstract] Distribution-agnostic motivation: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [results] Imputation call with dropout_rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] Dimensionality reduction for evaluation: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [intro] Control method distribution assumptions: "deep count autoencoder (DCA)9 assumes negative binomial distribution with or without zero inflation"
