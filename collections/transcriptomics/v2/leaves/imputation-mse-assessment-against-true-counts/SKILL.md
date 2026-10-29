---
name: imputation-mse-assessment-against-true-counts
description: Use when benchmarking scRNA-Seq imputation/denoising methods on simulated
  data where a ground-truth count matrix exists, and you need a quantitative recovery
  metric (MSE) plus cell-structure recovery (ASW) per noise type.
when_to_use_negative:
- Analyzing real scRNA-Seq data with no ground-truth count matrix — MSE against true
  values is undefined; use clustering or biological-validation metrics instead
- Data with noise types outside the five tested distributions (dropout, uniform, Gaussian,
  Gamma, Poisson, negative binomial) — recovery may not generalize, since existing
  methods largely focus on a single noise type
- Downstream tasks other than value recovery, e.g. differential expression analysis
  or batch effect removal, which the paper evaluates with different workflows and
  metrics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoClass
  role: Primary imputation method under evaluation; run as AutoClassImpute(X, dropout_rate=0.3)
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control imputation method (deep count autoencoder, negative binomial with/without
    zero inflation)
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Control imputation method
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: scImpute
  role: Control imputation method (Gaussian and Gamma mixture model)
  repo: https://github.com/Vivianstats/scImpute
- name: SAVER
  role: Control imputation method (negative binomial assumption)
  repo: https://github.com/mohuangx/SAVER
- name: Splatter
  role: Simulates the true count matrix and group structure used as ground truth
- name: Python 3
  role: Implementation environment for the evaluation workflow
- name: TensorFlow >=2.0
  role: Dependency for running AutoClass
- name: numpy
  role: Numerical computation for MSE and noise application
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:c6320b979e4b237574a4c968332f43b3daa79cc895b4f4e4307163ed4caa55ff
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# imputation-mse-assessment-against-true-counts

## Summary

Benchmark denoising/imputation of simulated scRNA-Seq data by computing mean squared error (MSE) between true and imputed expression values, complemented by average silhouette width (ASW) recovery of known group labels and dropout-zero recovery. This reproduces the AutoClass Fig. 2 evaluation where AutoClass, DCA, MAGIC, scImpute, and SAVER are compared across five noise types.

## When to use

Use when benchmarking scRNA-Seq imputation/denoising methods on simulated data where a ground-truth count matrix exists, and you need a quantitative recovery metric (MSE) plus cell-structure recovery (ASW) per noise type.

## When NOT to use

- Analyzing real scRNA-Seq data with no ground-truth count matrix — MSE against true values is undefined; use clustering or biological-validation metrics instead
- Data with noise types outside the five tested distributions (dropout, uniform, Gaussian, Gamma, Poisson, negative binomial) — recovery may not generalize, since existing methods largely focus on a single noise type
- Downstream tasks other than value recovery, e.g. differential expression analysis or batch effect removal, which the paper evaluates with different workflows and metrics

## Inputs

- True (ground-truth) simulated count matrix from Splatter, e.g. truecounts.csv / Dataset1_true.csv (1000 genes x 500 cells, six groups)
- Noisy count matrices per noise type (dropout, uniform, Gaussian, Gamma, Poisson, negative binomial), e.g. Dataset1_raw.csv
- Known Splatter group labels per cell (for ASW recovery)
- Imputed matrices from each method: AutoClass, DCA, MAGIC, scImpute, SAVER, plus the raw noisy matrix as baseline

## Outputs

- Per-method MSE between true and imputed expression values for Datasets 3-7
- Per-method ASW recovery of known group labels (expected 0.2-0.5 range, matching Fig. 2d)
- Average recovered value at dropout zero positions vs. average true value at those positions (Dataset 2)
- Combined results table saved as denoising_benchmark_results.csv

## How to apply

Simulate a true count matrix with Splatter (1000 genes, 500 cells, six groups per Supplementary Table 2), then derive Dataset 2 by applying dropout noise and Datasets 3-7 by applying uniform, Gaussian, Gamma, Poisson, and negative binomial noise to the same true data. Impute each noisy dataset with AutoClassImpute (dropout_rate=0.3) and the control methods DCA, MAGIC, scImpute, and SAVER, keeping the raw noisy data as a baseline. For each imputed dataset, normalize with take_norm, reduce with PCA (n_components=50) followed by t-SNE (n_components=2), and compute average silhouette width (ASW) recovery against the known Splatter group labels. For Datasets 3-7, compute the MSE between true and imputed expression values per method; for Dataset 2, additionally compute the average recovered value at dropout zero positions versus the average true value at those positions. The rationale is that AutoClass is distribution agnostic, so MSE across multiple noise distributions tests whether a method's distribution assumptions (e.g., DCA's zero-inflated negative binomial, scImpute's Gaussian-Gamma mixture, SAVER's negative binomial) limit recovery.

## Related tools

- **AutoClass** (Primary imputation method under evaluation; run as AutoClassImpute(X, dropout_rate=0.3)) — https://github.com/datapplab/AutoClass
- **DCA** (Control imputation method (deep count autoencoder, negative binomial with/without zero inflation)) — https://github.com/theislab/dca
- **MAGIC** (Control imputation method) — https://github.com/KrishnaswamyLab/MAGIC
- **scImpute** (Control imputation method (Gaussian and Gamma mixture model)) — https://github.com/Vivianstats/scImpute
- **SAVER** (Control imputation method (negative binomial assumption)) — https://github.com/mohuangx/SAVER
- **Splatter** (Simulates the true count matrix and group structure used as ground truth)
- **Python 3** (Implementation environment for the evaluation workflow)
- **TensorFlow >=2.0** (Dependency for running AutoClass)
- **numpy** (Numerical computation for MSE and noise application)

## Examples

```
from AutoClass import AutoClassImpute; res = AutoClassImpute(X, dropout_rate=0.3); mse = ((X_true - res)**2).mean()
```

## Evaluation signals

- Reproduced table matches Fig. 2d-f patterns: ASW recovery values fall in the 0.2-0.5 range across methods and datasets
- AutoClass attains the smallest MSE between true and imputed expression among the compared methods (AutoClass, DCA, MAGIC, scImpute, SAVER) on Datasets 3-7
- On Dataset 2 (dropout), average recovered value at dropout zero positions approaches the average true value at those positions for the best method
- ASW recovery is computed on t-SNE embeddings derived from PCA (n_components=50) of take_norm-normalized imputed data with fixed random_state=1 KMeans for consistency
- All ASW, MSE, and zero-recovery values assemble into a single denoising_benchmark_results.csv without missing method-by-dataset cells

## Limitations

- Validation thresholds are qualitative (ASW 0.2-0.5, AutoClass smallest MSE) rather than statistical tests; no significance values are given
- Supplementary Table 2 (simulation parameters) is referenced but not included in the provided text, so exact Splatter simulation settings must be obtained separately
- Ground truth exists only for simulated data; the metric cannot validate recovery on real datasets like the Baron dataset
- AutoClassImpute is run with a fixed dropout_rate=0.3; sensitivity of MSE to this hyperparameter is not assessed by this workflow (though the paper reports robustness to key hyperparameters)

## Evidence

- [results] AutoClass consistently achieved the smallest mean squared error between true and imputed data among the five tested methods, recovering cell-type pattern that control methods (DCA, MAGIC, scImpute, SAVER) failed to recover.: [evidence span withheld — anchor status: not_found]
- [intro] Simulation design: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] Imputation call: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] Dimensionality reduction for ASW: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [abstract] Distribution-agnostic rationale: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [results] Baseline is kept: "X = expr.values #Splash generated scRNA-seq data with dropout"
