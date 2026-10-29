---
name: filter-based-feature-selection-rank-correlation
description: Use when a data matrix plus response vector (e.g., fat datasets like
  Relathe 1,427×4,322 or SRBCT 63×2,308) needs ranked feature selection and out-of-sample
  classification performance assessment.
when_to_use_negative:
- The task requires wrapper or embedded FS with model interaction (RRCT is a filter
  algorithm that scores features before any classifier is trained)
- The data requires mutual-information estimation via proper density estimation on
  continuous variables — the paper's mRMR_Spearman variant explicitly avoids computationally
  demanding mutual information computation in favor of Spearman correlation
- 'The goal is unsupervised feature selection: the uts function requires a response
  vector y for relevance quantification'
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: uts (RRCT)
  role: Core RRCT feature-selection function invoked on each dataset to output the
    ranked feature list
  repo: https://github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Comparison filter algorithm using Spearman correlation instead of mutual information
    for mRMR-style selection
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
- name: FEAST
  role: MATLAB/C toolbox providing competing mutual-information filter algorithms
    (mrmr, jmi, cmim, relief) used for benchmarking; continuous inputs must be discretised
    first
  repo: https://github.com/Craigacp/FEAST
- name: MATLAB 2021b
  role: Environment in which all FS and random-forest experiments were run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:c02a144130720928435d5448fcd977d4b7f84b713feea5320aec205e78e2972e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/RRCT
  tools:
  - FEAST
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  - uts (RRCT)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# filter-based-feature-selection-rank-correlation

## Summary

A rank-correlation-based filter feature-selection workflow centered on the RRCT algorithm, which trades off relevance, redundancy, and complementarity via information-theoretic transformations of Spearman rank correlations and partial correlations. It is benchmarked by feeding selected feature subsets (1…min(M,30)) into a random forest and recording out-of-sample misclassification percentage.

## When to use

Use when a data matrix plus response vector (e.g., fat datasets like Relathe 1,427×4,322 or SRBCT 63×2,308) needs ranked feature selection and out-of-sample classification performance assessment.

## When NOT to use

- The task requires wrapper or embedded FS with model interaction (RRCT is a filter algorithm that scores features before any classifier is trained)
- The data requires mutual-information estimation via proper density estimation on continuous variables — the paper's mRMR_Spearman variant explicitly avoids computationally demanding mutual information computation in favor of Spearman correlation
- The goal is unsupervised feature selection: the uts function requires a response vector y for relevance quantification

## Inputs

- Data matrix X (n samples × M features) and response vector y, e.g., from UCI Machine Learning Repository or ASU FS Repository datasets such as Spambase [4,601 × 57], Relathe [1,427 × 4,322], Ovarian cancer [72 × 592], SRBCT [63 × 2,308]
- Optional integer K, the number of features to be selected (default K = M, or K = 30 when M > 30)

## Outputs

- Selected feature indices in descending order of importance (ranked by column index)
- Out-of-sample misclassification percentage for feature subsets of size 1…min(M,30) fed into a random forest, e.g., assembled into rrct_misclassification_rates.csv

## How to apply

Load the data matrix X and response y, then invoke the RRCT `uts` function, optionally passing K (default K = M, or K = 30 when the dataset has more than 30 features); `uts` quantifies feature-response relevance and feature-pair redundancy via an information-theoretic transformation of rank correlations and complementarity via partial correlations. Collect the output feature list ranked in descending order of importance by column index. Feed feature subsets of size 1…min(M,30) into a random forest (RF) classifier and record the out-of-sample misclassification percentage for each subset size. Optionally compare RRCT against competing filter algorithms (e.g., mRMR Peng, mRMR_Spearman, FEAST's mrmr/jmi/relief implementations) using the same RF evaluation. All experiments in the paper were run in MATLAB 2021b; RRCT was benchmarked against 19 FS algorithms on four synthetic and eight real-world datasets (Cardiotocography, Handwriting, Lymph, Ovarian cancer, Relathe, Spambase, SPECTF, SRBCT) from the UCI Machine Learning Repository or ASU FS Repository.

## Related tools

- **uts (RRCT)** (Core RRCT feature-selection function invoked on each dataset to output the ranked feature list) — https://github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Comparison filter algorithm using Spearman correlation instead of mutual information for mRMR-style selection) — https://github.com/ThanasisTsanas/mRMR_Spearman
- **FEAST** (MATLAB/C toolbox providing competing mutual-information filter algorithms (mrmr, jmi, cmim, relief) used for benchmarking; continuous inputs must be discretised first) — https://github.com/Craigacp/FEAST
- **MATLAB 2021b** (Environment in which all FS and random-forest experiments were run)

## Examples

```
features = uts(X, y, K) % MATLAB: RRCT uts function; select K features ranked by importance
```

## Evaluation signals

- The uts output is a feature list in descending order of importance, ranked by column index, of length K (default M, capped at 30 for fat datasets)
- Misclassification percentage is recorded for every subset size 1…min(M,30) fed into the random forest, yielding a full performance curve per dataset
- On synthetic datasets, feature-selection quality is verified by false discovery rate (FDR) computed at the number of true features (RRCT achieved FDR ≈ 0.10 while GSO and IAMB achieved 0)
- Results for the eight benchmark datasets should be assembled into rrct_misclassification_rates.csv
- RRCT's expected competitive behavior: clear winner on Relathe and ovarian-cancer, tied with a few algorithms on SRBCT, generally strong out-of-sample performance

## Limitations

- RRCT is a heuristic algorithm; the author tentatively makes suggestions on generalizability and application of the best-performing FS algorithms rather than guaranteeing optimality
- Popular alternatives such as RELIEF, mRMR Peng, CFS, and JMI may erroneously select probes in synthetic datasets, so benchmark results are algorithm- and dataset-dependent
- FEAST requires discrete inputs — continuous values must be discretised (10 equal-width bins suggested) or results are unreliable and slow; C library expects column-major matrices

## Evidence

- [abstract] RRCT integrates relevance, redundancy, complementarity: "We present a new heuristic feature-selection (FS) algorithm that integrates in a principled algorithmic framework the three key FS components: relevance, redundancy, and complementarity."
- [abstract] Association strength via information theoretic transformation of rank correlations: "The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients"
- [methods] uts optional input K: "K        -> number of features to be selected (integer value)"
- [methods] Selected features output ordering: "features -> The selected features in descending order of importance (ranked by column index)"
- [results] RF evaluation of subsets: "as a function of the number of selected features presented into the random forest (RF)"
- [readme] RRCT README description: "This function is a computationally efficient, robust approach for feature selection using the RRCT algorithm."
