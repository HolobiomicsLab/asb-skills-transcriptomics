---
name: multiset-multicover-approximation
description: Use when a scRNA-seq or similar dataset has many phenotype groups (e.g.,
  cell-type/tissue combinations) with overlapping DE genes, and a global marker set
  is needed that discriminates every phenotype pair at least K times.
when_to_use_negative:
- When only a small number of distinct groups is present and standard per-group differential-expression
  marker selection suffices (the paper notes DE-based selection breaks down with many
  groups with overlapping DE genes)
- When a provably optimal minimum marker set is required — G-PC is a polynomial-time
  approximation and does not provide an optimal solution
- When the goal is all-relevant feature identification rather than a minimal-optimal
  discriminative subset (e.g., Boruta-style tasks)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Python package providing GreedyPC (extended greedy) and CEMPC (cross-entropy
    method) solvers for the phenotype cover problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion package implementing multiset multicover, to which PC is equivalent;
    installed alongside phenotype-cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Extended-greedy solver that iteratively picks the feature with the largest
    corrected multiplicity-weighted coverage gain; C++ implementation with Python
    bindings is fastest
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Alternative cross-entropy-method approximation algorithm for the same PC problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: 'Used for evaluation: train logistic regression on the selected marker subset
    and measure F1 on left-out data'
- name: numpy
  role: Dependency of phenotype-cover used to compute and round the phenotype × feature
    score matrix M into integer multiplicities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:9bd7534d1dce51879f16db33a6b845b8ebbcba36a6b857a4b7046aa163f7795d
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
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multiset-multicover-approximation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Approximates the phenotype cover (PC) marker-selection problem — shown to be equivalent to the NP-complete multiset multicover problem — using polynomial-time algorithms: G-PC (extended greedy set cover) and CEM-PC (cross-entropy method), to select a small feature set that distinguishes every phenotype pair to a desired coverage K.

## When to use

Use when a scRNA-seq or similar dataset has many phenotype groups (e.g., cell-type/tissue combinations) with overlapping DE genes, and a global marker set is needed that discriminates every phenotype pair at least K times.

## When NOT to use

- When only a small number of distinct groups is present and standard per-group differential-expression marker selection suffices (the paper notes DE-based selection breaks down with many groups with overlapping DE genes)
- When a provably optimal minimum marker set is required — G-PC is a polynomial-time approximation and does not provide an optimal solution
- When the goal is all-relevant feature identification rather than a minimal-optimal discriminative subset (e.g., Boruta-style tasks)

## Inputs

- Feature-by-cell expression matrix X (log-transform if working with raw scRNA-seq counts)
- Phenotype/class label vector y (e.g., cell types from the mouse cortex, IPF lung, or human cell atlas scRNA-seq datasets)
- Phenotype × feature score matrix M used for marker selection
- Integer coverage parameter K

## Outputs

- Selected marker feature set S (ranked list of features covering every phenotype pair to at least K)
- Coverage attained per greedy iteration (coverage_per_iter_)
- Number of elements remaining per iteration (n_elements_remaining_per_iter_)
- List of phenotype pairs with incomplete cover (pairs_with_incomplete_cover_)
- F1 score of a logistic regression trained on the selected markers and evaluated on left-out data

## How to apply

Compute the phenotype × feature score matrix M, then round its entries to integer multiplicities so that a feature s covers phenotype pair (i,j) when Mi,s >= Mj,s + 1, framing marker selection as a multiset multicover instance. Install and import GreedyPC (or CEMPC) from the phenotype-cover package (pip install phenotype-cover; pip install multiset-multicover), and fit it on the feature matrix X and phenotype labels y; G-PC iteratively picks the feature with the largest corrected multiplicity-weighted coverage gain until all pairs are covered, with O(kP^2F) complexity. Choose a target coverage (e.g., gpc.select(100) for coverage of 100, or a fixed coverage of 8 as in the HuBMAP ASCT+B enrichment analysis) and inspect gpc.coverage_per_iter_, gpc.n_elements_remaining_per_iter_, and gpc.pairs_with_incomplete_cover_. Validate the selected marker set S by training a scikit-learn logistic regression on a subset of the data and measuring F1 on left-out data. For speed, a C++ implementation of G-PC with Python bindings is available and was the fastest method tested.

## Related tools

- **phenotype-cover** (Python package providing GreedyPC (extended greedy) and CEMPC (cross-entropy method) solvers for the phenotype cover problem) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion package implementing multiset multicover, to which PC is equivalent; installed alongside phenotype-cover) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Extended-greedy solver that iteratively picks the feature with the largest corrected multiplicity-weighted coverage gain; C++ implementation with Python bindings is fastest) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Alternative cross-entropy-method approximation algorithm for the same PC problem) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Used for evaluation: train logistic regression on the selected marker subset and measure F1 on left-out data)
- **numpy** (Dependency of phenotype-cover used to compute and round the phenotype × feature score matrix M into integer multiplicities)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100)
```

## Evaluation signals

- The selected marker set attains the paper's reported phenotype-pair coverage on the chosen dataset (e.g., mouse cortex) within a small tolerance
- F1 of a scikit-learn logistic regression trained on the selected markers matches reported held-out performance within a small tolerance
- gpc.coverage_per_iter_ increases monotonically toward the target coverage and n_elements_remaining_per_iter_ decreases to zero
- pairs_with_incomplete_cover_ is empty when the target coverage is attainable (cross-check against gpc.max_coverage())
- Every phenotype pair (i,j) satisfies the coverage condition Mi,s >= Mj,s + 1 for at least K selected features s

## Limitations

- G-PC does not provide an optimal solution, even though it worked well on the data analyzed in the paper
- PC/multiset multicover is NP-complete, so only polynomial-time approximations are practical
- Other coverage-optimizing approximation algorithms remain untested on biological data
- X may need log-transformation when working with raw counts, per the README

## Evidence
