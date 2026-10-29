---
name: cross-run-feature-set-comparison
description: Use when repeating a marker-selection procedure (e.g., phenotype-cover
  G-PC/CEM-PC) over multiple resampled runs and you need a stability index showing
  how much of the feature set is retained across runs.
when_to_use_negative:
- When the goal is classification accuracy or deconvolution performance rather than
  reproducibility of the selected marker set
- When only a single deterministic selection run is planned — stability metrics require
  multiple resampled runs
- When the data are not split into resamples with a shared phenotype/label structure,
  so feature sets across runs are not comparable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: phenotype-cover
  role: Provides GreedyPC and CEMPC solvers whose selected marker sets are compared
    across resampled runs
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Greedy sequential solver whose feature retention (~60-70% across runs) is
    quantified by the stability index
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy solver expected to show the lowest stability due to its random
    sampling nature
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Used for set operations computing pairwise overlap-over-union statistics and
    aggregating stability indices
- name: mrmr
  role: Competing minimum-redundancy-maximum-relevance selection method against which
    PC stability can be compared
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:8c801e239dc93f00a8a3bcd7d165ac7fc921d2d9bb56d947a7b36721521cf27f
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
  - mrmr
  - numpy
  - phenotype-cover
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-run-feature-set-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the stability of marker/feature sets selected by G-PC (GreedyPC), CEM-PC (CEMPC), and competing methods (e.g., ReliefF, F values, mRMR) by repeating selection over 5 runs on randomly sampled halves of scRNA-seq data and computing pairwise overlap statistics across runs.

## When to use

Use when repeating a marker-selection procedure (e.g., phenotype-cover G-PC/CEM-PC) over multiple resampled runs and you need a stability index showing how much of the feature set is retained across runs.

## When NOT to use

- When the goal is classification accuracy or deconvolution performance rather than reproducibility of the selected marker set
- When only a single deterministic selection run is planned — stability metrics require multiple resampled runs
- When the data are not split into resamples with a shared phenotype/label structure, so feature sets across runs are not comparable

## Inputs

- Preprocessed gene-expression matrix (e.g., IPF lung, mouse cortex, or HCA scRNA-seq data) loaded via numpy/pandas
- Phenotype/cell-type labels per cell for supervised feature selection
- Selection method instances: GreedyPC, CEMPC (phenotype-cover), and comparison methods (ReliefF, F values, mRMR)

## Outputs

- Per-method stability index aggregated from pairwise overlap-over-union statistics across 5 runs
- Fraction of features retained across runs per method (G-PC expected near ~0.70)
- Stability results table written to stability_index.csv

## How to apply

Load a preprocessed expression matrix (IPF lung, mouse cortex, or HCA scRNA-seq data) with numpy/pandas. For each of 5 runs, randomly sample half the cells and fit GreedyPC and CEMPC from the phenotype-cover package to select a marker set on each resample. For each method, compute the pairwise overlap-over-union statistic between the feature sets from different runs using numpy set operations, then aggregate the pairwise statistics into a per-method stability index and record the fraction of features retained. Compare methods: expect G-PC to retain roughly 60-70% of the same genes across runs, global methods like ReliefF and F values to be more stable, and CEM-PC to be the least stable due to its random (cross-entropy) sampling nature. Write the stability results table (e.g., to stability_index.csv) and validate the G-PC index against the reported ~0.70 retention.

## Related tools

- **phenotype-cover** (Provides GreedyPC and CEMPC solvers whose selected marker sets are compared across resampled runs) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Greedy sequential solver whose feature retention (~60-70% across runs) is quantified by the stability index) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy solver expected to show the lowest stability due to its random sampling nature) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Used for set operations computing pairwise overlap-over-union statistics and aggregating stability indices)
- **mrmr** (Competing minimum-redundancy-maximum-relevance selection method against which PC stability can be compared) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100)
```

## Evaluation signals

- G-PC stability index is close to the reported ~0.70 feature retention (60-70% of the same genes across runs)
- CEM-PC has the lowest stability index among all tested methods
- Global methods (ReliefF, F values) show higher stability than G-PC, consistent with G-PC's greedy sequential nature
- Pairwise overlap-over-union values are symmetric and bounded in [0,1] for all run pairs
- stability_index.csv contains one stability index per method computed from 5 resampled runs

## Limitations

- G-PC is less stable than global methods like ReliefF and F values due to its greedy sequential nature
- CEM-PC is highly unstable across runs because of its random sampling (cross-entropy) nature
- G-PC does not provide an optimal solution; stability of its output reflects the greedy approximation rather than a global optimum

## Evidence
