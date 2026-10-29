---
name: information-theoretic-correlation-transformation
description: Use when you must quantify association strength between continuous/ordinal
  features and a response, and overlapping information between feature pairs, from
  Spearman rank correlations without expensive mutual-information density estimation.
when_to_use_negative:
- When features are inherently discrete and a mutual-information toolbox is appropriate
  — FEAST expects discrete inputs and continuous values must be discretised (e.g.,
  10 equal-width bins) before use
- When you need a full probabilistic mutual-information estimate rather than a correlation-based
  surrogate — the transformation is a heuristic nonlinear mapping of rank correlations,
  not proper MI density estimation
- When the analysis goal is not ranking/selecting features from a data matrix plus
  response (RRCT is designed specifically as an off-the-shelf filter feature-selection
  step)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: MATLAB 2021b
  role: Implementation environment in which the Spearman correlations, IT transformation,
    partial correlations, and greedy RRCT scoring are computed
  repo: github.com/ThanasisTsanas/RRCT
- name: uts
  role: Reference RRCT implementation used to validate that the reconstructed feature
    ranking matches its emitted feature list
  repo: github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Simplified Spearman-correlation-based mRMR precursor illustrating the relevance/redundancy-only
    criterion that RRCT extends with complementarity
  repo: ThanasisTsanas/mRMR_Spearman
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:b8210e2c857d1c73b1ecf401446eeedd845a6acb2b0402f531ad62d9050b6aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# information-theoretic-correlation-transformation

## Summary

A nonlinear, information-theoretic transformation of Spearman rank correlation coefficients that quantifies feature-response relevance and feature-pair redundancy, and, together with partial correlations, the complementarity term used in the RRCT feature-selection criterion.

## When to use

Use when you must quantify association strength between continuous/ordinal features and a response, and overlapping information between feature pairs, from Spearman rank correlations without expensive mutual-information density estimation.

## When NOT to use

- When features are inherently discrete and a mutual-information toolbox is appropriate — FEAST expects discrete inputs and continuous values must be discretised (e.g., 10 equal-width bins) before use
- When you need a full probabilistic mutual-information estimate rather than a correlation-based surrogate — the transformation is a heuristic nonlinear mapping of rank correlations, not proper MI density estimation
- When the analysis goal is not ranking/selecting features from a data matrix plus response (RRCT is designed specifically as an off-the-shelf filter feature-selection step)

## Inputs

- Data matrix X (samples x features), e.g., SPECTF [80 x 44; 187 x 44] from the UCI/ASU repositories
- Response vector y (regression or classification target)
- Optional integer K: number of features to be selected (default K = M, or K = 30 when M > 30)

## Outputs

- IT-transformed relevance and redundancy terms derived from Spearman rank correlation coefficients
- Ranked feature list in descending order of importance (ranked by column index), written to selected_features.txt

## How to apply

In MATLAB 2021b, load the data matrix and response vector (e.g., SPECTF from the UCI/ASU repositories) and compute Spearman rank correlation coefficients between each feature and the response and between all feature pairs. Apply the nonlinear information-theoretic transformation to these coefficients to derive the feature-response relevance term and the pairwise redundancy term, avoiding the demanding density estimation (or crude histograms) used by classic mRMR implementations. Compute rank partial correlations rp(fj,y|S) between the candidate feature and the response controlling for the already-selected subset S, IT-transform them to form the complementarity (conditional relevance) term, and add or subtract it based on sign terms in the RRCT criterion. Greedily combine relevance, redundancy, and complementarity to select K features (default K = M, or K = 30 when the dataset has more than 30 features), emitting the selected features in descending order of importance ranked by column index to selected_features.txt. Validate by running the reference uts implementation on the identical dataset and confirming the reconstructed ranking matches its emitted feature list.

## Related tools

- **MATLAB 2021b** (Implementation environment in which the Spearman correlations, IT transformation, partial correlations, and greedy RRCT scoring are computed) — github.com/ThanasisTsanas/RRCT
- **uts** (Reference RRCT implementation used to validate that the reconstructed feature ranking matches its emitted feature list) — github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Simplified Spearman-correlation-based mRMR precursor illustrating the relevance/redundancy-only criterion that RRCT extends with complementarity) — ThanasisTsanas/mRMR_Spearman

## Examples

```
features = mRMR_Spearman(X, y);
```

## Evaluation signals

- Run the reference uts implementation on the identical dataset and confirm the reconstructed ranking matches its emitted feature list
- Recomputed Spearman feature-response and feature-pair correlations reproduce the values feeding the relevance and redundancy terms
- Output file selected_features.txt lists exactly K features (default K = M, or K = 30 when the dataset has more than 30 features) in descending order of importance ranked by column index
- Complementarity terms are computed as rank partial correlations rp(fj,y|S) controlling for the already-selected subset, with sign-dependent add/subtract in the RRCT criterion
- Greedy selection runs efficiently — the RRCT algorithm can run within a few seconds including on datasets with thousands of features

## Limitations

- It is a heuristic criterion: RRCT is described as 'a new heuristic feature-selection algorithm', so transformed correlation quantities are surrogates for information-theoretic quantities, not exact MI
- Crude histogram-based MI estimation (as in many open-source mRMR packages) is 'likely problematic'; this skill's correlation-based route trades exactness for computational efficiency
- Continuous inputs are not natively supported by MI-based alternatives like FEAST, which produces unreliable results on continuous values unless discretised (e.g., 10 equal-width bins)

## Evidence

- [abstract] The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients: "The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients"
- [intro] RRCT uses a simple nonlinear transformation of the correlation coefficients using information theoretic concepts: "RRCT uses a simple nonlinear transformation of the correlation coefficients using information theoretic concepts to quantify the association of the features with the response and the overlapping"
- [methods] Selected features output ordering: "features -> The selected features in descending order of importance (ranked by column index)"
- [methods] uts K parameter: "K        -> number of features to be selected (integer value)"
- [results] Experimental environment: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [readme] README on RRCT extending mRMR with complementarity: "RRCT excplicitly takes into account relevance and redundancy (like mRMR), and also introduces an additional third term to account for conditional relevance (also known as complementarity)."
