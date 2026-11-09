---
name: scrnaseq-half-sample-resampling
description: Use when comparing marker-selection methods (e.g. G-PC, CEM-PC, mRMR,
  ReliefF, F values) on scRNA-seq expression matrices and you need to quantify how
  reproducible each method's selected feature sets are across repeated random resampling.
when_to_use_negative:
- The goal is classification/deconvolution accuracy rather than reproducibility of
  the selected feature sets — use the logistic-regression train/test split or pseudo-bulk
  deconvolution evaluation instead
- You need a globally optimal marker set — note that G-PC does not provide an optimal
  solution, so stability of its output does not certify optimality
- The expression matrix is raw counts that have not been preprocessed/log-transformed,
  since the solvers expect a suitable score/normalized matrix
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Provides the GreedyPC and CEMPC solvers re-run on each half-sample resample
    to produce the feature sets whose stability is measured
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Used for random cell subsampling and set operations computing pairwise overlap-over-union
    statistics across runs
- name: mrmr_selection
  role: Competitor minimum-redundancy-maximum-relevance feature selector whose stability
    can be benchmarked against G-PC and CEM-PC
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:8b640b790392a2f8224a3814740311f331e37ad61d0f82f60c6b30bd920997fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - CEMPC
  - GreedyPC
  - Python
  - mrmr_selection
  - numpy
  - phenotype-cover
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scrnaseq-half-sample-resampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stability-assessment protocol for marker/feature selection on scRNA-seq data: randomly sample half the cells over 5 runs, re-run each selection method per resample, and quantify pairwise feature-set overlap (overlap-over-union) to compute a per-method stability index.

## When to use

Use when comparing marker-selection methods (e.g. G-PC, CEM-PC, mRMR, ReliefF, F values) on scRNA-seq expression matrices and you need to quantify how reproducible each method's selected feature sets are across repeated random resampling.

## When NOT to use

- The goal is classification/deconvolution accuracy rather than reproducibility of the selected feature sets — use the logistic-regression train/test split or pseudo-bulk deconvolution evaluation instead
- You need a globally optimal marker set — note that G-PC does not provide an optimal solution, so stability of its output does not certify optimality
- The expression matrix is raw counts that have not been preprocessed/log-transformed, since the solvers expect a suitable score/normalized matrix

## Inputs

- Preprocessed scRNA-seq gene-expression matrix (cells x genes) with phenotype/cell-type labels (e.g. IPF lung, mouse cortex Zeisel et al., or HCA He et al. datasets)
- Fitted marker-selection solver (e.g. GreedyPC or CEMPC from phenotype_cover) or competitor feature selector (mRMR, ReliefF, F values)
- Number of runs (5) and resample fraction (half the data)

## Outputs

- Per-method stability index aggregated from pairwise overlap-over-union statistics
- Fraction of features retained across runs (e.g. ~0.70 for G-PC)
- stability_index.csv results table

## How to apply

Load a preprocessed scRNA-seq expression matrix (e.g. the IPF lung, mouse cortex, or Human Cell Atlas datasets used in the paper) with numpy/pandas. For each of 5 runs, randomly sample half the cells and run the selection method — GreedyPC or CEMPC from the phenotype-cover package, or a competitor such as mrmr_classif — on each resample. For each method, compute the pairwise overlap-over-union statistic between the feature sets from different runs using numpy set operations, then aggregate into a per-method stability index and record the fraction of features retained across runs. The rationale is that greedy sequential methods like G-PC are expected to be less stable than global methods like ReliefF and F values yet still retain roughly 60-70% of genes across runs, while CEM-PC should be the least stable because of its random-sampling (cross-entropy) nature. Write the results to stability_index.csv and compare methods against these expected orderings.

## Related tools

- **phenotype-cover** (Provides the GreedyPC and CEMPC solvers re-run on each half-sample resample to produce the feature sets whose stability is measured) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Used for random cell subsampling and set operations computing pairwise overlap-over-union statistics across runs)
- **mrmr_selection** (Competitor minimum-redundancy-maximum-relevance feature selector whose stability can be benchmarked against G-PC and CEM-PC) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X_half, y_half); feats = gpc.select(100)
```

## Evaluation signals

- G-PC stability index is close to the reported ~0.70 feature retention (60-70% of the same genes across runs)
- CEM-PC has the lowest stability index among all methods tested, consistent with its random cross-entropy sampling nature
- Exactly 5 runs with half the cells randomly sampled per run are performed
- stability_index.csv is written with one aggregated stability index per method
- Global methods such as ReliefF and F values show higher stability than G-PC

## Limitations

- G-PC is less stable than global methods like ReliefF and F values due to its greedy sequential nature
- CEM-PC is the least stable method because of its random sampling (cross-entropy) nature
- Stability alone does not indicate solution optimality — G-PC does not provide an optimal solution

## Evidence

- [results] G-PC vs CEM-PC stability finding: [evidence span withheld — anchor status: not_found]
