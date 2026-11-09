---
name: spearman-rank-correlation-computation
description: Use when a continuous or ordinal data matrix and response are given and
  you need fast, robust, nonparametric quantification of feature-response and feature-pair
  associations to feed a correlation-based filter (RRCT, mRMR_Spearman).
when_to_use_negative:
- The analysis requires capturing non-monotonic or purely categorical associations
  that rank correlations cannot quantify (crude histogram-based mutual information
  is 'likely problematic', so a discrete-MI toolbox such as FEAST is a different choice,
  but not a fix for continuous inputs)
- The goal is to account for feature interactions/complementarity at selection time
  using Spearman correlations alone — mRMR_Spearman ignores the conditional-relevance
  term; use the full RRCT criterion instead
- Features are expected to be independent and redundant filtering provides no benefit
  (correlation-based redundancy penalties would discard useful features)
tools:
- name: MATLAB 2021b
  role: Implementation environment in which the Spearman correlations, the information-theoretic
    transformation, and the greedy mRMR/RRCT selection are computed
- name: mRMR_Spearman
  role: Provides the Spearman-correlation-based mRMR criterion (mRMR_Spearman(X, y))
    that replaces the demanding mutual-information estimation of Peng et al. with
    fast rank correlations
  repo: ThanasisTsanas/mRMR_Spearman
- name: RRCT
  role: Reference implementation (MATLAB and Python) that consumes the Spearman-derived
    relevance/redundancy terms plus partial-correlation complementarity; the uts function
    implements it with an optional K input
  repo: github.com/ThanasisTsanas/RRCT
- name: uts
  role: Reference RRCT implementation used to validate that the reconstructed feature
    ranking matches its emitted feature list for the identical dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:fa9b50a27b58b4f676a1fa101afa0e5b04f56ccbc70f6f157a0dee026e595852
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ThanasisTsanas/mRMR_Spearman
  tools:
  - MATLAB
  - MATLAB 2021b
  - RRCT
  - mRMR_Spearman
  - uts
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spearman-rank-correlation-computation

## Summary

Computes Spearman rank correlation coefficients between each feature and the response and between all feature pairs, then optionally applies the nonlinear information-theoretic transformation used by RRCT/mRMR_Spearman to derive relevance and redundancy terms. This is the fast, robust correlation backbone of the RRCT feature-selection framework (Tsanas, Patterns 2022).

## When to use

Use when a continuous or ordinal data matrix and response are given and you need fast, robust, nonparametric quantification of feature-response and feature-pair associations to feed a correlation-based filter (RRCT, mRMR_Spearman).

## When NOT to use

- The analysis requires capturing non-monotonic or purely categorical associations that rank correlations cannot quantify (crude histogram-based mutual information is 'likely problematic', so a discrete-MI toolbox such as FEAST is a different choice, but not a fix for continuous inputs)
- The goal is to account for feature interactions/complementarity at selection time using Spearman correlations alone — mRMR_Spearman ignores the conditional-relevance term; use the full RRCT criterion instead
- Features are expected to be independent and redundant filtering provides no benefit (correlation-based redundancy penalties would discard useful features)

## Inputs

- Data matrix X (n samples × m features), e.g., SPECTF [187 × 44], Spambase [4,601 × 57], or Relathe [1,427 × 4,322]
- Response vector y (continuous for regression or class labels for classification)
- Optional integer K = number of features to select (default K = M, or K = 30 when the dataset has more than 30 features)

## Outputs

- Feature-response Spearman rank correlation vector
- Pairwise feature-feature Spearman correlation matrix
- Information-theoretic transformed relevance and redundancy terms
- selected_features.txt: K selected features in descending order of importance (ranked by column index)

## How to apply

Load the dataset (e.g., SPECTF as a 187 × 44 data matrix plus response vector) into MATLAB 2021b and compute Spearman rank correlation coefficients between each feature and the response and between all feature pairs. Apply the paper's nonlinear information-theoretic transformation to these rank correlation coefficients to derive the feature-response relevance term and the pairwise redundancy term, since RRCT explicitly accounts for feature interactions via this transformation. In the mRMR_Spearman simplification, Spearman correlations replace the computationally demanding mutual-information estimation used by Peng et al. (2005), yielding a fast greedy selection that emits K features in descending order of importance (ranked by column index), with K defaulting to M or 30 when the dataset has more than 30 features. Complementarity is not captured by raw Spearman correlations; it requires rank partial correlations rp(fj,y|S) against the already-selected subset, which is the separate RRCT criterion. Validate by running the reference uts implementation on the identical dataset and confirming the emitted feature list matches.

## Related tools

- **MATLAB 2021b** (Implementation environment in which the Spearman correlations, the information-theoretic transformation, and the greedy mRMR/RRCT selection are computed)
- **mRMR_Spearman** (Provides the Spearman-correlation-based mRMR criterion (mRMR_Spearman(X, y)) that replaces the demanding mutual-information estimation of Peng et al. with fast rank correlations) — ThanasisTsanas/mRMR_Spearman
- **RRCT** (Reference implementation (MATLAB and Python) that consumes the Spearman-derived relevance/redundancy terms plus partial-correlation complementarity; the uts function implements it with an optional K input) — github.com/ThanasisTsanas/RRCT
- **uts** (Reference RRCT implementation used to validate that the reconstructed feature ranking matches its emitted feature list for the identical dataset)

## Examples

```
features = mRMR_Spearman(X, y)
```

## Evaluation signals

- All Spearman coefficients lie in [-1, 1] and the pairwise correlation matrix is symmetric with unit diagonal
- The emitted selected_features.txt contains exactly K features, where K defaults to M or 30 when the dataset has more than 30 features, listed in descending order of importance ranked by column index
- The reconstructed ranking matches the feature list emitted by the reference uts implementation on the identical dataset
- Feature-response relevance is monotonically consistent with |Spearman| values: the first selected feature is the one with the highest transformed relevance
- Runtime stays within the paper's stated efficiency envelope (RRCT runs 'within a few seconds including on massive datasets with thousands of features')

## Limitations

- Spearman correlations capture only monotonic associations; they are a simplification of the original mRMR mutual-information criterion of Peng et al. (2005)
- mRMR_Spearman accounts for relevance and redundancy but not complementarity; the conditional-relevance term requires rank partial correlations rp(fj,y|S) as in full RRCT
- Benchmarked selection setting is K = min(M, 30); behavior for other K values or larger feature budgets is not specified in the source cards

## Evidence

- [readme] here I opt for the Spearman correlation coefficient criterion which allows for a fast and computationally inexpensive feature selection algorithm: [evidence span withheld — anchor status: external_doc]
- [intro] RRCT uses a simple nonlinear transformation of the correlation coefficients using information theoretic concepts to quantify the association of the features with the response and the overlapping: "RRCT uses a simple nonlinear transformation of the correlation coefficients using information theoretic concepts to quantify the association of the features with the response and the overlapping"
- [methods] The selected features in descending order of importance (ranked by column index): "features -> The selected features in descending order of importance (ranked by column index)"
- [results] All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b.: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
