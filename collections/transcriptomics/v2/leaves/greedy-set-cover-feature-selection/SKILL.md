---
name: greedy-set-cover-feature-selection
description: Use when you need a global, discriminative marker set across many overlapping
  phenotype pairs (e.g., cell-type/tissue combinations in scRNA-seq data) rather than
  per-group differentially expressed genes, and can supply a phenotype-pair × feature
  score matrix.
when_to_use_negative:
- When only a small number of distinct groups is present and standard per-group differential
  expression (top DE genes) suffices — the PC formulation targets many groups with
  overlapping DE genes
- When a provably optimal solution to the multicover problem is required — G-PC is
  a greedy approximation and 'does not provide an optimal solution'
- When the input is already a feature table rather than raw counts needing preprocessing
  into the phenotype × feature score matrix M
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Provides the GreedyPC class implementing the greedy phenotype-cover marker-selection
    algorithm
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Dependency package to install alongside phenotype-cover for the multicover
    formulation underlying GreedyPC
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Alternative cross-entropy-method solver for the same PC problem, usable as
    a comparison against the greedy selection
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Dependency of phenotype-cover; supplies make_classification for examples and
    logistic regression for evaluating selected feature sets
- name: numpy
  role: Dependency of phenotype-cover used to load and preprocess count matrices into
    the score matrix format
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:ae933ef1b8227d80e39616c1ef63208a0491fe578d3e5a4206194f6d225e2b57
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
  - C++
  - CEMPC
  - GreedyPC
  - Python
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# greedy-set-cover-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Greedy feature selection (G-PC) based on the extended greedy algorithm to set cover, applied to the phenotype cover (PC) marker-selection problem formulated over a phenotype × feature score matrix and shown equivalent to NP-complete multiset multicover. Implemented as GreedyPC in the phenotype-cover Python package, it greedily approximates a small marker set such that every phenotype pair is covered to a chosen coverage level.

## When to use

Use when you need a global, discriminative marker set across many overlapping phenotype pairs (e.g., cell-type/tissue combinations in scRNA-seq data) rather than per-group differentially expressed genes, and can supply a phenotype-pair × feature score matrix.

## When NOT to use

- When only a small number of distinct groups is present and standard per-group differential expression (top DE genes) suffices — the PC formulation targets many groups with overlapping DE genes
- When a provably optimal solution to the multicover problem is required — G-PC is a greedy approximation and 'does not provide an optimal solution'
- When the input is already a feature table rather than raw counts needing preprocessing into the phenotype × feature score matrix M

## Inputs

- scRNA-seq count matrix (e.g., IPF lung Adams et al., mouse cortex Zeisel et al., human cell atlas He et al.) preprocessed via filtering, normalization, and log transform
- Phenotype labels y per cell (phenotype/context classes such as cell type and tissue combinations)
- Phenotype × feature score matrix M of phenotype-pair discriminative scores
- Integer coverage parameter (e.g., 8 in the paper's GSEA analysis) or target number of features (e.g., 42, 66, 121, 178 depending on dataset/task)

## Outputs

- Ranked/ordered set of selected marker features S (e.g., gpc.select(coverage) returning K features)
- Diagnostics: n_elements_remaining_per_iter_, coverage_per_iter_, pairs_with_incomplete_cover_, max_coverage()
- Macro-F1 scores from logistic regression on left-out cells (e.g., G-PC 0.70 on IPF with 42 genes, 0.94-0.95 on MC, 0.68 on HCA)

## How to apply

Preprocess the scRNA-seq count matrix (gene/cell filtering, normalization, log transform; log-transform X if working with raw counts), then compute phenotype-pair discriminative scores to build the phenotype × feature score matrix M that the PC problem optimizes over. Instantiate GreedyPC from the phenotype-cover package (which depends on numpy, matplotlib, and scikit-learn, and requires multiset-multicover installed via pip), call fit(X, y), then select features at an explicit coverage value — e.g., a fixed coverage of 8 was used in the paper, selecting from the solution S all genes s for which some phenotype j satisfies M(i,s) >= M(j,s) + 1. GreedyPC is a polynomial-time approximation (not an optimal solution) to multiset multicover; choose coverage and feature-count operating points per dataset (e.g., 42 genes on IPF, 178 on IPF runtime runs, 66 on MC, 121 on HCA). Downstream, train a logistic regression model on the subset used for feature selection and evaluate macro-F1 on left-out cells to validate the selected markers.

## Related tools

- **phenotype-cover** (Provides the GreedyPC class implementing the greedy phenotype-cover marker-selection algorithm) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Dependency package to install alongside phenotype-cover for the multicover formulation underlying GreedyPC) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Alternative cross-entropy-method solver for the same PC problem, usable as a comparison against the greedy selection) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Dependency of phenotype-cover; supplies make_classification for examples and logistic regression for evaluating selected feature sets)
- **numpy** (Dependency of phenotype-cover used to load and preprocess count matrices into the score matrix format)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(8)
```

## Evaluation signals

- Reproduced G-PC macro-F1 reaches the reported values: 0.70 on IPF with 42 genes, 0.94-0.95 on MC, 0.68 on HCA, outperforming scGeneFit (0.65) and CEM-PC (0.61) on IPF
- All phenotype pairs (or an inspected set) are covered to the requested coverage; check gpc.pairs_with_incomplete_cover_ is empty and coverage_per_iter_ increases monotonically per iteration
- Selected feature count matches the specified operating point (e.g., 42 genes on IPF, 66 on MC, 121 on HCA)
- C++ GreedyPC with Python bindings runs in under 2 seconds on IPF, MC, and HCA, faster than CEM-PC and all competitor methods (Table 2)
- Number of elements remaining decreases every greedy iteration (inspect n_elements_remaining_per_iter_ via gpc.plot_progress())

## Limitations

- G-PC is a greedy approximation and 'does not provide an optimal solution' to the multiset multicover problem
- The problem is NP-complete, so the greedy algorithm only approximates the minimum cover in polynomial time
- Raw count data must be log-transformed before fitting (README: 'You may need to log-transform X if working with raw counts')
- Some phenotype pairs may remain covered below the desired coverage threshold (pairs_with_incomplete_cover_ may be non-empty)

## Evidence
