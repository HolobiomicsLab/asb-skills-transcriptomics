---
name: greedy-set-cover-implementation
description: Use when you need a compact, globally discriminative marker set from
  a phenotype × feature score matrix (e.g., scRNA-seq cell-type × gene scores) such
  that every phenotype pair is covered (distinguished) at least K times.
when_to_use_negative:
- You only need per-group differentially expressed genes for a small number of distinct
  cell types — the paper notes standard DE approaches work well for few distinct groups
  but fail with many overlapping groups.
- You require a provably optimal minimum-size cover — the paper states 'While G-PC
  worked well for the data analyzed in this paper, it does not provide an optimal
  solution.'
- Input is already a compact curated marker list (e.g., HuBMAP ASCT+B markers used
  as reference) rather than a matrix requiring selection.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GreedyPC
  role: Core extended-greedy solver that iteratively selects features maximizing corrected
    multiplicity-weighted coverage gain until all phenotype pairs reach coverage K
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy-method alternative to the greedy solver for the same PC/multiset-multicover
    problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Python package providing GreedyPC/CEMPC with a C++ implementation and Python
    bindings, making G-PC the fastest method tested
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion package implementing the multiset-multicover machinery used by the
    PC formulation
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Array backend dependency for loading and rounding the phenotype × feature
    score matrix M
- name: scikit-learn
  role: Trains the logistic regression used to compute held-out F1 for validating
    the selected marker set
- name: mrmr
  role: Competing minimal-optimal feature-selection baseline (minimum redundancy -
    maximum relevance) against which G-PC is compared
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:2e5d2854ccebfddbd0557b2d2295c302d41b35ce53f075843cceb608817356be
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
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# greedy-set-cover-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements G-PC, the extended greedy algorithm for the phenotype cover (PC) problem, which selects a minimal marker set so that every phenotype pair is distinguished at coverage K via a multiset-multicover formulation of the phenotype × feature score matrix M.

## When to use

Use when you need a compact, globally discriminative marker set from a phenotype × feature score matrix (e.g., scRNA-seq cell-type × gene scores) such that every phenotype pair is covered (distinguished) at least K times.

## When NOT to use

- You only need per-group differentially expressed genes for a small number of distinct cell types — the paper notes standard DE approaches work well for few distinct groups but fail with many overlapping groups.
- You require a provably optimal minimum-size cover — the paper states 'While G-PC worked well for the data analyzed in this paper, it does not provide an optimal solution.'
- Input is already a compact curated marker list (e.g., HuBMAP ASCT+B markers used as reference) rather than a matrix requiring selection.

## Inputs

- Phenotype × feature score matrix M (e.g., cell type × gene scores from scRNA-seq datasets such as mouse cortex Zeisel et al., IPF lung Adams et al., or human cell atlas He et al.)
- Integer coverage parameter K (paper examples use coverage of 8 and 100)
- Feature expression matrix X (log-transformed if raw counts) with phenotype labels y

## Outputs

- Selected marker feature set S achieving at least K-fold coverage of every phenotype pair (or a report of pairs_with_incomplete_cover_)
- Coverage-per-iteration and elements-remaining-per-iteration traces (coverage_per_iter_, n_elements_remaining_per_iter_)
- Downstream classification metrics (e.g., logistic-regression F1 on held-out data) and coverage summaries, e.g., written to gpc_results.csv

## How to apply

Formulate the phenotype × feature score matrix M and round its entries to integer multiplicities so that a feature s covers phenotype pair (i,j) when Mi,s >= Mj,s + 1 — this casts PC as multiset multicover, an NP-complete problem, motivating the polynomial-time extended-greedy approximation. Install the C++ implementation with Python bindings via `pip install phenotype-cover` and `pip install multiset-multicover`, then run GreedyPC, which iteratively picks the feature with the largest corrected multiplicity-weighted coverage gain until all pairs reach the requested coverage K (O(kP^2F) complexity). An alternative CEM-PC based on the cross-entropy method is available when greedy behavior is unsuitable. Log-transform X before fitting if working with raw counts, as the README advises. Evaluate the selected set S by training a scikit-learn logistic regression on a subset and measuring F1 on left-out data, and optionally use GreedyPC's introspection methods (plot_progress, pairs_with_incomplete_cover_) to audit coverage attainment.

## Related tools

- **GreedyPC** (Core extended-greedy solver that iteratively selects features maximizing corrected multiplicity-weighted coverage gain until all phenotype pairs reach coverage K) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy-method alternative to the greedy solver for the same PC/multiset-multicover problem) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Python package providing GreedyPC/CEMPC with a C++ implementation and Python bindings, making G-PC the fastest method tested) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion package implementing the multiset-multicover machinery used by the PC formulation) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Array backend dependency for loading and rounding the phenotype × feature score matrix M)
- **scikit-learn** (Trains the logistic regression used to compute held-out F1 for validating the selected marker set)
- **mrmr** (Competing minimal-optimal feature-selection baseline (minimum redundancy - maximum relevance) against which G-PC is compared) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(8)
```

## Evaluation signals

- The selected marker set S must attain the paper's reported coverage on the chosen dataset (e.g., coverage of 8 in the HuBMAP GSEA analysis) — every phenotype pair (i,j) has at least K features s with Mi,s >= Mj,s + 1.
- Held-out logistic-regression F1 (train on a subset, evaluate on left-out data) must match the paper's reported performance within a small tolerance.
- greedy PC is expected to be the fastest tested method owing to the C++ implementation with Python bindings (Table 2), so runtime should be consistent with that claim.
- GreedyPC diagnostics should be consistent: coverage_per_iter_ increases monotonically toward the target and pairs_with_incomplete_cover_ is empty when the target coverage is attainable.
- Stability check: resampling half the data over 5 runs and computing a stability index for selected features should show reasonable agreement across runs.

## Limitations

- G-PC does not provide an optimal solution to the multiset-multicover/PC problem.
- Raw counts need log-transformation before fitting; the score matrix must be correctly rounded to integer multiplicities or the cover semantics break.
- Other coverage-optimizing approximation algorithms remain untested on biological data; G-PC and CEM-PC were validated only on the analyzed datasets.
- Biological validation relied on overlap with existing gene marker lists (e.g., HuBMAP ASCT+B) that may be incomplete.

## Evidence
