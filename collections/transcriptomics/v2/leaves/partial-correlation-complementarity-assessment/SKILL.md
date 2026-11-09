---
name: partial-correlation-complementarity-assessment
description: 'Use when implementing or validating a greedy feature-selection criterion
  (e.g., RRCT or an mRMR extension) that must account for feature interactions: candidate
  features need scoring via conditional relevance given the already-selected subset,
  not just marginal relevance and pairwise redundancy.'
when_to_use_negative:
- Features are already selected and only a classifier evaluation (e.g., misclassification
  rate via random forest over 1...min(M,30) subsets) is needed — this skill only scores
  features
- The selection criterion intentionally ignores interactions (e.g., plain mRMR_Spearman,
  which combines only relevance and redundancy without a complementarity term)
- The data matrix is not a standard X/y setup (no response vector), since relevance
  and conditional relevance are both defined against y
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: uts
  role: Reference implementation of the RRCT uts function used to validate the reconstructed
    relevance-redundancy-complementarity ranking on an identical dataset
  repo: github.com/ThanasisTsanas/RRCT
- name: MATLAB 2021b
  role: Environment in which Spearman rank correlations, partial correlations, and
    the IT transformation are computed (all experiments run in MATLAB 2021b)
- name: mRMR_Spearman
  role: Simpler relevance-redundancy-only baseline against which the added complementarity
    (partial correlation) term is contrasted
  repo: https://github.com/ThanasisTsanas/mRMR_Spearman
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:89eace195cacde630c619fd8600dc2b68bc25c224a62b7b71e1d78b195a8cf9d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# partial-correlation-complementarity-assessment

## Summary

Quantifies the complementarity (conditional relevance) term of the RRCT feature-selection criterion by computing rank partial correlations between each candidate feature and the response, controlling for the already-selected feature subset rp(fj,y|S), alongside the information-theoretic transformation of Spearman rank correlations used for relevance and redundancy.

## When to use

Use when implementing or validating a greedy feature-selection criterion (e.g., RRCT or an mRMR extension) that must account for feature interactions: candidate features need scoring via conditional relevance given the already-selected subset, not just marginal relevance and pairwise redundancy.

## When NOT to use

- Features are already selected and only a classifier evaluation (e.g., misclassification rate via random forest over 1...min(M,30) subsets) is needed — this skill only scores features
- The selection criterion intentionally ignores interactions (e.g., plain mRMR_Spearman, which combines only relevance and redundancy without a complementarity term)
- The data matrix is not a standard X/y setup (no response vector), since relevance and conditional relevance are both defined against y

## Inputs

- Data matrix X (e.g., SPECTF [80 x 44; 187 x 44] or Spambase [4,601 x 57]) as numeric features in columns
- Response vector y (class labels for classification or continuous values for regression)
- Optional integer K, the number of features to select (default K = M, or K = 30 when M > 30)
- Already-selected feature subset S from prior greedy iterations

## Outputs

- Rank partial correlation values rp(fj,y|S) per candidate feature quantifying conditional relevance
- IT-transformed relevance (feature-response) and redundancy (feature-pair) terms from Spearman rank correlations
- Ranked feature list of size K, in descending order of importance by column index (e.g., selected_features.txt)

## How to apply

Load the dataset into MATLAB 2021b as a data matrix X and response vector y. Compute Spearman rank correlation coefficients between each feature and the response and between all feature pairs, then apply the nonlinear information-theoretic transformation to these coefficients to derive the feature-response relevance term and the pairwise redundancy term. Next, compute partial correlation coefficients between each candidate feature and the response controlling for the already-selected feature subset S, i.e. rp(fj,y|S); this IT-transformed quantity is added or subtracted in the RRCT criterion depending on sign terms, capturing complementarity (conditional relevance) that mRMR ignores. Greedily combine the relevance, redundancy, and complementarity terms at each step to select K features (default K = M, or K = 30 when the dataset has more than 30 features, as in Relathe [1,427 x 4,322] or Ovarian cancer [72 x 592]). Emit the selected features in descending order of importance, ranked by column index.

## Related tools

- **uts** (Reference implementation of the RRCT uts function used to validate the reconstructed relevance-redundancy-complementarity ranking on an identical dataset) — github.com/ThanasisTsanas/RRCT
- **MATLAB 2021b** (Environment in which Spearman rank correlations, partial correlations, and the IT transformation are computed (all experiments run in MATLAB 2021b))
- **mRMR_Spearman** (Simpler relevance-redundancy-only baseline against which the added complementarity (partial correlation) term is contrasted) — https://github.com/ThanasisTsanas/mRMR_Spearman

## Examples

```
features = uts(X, y, K)  % MATLAB 2021b; K = M or K = 30 when M > 30
```

## Evaluation signals

- Run the reference uts implementation on the identical dataset and confirm the reconstructed greedy ranking matches its emitted feature list (descending importance, ranked by column index)
- The complementarity term equals the rank partial correlation rp(fj,y|S) after IT transformation, changing as S grows each greedy step
- Number of selected features equals K (K = M, or K = 30 when M > 30, e.g., selecting 30 of 4,322 Relathe features)
- Sign terms: the IT-transformed partial correlation is added or subtracted in the RRCT criterion consistent with its sign
- Feature selection completes within seconds even on fat datasets (thousands of features), consistent with the paper's computational-efficiency claim

## Limitations

- The criterion is heuristic; complementarity is captured only via partial correlations, not a full conditional-dependence model
- Default K = M means all features are ranked on small datasets; the K = 30 cap only kicks in for datasets with more than 30 features
- FEAST-based mutual-information implementations require discretisation (10 equal-width bins suggested), whereas RRCT's rank-correlation route avoids discretisation but measures association through Spearman coefficients only

## Evidence

- [abstract] The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients: "The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients, and the feature"
- [readme] RRCT introduces a third complementarity term beyond mRMR: "RRCT excplicitly takes into account relevance and redundancy (like mRMR), and also introduces an additional third term to account for conditional relevance (also known as complementarity)."
- [methods] uts optional K input and greedy selection output: "K        -> number of features to be selected (integer value)"
- [methods] Greedy selection emits ranked features by column index: [evidence span withheld — anchor status: not_found]
- [results] Experiments executed in MATLAB 2021b: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
