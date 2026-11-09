---
name: gene-marker-selection
description: Use when scRNA-seq data contains many phenotype groups (e.g., cell-type/tissue
  combinations) with overlapping DE genes and a single global marker set is needed
  that distinguishes every phenotype pair, not just markers per individual cell type.
when_to_use_negative:
- When only a small number of distinct groups is present and standard per-group differential
  expression suffices — PC targets many groups with overlapping DE genes
- When the goal is an all-relevant feature ranking rather than a minimal globally
  discriminative cover (e.g., use mRMR, a minimal-optimal alternative)
- When an optimal (not approximate) solution to the cover is required — G-PC is a
  polynomial-time approximation and NP-completeness of multiset multicover precludes
  guaranteed optima
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Python package providing GreedyPC (extended greedy) and CEMPC (cross-entropy
    method) solvers for the phenotype cover problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: 'Primary solver: iteratively picks features with largest corrected multiplicity-weighted
    coverage gain until all phenotype pairs are covered'
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Alternative solver based on the cross-entropy method for the same PC problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion package for the multiset multicover formulation into which PC is
    reduced
- name: numpy
  role: Array computation for the score matrix M and integer rounding of multiplicities
- name: scikit-learn
  role: Logistic regression training and F1 evaluation of the selected marker set
    on left-out data
- name: mrmr
  role: Minimum-redundancy maximum-relevance feature selection, a minimal-optimal
    competitor/baseline for marker selection
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:7c1d908deeaf6bb64335e883631d84d711fe80dd00e7a1de1bfbae16f4f4f385
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-marker-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a compact, globally discriminative gene marker set for scRNA-seq phenotypes by formulating marker selection as a phenotype cover (PC) problem, equivalent to NP-complete multiset multicover, and approximating it in polynomial time with the extended-greedy G-PC algorithm (or the cross-entropy CEM-PC alternative).

## When to use

Use when scRNA-seq data contains many phenotype groups (e.g., cell-type/tissue combinations) with overlapping DE genes and a single global marker set is needed that distinguishes every phenotype pair, not just markers per individual cell type.

## When NOT to use

- When only a small number of distinct groups is present and standard per-group differential expression suffices — PC targets many groups with overlapping DE genes
- When the goal is an all-relevant feature ranking rather than a minimal globally discriminative cover (e.g., use mRMR, a minimal-optimal alternative)
- When an optimal (not approximate) solution to the cover is required — G-PC is a polynomial-time approximation and NP-completeness of multiset multicover precludes guaranteed optima

## Inputs

- scRNA-seq expression dataset (e.g., mouse cortex (MC) scRNA-seq from Zeisel et al., IPF lung from Adams et al., or human cell atlas (HCA) from He et al.) with phenotype/cell-type labels
- Phenotype x feature score matrix M
- Desired coverage K (integer multiset multicover requirement)

## Outputs

- Selected marker set S (list of genes/features covering every phenotype pair at least K times)
- Phenotype-pair coverage attained by S
- F1 score of a logistic regression trained on S and evaluated on left-out data (e.g., written to gpc_results.csv)

## How to apply

Compute the phenotype x feature score matrix M from a scRNA-seq dataset (e.g., the mouse cortex dataset from Zeisel et al.), then round entries of M to integer multiplicities per the multiset multicover formulation, where a feature s covers phenotype pair (i,j) when Mi,s >= Mj,s + 1. Run GreedyPC from the phenotype-cover package, which iteratively selects the feature with the largest corrected multiplicity-weighted coverage gain until every phenotype pair is covered to the desired coverage K (O(kP^2F) complexity; a C++ implementation with Python bindings is the fastest variant). Fix a coverage (e.g., 8 for GSEA against the HuBMAP ASCT+B marker set) and, for each phenotype i, select from the solution S all genes s for which there exists phenotype j with Mi,s >= Mj,s + 1. Evaluate the selected set S by training a scikit-learn logistic regression on a subset of the data, measuring F1 on left-out data, and reporting phenotype-pair coverage achieved. Alternatively use CEMPC (cross-entropy method) as a second polynomial-time approximation. Note the greedy G-PC does not guarantee an optimal solution.

## Related tools

- **phenotype-cover** (Python package providing GreedyPC (extended greedy) and CEMPC (cross-entropy method) solvers for the phenotype cover problem) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Primary solver: iteratively picks features with largest corrected multiplicity-weighted coverage gain until all phenotype pairs are covered) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Alternative solver based on the cross-entropy method for the same PC problem) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion package for the multiset multicover formulation into which PC is reduced)
- **numpy** (Array computation for the score matrix M and integer rounding of multiplicities)
- **scikit-learn** (Logistic regression training and F1 evaluation of the selected marker set on left-out data)
- **mrmr** (Minimum-redundancy maximum-relevance feature selection, a minimal-optimal competitor/baseline for marker selection) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100)
```

## Evaluation signals

- The selected marker set S attains the paper's reported phenotype-pair coverage on the chosen dataset (e.g., the mouse cortex dataset)
- F1 of the logistic regression trained on S matches the reported held-out performance within a small tolerance
- gpc.coverage_per_iter_ and gpc.n_elements_remaining_per_iter_ show monotone coverage growth with decreasing remaining pairs across greedy iterations
- gpc.pairs_with_incomplete_cover_ is empty (or its size is reported) at the target coverage K
- Selected genes overlap plausibly with reference marker sets such as the HuBMAP ASCT+B marker set in GSEA

## Limitations

- G-PC does not provide an optimal solution — while it worked well on the analyzed data, optimality is not guaranteed
- Other coverage-optimizing approximation algorithms remain untested on biological data
- Biological analysis relied on overlap with existing gene marker lists that may be incomplete; no thorough analysis of selected genes' relation to each cell type was performed
- Raw counts may need log-transformation before fitting (per the phenotype-cover README)

## Evidence
