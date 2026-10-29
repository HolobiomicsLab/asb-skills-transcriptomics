---
name: greedy-feature-ranking-scoring
description: Use when a data matrix and response vector must be reduced to a small
  ranked feature subset, and plain mRMR is insufficient because feature interactions
  (complementarity) should be explicitly scored.
when_to_use_negative:
- Inputs are continuous and a mutual-information toolbox like FEAST is required —
  FEAST expects discrete inputs and produces unreliable results on continuous values
  without discretisation
- The task requires exhaustive or non-greedy subset search rather than a fast heuristic
  off-the-shelf ranking (RRCT is a greedy heuristic extension of mRMR)
- The problem is not representable as a data matrix plus response vector, the scope
  the article sets for a generic FS algorithm
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3577
tools:
- name: MATLAB 2021b
  role: Implementation environment in which the RRCT scoring, Spearman/partial correlations,
    and greedy selection are executed (all experiments run on Windows 10, Intel i9-9900K,
    64 GB RAM)
- name: uts
  role: Reference RRCT implementation accepting K (number of features to select);
    used to validate the reconstructed ranking against its emitted feature list
  repo: github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Related simpler baseline by the same author using the Spearman-correlation
    mRMR criterion without the complementarity term
  repo: github.com/ThanasisTsanas/RRCT
- name: FEAST
  role: Comparison toolbox of mutual-information filter FS algorithms (mim, mrmr,
    jmi, cmim, etc.) benchmarked against RRCT
  repo: https://github.com/Craigacp/FEAST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:9b421a52ffbb31177b750ebe6c5cfd3686cacf5d1394502f346ce9129fb4989f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# greedy-feature-ranking-scoring

## Summary

Implements the RRCT greedy feature-selection scoring criterion, which combines Spearman rank-correlation-based relevance and redundancy terms with a partial-correlation-based complementarity (conditional relevance) term to rank candidate features at each selection step.

## When to use

Use when a data matrix and response vector must be reduced to a small ranked feature subset, and plain mRMR is insufficient because feature interactions (complementarity) should be explicitly scored.

## When NOT to use

- Inputs are continuous and a mutual-information toolbox like FEAST is required — FEAST expects discrete inputs and produces unreliable results on continuous values without discretisation
- The task requires exhaustive or non-greedy subset search rather than a fast heuristic off-the-shelf ranking (RRCT is a greedy heuristic extension of mRMR)
- The problem is not representable as a data matrix plus response vector, the scope the article sets for a generic FS algorithm

## Inputs

- Data matrix X (e.g., MATLAB matrix; benchmark examples: SPECTF [80 x 44; 187 x 44], Spambase [4,601 x 57], Relathe [1,427 x 4,322])
- Response vector y (regression or classification labels; RRCT generalizes to both without adjusting for mixed-type variables)
- K, the number of features to be selected (integer; optional input to uts)

## Outputs

- selected_features.txt listing the selected features in descending order of importance (ranked by column index)

## How to apply

Load the benchmark data (e.g., SPECTF from the UCI/ASU repositories) into MATLAB 2021b as a data matrix and response vector. Compute Spearman rank correlations between each feature and the response and between all feature pairs, then apply the nonlinear information-theoretic transformation to obtain the feature-response relevance term and the pairwise redundancy term. Quantify complementarity via rank partial correlation of the candidate feature with the response controlling for the already-selected subset (rp(fj,y|S)), which is IT-transformed and added or subtracted based on sign terms in the RRCT criterion. Greedily combine the three terms to select K features (default K = M, or K = 30 when the dataset has more than 30 features), and emit features in descending order of importance ranked by column index to selected_features.txt.

## Related tools

- **MATLAB 2021b** (Implementation environment in which the RRCT scoring, Spearman/partial correlations, and greedy selection are executed (all experiments run on Windows 10, Intel i9-9900K, 64 GB RAM))
- **uts** (Reference RRCT implementation accepting K (number of features to select); used to validate the reconstructed ranking against its emitted feature list) — github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Related simpler baseline by the same author using the Spearman-correlation mRMR criterion without the complementarity term) — github.com/ThanasisTsanas/RRCT
- **FEAST** (Comparison toolbox of mutual-information filter FS algorithms (mim, mrmr, jmi, cmim, etc.) benchmarked against RRCT) — https://github.com/Craigacp/FEAST

## Examples

```
features = uts(X, y, 'K', 30);  % RRCT greedy selection of top 30 features in MATLAB 2021b
```

## Evaluation signals

- Reconstructed ranking must exactly match the feature list emitted by the reference uts implementation on the identical dataset
- On synthetic datasets with probe features, false discovery rate (FDR) measured at the number of true features should approach 0, as RRCT achieved FDR of 0, 0, 0, and 0.10 across the four synthetic datasets
- Output file must list features in descending order of importance ranked by column index
- Selected subsets fed to a random forest classifier should yield competitive misclassification percentages as a function of 1…min(M,30) features

## Limitations

- RRCT is a heuristic algorithm; on synthetic data GSO and IAMB matched its probe-discrimination (FDR 0) and RRCT had FDR 0.10 on one synthetic dataset
- Popular alternatives it was compared to (RELIEF, mRMR Peng, CFS, JMI) may stumble and erroneously select probes — but RRCT's advantage over them is empirical, not guaranteed on all data
- Runtime and memory behaviour depends on dataset dimensionality; benchmarking spanned 'fat' datasets such as SRBCT [63 x 2,308] and Ovarian cancer [72 x 592]

## Evidence

- [methods] The complementarity term quantifies conditional relevance as the rank partial correlation between a candidate feature and the response controlling for the already-selected feature subset (rp(fj,y|S)): "The association strength between each feature and the response and between feature pairs is quantified via an information theoretic transformation of rank correlation coefficients, and the feature"
- [methods] greedy selection output format: "features -> The selected features in descending order of importance (ranked by column index)"
- [readme] README positioning of RRCT relative to mRMR: "RRCT excplicitly takes into account relevance and redundancy (like mRMR), and also introduces an additional third term to account for conditional relevance (also known as complementarity)"
- [results] empirical benchmarking result on synthetic probes: "The results in Figure 1 and Table 1 illustrate that GSO, IAMB, and RRCT are all very effective at discarding probes."
- [results] implementation environment: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
