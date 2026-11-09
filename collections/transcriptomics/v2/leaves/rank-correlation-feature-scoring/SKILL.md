---
name: rank-correlation-feature-scoring
description: Use when a data matrix and response are given and features must be scored
  for relevance and redundancy with a fast, robust criterion — especially for fat
  datasets (e.g. 100x500, 1000x500) or mixed-type variables.
when_to_use_negative:
- Features are already pre-selected or the goal is wrapper/embedding selection rather
  than filter-based scoring of the full feature set
- The analysis requires wrapper-based or causal-discovery methods (e.g. the IAMB-style
  algorithms from CausalExplorer) rather than a fast filter criterion
- Rank correlation is inappropriate for the data structure, e.g. when feature-response
  association is purely conditional and not captured by marginal rank statistics
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: uts (RRCT)
  role: MATLAB function implementing the rank-correlation-based RRCT scoring; takes
    X, y and optional K, returns features ranked by importance
  repo: github.com/ThanasisTsanas/RRCT
- name: mRMR_Spearman
  role: Companion implementation using Spearman correlation instead of mutual information
    for the mRMR criterion
  repo: ThanasisTsanas/mRMR_Spearman
- name: MATLAB 2021b
  role: Environment in which all RRCT experiments and synthetic-benchmark FDR computations
    were run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:321be988962a5825bc88480a3bd435309928d503241d540e2aba37ce271d30bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - MATLAB 2021b
  - mRMR_Spearman
  - uts
  - uts (RRCT)
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rank-correlation-feature-scoring

## Summary

Quantify feature-response association and feature-pair redundancy using Spearman rank correlation coefficients transformed into information theoretic quantities, as done in the RRCT feature-selection algorithm. This provides a fast, robust alternative to mutual information estimation via histograms or density estimation.

## When to use

Use when a data matrix and response are given and features must be scored for relevance and redundancy with a fast, robust criterion — especially for fat datasets (e.g. 100x500, 1000x500) or mixed-type variables.

## When NOT to use

- Features are already pre-selected or the goal is wrapper/embedding selection rather than filter-based scoring of the full feature set
- The analysis requires wrapper-based or causal-discovery methods (e.g. the IAMB-style algorithms from CausalExplorer) rather than a fast filter criterion
- Rank correlation is inappropriate for the data structure, e.g. when feature-response association is purely conditional and not captured by marginal rank statistics

## Inputs

- Data matrix X (e.g. MATLAB matrix; synthetic benchmarks were 60x30 C2, 1000x100 C2, 1000x500 C10, 100x500 C8)
- Response vector y (class labels or continuous response, supporting regression and classification)
- Optional integer K: number of features to be selected

## Outputs

- 'features' output: selected feature indices in descending order of importance, ranked by column index
- Feature scores reflecting the relevance/redundancy/complementarity trade-off per candidate feature
- FDR values when evaluated at the number of true features on benchmark data (e.g. written to synthetic_fdr_results.csv)

## How to apply

Compute Spearman rank correlation coefficients between each feature and the response, and between feature pairs. Apply the paper's nonlinear information theoretic transformation of these coefficients to quantify feature-response association strength and overlapping information between features, explicitly accounting for feature interactions; quantifying complementarity additionally uses partial correlation coefficients. In the uts (RRCT) MATLAB implementation, call the function on the data matrix with the optional K input (number of features to select; default K = M, or K = 30 when the dataset has more than 30 features), and collect the 'features' output listing selected features in descending order of importance ranked by column index. Validate on synthetic benchmarks by measuring the false discovery rate (FDR) at the number of true features — the fraction of selected features that are irrelevant probes — where RRCT achieved FDR of 0, 0, 0, and 0.10 on Synthetics 1-4.

## Related tools

- **uts (RRCT)** (MATLAB function implementing the rank-correlation-based RRCT scoring; takes X, y and optional K, returns features ranked by importance) — github.com/ThanasisTsanas/RRCT
- **mRMR_Spearman** (Companion implementation using Spearman correlation instead of mutual information for the mRMR criterion) — ThanasisTsanas/mRMR_Spearman
- **MATLAB 2021b** (Environment in which all RRCT experiments and synthetic-benchmark FDR computations were run)

## Examples

```
features = uts(X, y, K);  % K optional; default K = M, or K = 30 when the dataset has more than 30 features
```

## Evaluation signals

- FDR measured at the number of true features should be near zero (RRCT: 0, 0, 0 on Synthetics 1-3; 0.10 on Synthetic 4, missing only the last true feature of the fat dataset)
- All true features of each synthetic dataset should appear in the top-ranked 'features' output on Synthetics 1-3
- Selected-feature output must be returned in descending order of importance, ranked by column index
- Results should be comparable to GSO and IAMB, which also achieved FDR of 0, 0, 0, and 0.10 on the same four synthetics

## Limitations

- Popular algorithms such as RELIEF, mRMR Peng (the default implementation most studies use), CFS, and JMI may stumble and erroneously select probes on synthetic data — verify probe rejection rather than assuming filter robustness
- The heuristic is correlation-based; complementarity is quantified via partial correlation coefficients, which may not capture non-linear interactions beyond what rank correlations expose
- RRCT is a heuristic, not a provably optimal selector; performance was validated empirically on four synthetic and eight real-world datasets rather than guaranteed theoretically

## Evidence

- [intro] RRCT uses a nonlinear transformation of correlation coefficients with information theoretic concepts to quantify feature-response association and overlapping information between features, explicitly accounting for feature interactions: "RRCT uses a simple nonlinear transformation of the correlation coefficients using information theoretic concepts to quantify the association of the features with the response and the overlapping"
- [methods] uts optional K input selects number of features; output is features in descending importance ranked by column index: "K        -> number of features to be selected (integer value)"
- [methods] uts output 'features' returns selected features in descending order of importance: "features -> The selected features in descending order of importance (ranked by column index)"
- [results] GSO, IAMB, and RRCT all very effective at discarding probes on the four synthetic datasets: "The results in Figure 1 and Table 1 illustrate that GSO, IAMB, and RRCT are all very effective at discarding probes."
- [readme] README: Spearman correlation criterion allows fast, computationally inexpensive feature selection: [evidence span withheld — anchor status: external_doc]
