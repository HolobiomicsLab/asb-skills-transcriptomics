---
name: greedy-vs-cross-entropy-marker-solver-benchmarking
description: Use when you need to quantify how reproducible the feature sets selected
  by G-PC or CEM-PC are across repeated runs on randomly sampled scRNA-seq data, e.g.
  IPF lung, mouse cortex, or Human Cell Atlas matrices.
when_to_use_negative:
- When the goal is to evaluate discriminative classification accuracy of the marker
  sets rather than their run-to-run reproducibility
- When benchmarking deterministic global marker-selection methods (e.g. ReliefF, F
  values) alone, whose stability behavior the paper reports separately from G-PC and
  CEM-PC
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: phenotype-cover
  role: Python package providing the GreedyPC and CEMPC solvers whose marker selections
    are compared for stability across resampled runs
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: G-PC solver based on the extended greedy set cover algorithm; its selected
    feature sets are benchmarked for ~70% cross-run retention
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: CEM-PC cross-entropy-method solver; expected to yield the lowest stability
    index due to its random sampling nature
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Used for random cell resampling and set-based overlap-over-union computation
    between runs' feature sets
- name: scikit-learn
  role: Dependency of phenotype-cover and source of synthetic test data (e.g. make_classification)
    for exercising the solvers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:98f645511d4ad378fd43f62c8d7db60b0f41c36aaaadb333f554a00ec329357e
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
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# greedy-vs-cross-entropy-marker-solver-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the stability of marker sets produced by the phenotype-cover solvers GreedyPC (G-PC) and CEMPC (CEM-PC) by repeatedly re-running each solver on randomly resampled half-datasets and computing pairwise overlap statistics of the selected features.

## When to use

Use when you need to quantify how reproducible the feature sets selected by G-PC or CEM-PC are across repeated runs on randomly sampled scRNA-seq data, e.g. IPF lung, mouse cortex, or Human Cell Atlas matrices.

## When NOT to use

- When the goal is to evaluate discriminative classification accuracy of the marker sets rather than their run-to-run reproducibility
- When benchmarking deterministic global marker-selection methods (e.g. ReliefF, F values) alone, whose stability behavior the paper reports separately from G-PC and CEM-PC

## Inputs

- Preprocessed scRNA-seq expression matrix (cells x genes) with phenotype/cell-type labels, e.g. from IPF lung (Adams et al.), mouse cortex (Zeisel et al.), or Human Cell Atlas (He et al.) datasets
- Solver hyperparameters: number of runs (5), resample fraction (half the data), marker set size or coverage level

## Outputs

- Per-method stability index computed from pairwise overlap-over-union statistics across the 5 runs
- Fraction of features retained across runs per method (expected ~0.70 for G-PC)
- Stability results table written to stability_index.csv

## How to apply

Load a preprocessed expression matrix (e.g. IPF lung, mouse cortex Zeisel, or HCA He scRNA-seq) with numpy/pandas, then for each of 5 runs randomly sample half the cells and fit GreedyPC and CEMPC from the phenotype-cover package, selecting a marker set at a fixed coverage level. For each method compute the pairwise overlap-over-union statistic between feature sets from different runs using numpy set operations, aggregate into a per-method stability index, and record the fraction of features retained across runs. The rationale, per the article, is that G-PC's greedy sequential nature makes it less stable than global methods like ReliefF and F values (~70% feature retention across runs), while CEM-PC's random sampling nature makes it the least stable method. Write the aggregated per-method stability values to a results table (e.g. stability_index.csv) and compare methods to confirm G-PC outperforms CEM-PC on stability while still providing competitive discriminative marker sets.

## Related tools

- **phenotype-cover** (Python package providing the GreedyPC and CEMPC solvers whose marker selections are compared for stability across resampled runs) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (G-PC solver based on the extended greedy set cover algorithm; its selected feature sets are benchmarked for ~70% cross-run retention) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (CEM-PC cross-entropy-method solver; expected to yield the lowest stability index due to its random sampling nature) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Used for random cell resampling and set-based overlap-over-union computation between runs' feature sets)
- **scikit-learn** (Dependency of phenotype-cover and source of synthetic test data (e.g. make_classification) for exercising the solvers)

## Examples

```
from phenotype_cover import GreedyPC, CEMPC; gpc = GreedyPC(); gpc.fit(X_sub, y_sub); features = gpc.select(100)
```

## Evaluation signals

- G-PC stability index is close to the reported ~0.70 feature-retention rate across the 5 runs
- CEM-PC has the lowest stability index among the tested marker-selection methods
- All pairwise overlap-over-union values are in [0, 1] and the aggregated per-method index is a valid average of 5-choose-2 pairwise statistics
- stability_index.csv exists with one row per method and no missing values
- Each run uses a disjoint random half-sample of cells so that resampling variance is actually present

## Limitations

- G-PC does not provide an optimal solution; its greedy sequential nature makes it less stable than global methods like ReliefF and F values
- CEM-PC relies on random sampling, making it the least stable method and introducing run-to-run variance in the selected feature sets
- Stability was tested by sampling half the data over only 5 runs, so estimates of the stability index carry sampling noise

## Evidence

- [results] G-PC retains 60-70% of genes across runs; CEM-PC least stable: [evidence span withheld — anchor status: not_found]
