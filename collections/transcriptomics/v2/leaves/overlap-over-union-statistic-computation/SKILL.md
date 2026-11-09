---
name: overlap-over-union-statistic-computation
description: Use when you have feature sets selected by a method (e.g., G-PC, CEM-PC,
  mrmr, ReliefF, F) over multiple runs on randomly sampled data and need to measure
  how reproducible the selected gene/feature sets are across runs.
when_to_use_negative:
- Only a single feature set exists (no repeated runs over resampled data), so no pairwise
  overlap statistics can be computed
- The goal is discriminative performance (classification accuracy or deconvolution
  error) rather than selection reproducibility
- Feature sets come from datasets with non-matching feature spaces (different gene
  identifiers), making set overlap meaningless
tools:
- name: GreedyPC
  role: Greedy phenotype-cover solver run on each of 5 random half-samples to produce
    the feature sets whose cross-run stability is measured (expected ~60-70% retention)
- name: CEMPC
  role: Cross-entropy-method solver run on the same resamples; its random sampling
    nature makes it the least-stable baseline in the comparison
- name: numpy
  role: Set operations used to compute pairwise overlap-over-union statistics and
    aggregate them into per-method stability indices
- name: mrmr
  role: Competing minimum-redundancy-maximum-relevance feature-selection method whose
    stability can be benchmarked against the PC solvers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:fed0d77b8db2cb56eb44be2882fd554e4f39afd7aacbd7aa079e5954cb48c802
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  tools:
  - CEMPC
  - GreedyPC
  - Python
  - mrmr
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# overlap-over-union-statistic-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the reproducibility (stability) of marker-selection procedures by computing pairwise overlap-over-union statistics between feature sets selected across repeated runs on randomly resampled data, then aggregating them into a per-method stability index. Used in Hasanaj et al. (Cell Reports Methods, 2022) to show G-PC retains ~60-70% of genes across runs while CEM-PC is the least stable.

## When to use

Use when you have feature sets selected by a method (e.g., G-PC, CEM-PC, mrmr, ReliefF, F) over multiple runs on randomly sampled data and need to measure how reproducible the selected gene/feature sets are across runs.

## When NOT to use

- Only a single feature set exists (no repeated runs over resampled data), so no pairwise overlap statistics can be computed
- The goal is discriminative performance (classification accuracy or deconvolution error) rather than selection reproducibility
- Feature sets come from datasets with non-matching feature spaces (different gene identifiers), making set overlap meaningless

## Inputs

- Preprocessed scRNA-seq expression matrix with phenotype/cell-type labels (e.g., IPF lung, mouse cortex, or Human Cell Atlas datasets)
- Selected marker/feature sets from 5 runs per method on randomly sampled half-data (e.g., from GreedyPC and CEMPC solvers)
- List of competing marker-selection methods to compare (G-PC, CEM-PC, ReliefF, F, mrmr)

## Outputs

- Pairwise overlap-over-union statistics between feature sets from different runs, per method
- Per-method aggregated stability index
- Fraction of features retained across runs (e.g., ~0.70 for G-PC)
- Results table written to stability_index.csv

## How to apply

Load a preprocessed expression matrix (e.g., the IPF lung, mouse cortex, or HCA scRNA-seq datasets) and, for each of 5 runs, randomly sample half the cells and run the selection method — importing GreedyPC or CEMPC from the phenotype-cover package — to obtain a marker set per resample. For each method, compute the pairwise overlap-over-union (Jaccard) statistic between the feature sets from every pair of runs using numpy set operations. Aggregate the pairwise statistics into a per-method stability index and record the fraction of features retained across runs. Validate against the paper's calibration points: G-PC should retain approximately 60-70% (stability index ~0.70) of the same genes across runs, global methods such as ReliefF and F values should be more stable than G-PC, and CEM-PC — due to its random sampling nature — should show the lowest stability. Write the aggregated results table to stability_index.csv.

## Related tools

- **GreedyPC** (Greedy phenotype-cover solver run on each of 5 random half-samples to produce the feature sets whose cross-run stability is measured (expected ~60-70% retention))
- **CEMPC** (Cross-entropy-method solver run on the same resamples; its random sampling nature makes it the least-stable baseline in the comparison)
- **numpy** (Set operations used to compute pairwise overlap-over-union statistics and aggregate them into per-method stability indices)
- **mrmr** (Competing minimum-redundancy-maximum-relevance feature-selection method whose stability can be benchmarked against the PC solvers)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X_half, y_half); features = gpc.select(100)
```

## Evaluation signals

- Computed G-PC stability index is close to the reported ~0.70 feature retention (60-70% of the same genes across runs)
- CEM-PC has the lowest stability index among all tested methods
- All pairwise overlap-over-union values fall within [0, 1]
- Global methods such as ReliefF and F values show higher stability than the greedy G-PC
- stability_index.csv is produced with one aggregated stability entry per method

## Limitations

- G-PC is less stable than global methods like ReliefF and F values because of its greedy sequential nature
- CEM-PC is the least stable method because of its random sampling nature
- G-PC does not provide an optimal solution, so stability results reflect an approximation rather than a unique optimum

## Evidence
