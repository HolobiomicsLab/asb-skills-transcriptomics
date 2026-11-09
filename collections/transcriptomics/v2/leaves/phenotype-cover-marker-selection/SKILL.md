---
name: phenotype-cover-marker-selection
description: Use when you must select a small global feature/gene set that jointly
  distinguishes every pair of phenotypes (e.g., cell type × tissue combinations) from
  a phenotype × gene score matrix, where per-group DE genes overlap and pairwise discrimination
  matters.
when_to_use_negative:
- When only a small number of distinct groups exist with non-overlapping differentially
  expressed genes — standard per-group DE marker selection suffices (the paper targets
  many groups with overlapping DE genes).
- When you need an optimal (not approximate) solution — G-PC is a greedy approximation
  and does not provide an optimal solution.
- When input is raw scRNA-seq counts without a phenotype × feature score matrix —
  the matrix must be constructed (and log-transformed) first.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: phenotype-cover
  role: Python package providing the GreedyPC and CEMPC solvers for the phenotype
    cover problem; CEMPC run with I=500, Rs=1000 and 0.98 score threshold
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion pip-installable package required for phenotype-cover (PC is equivalent
    to multiset multicover)
- name: CEMPC
  role: Cross-entropy-method solver used to produce the final IPF marker set exported
    to cem_pc_ipf_selected_markers.csv
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Greedy set cover solver supporting select(K) with a target coverage, progress
    plotting, and pairs_with_incomplete_cover_ inspection
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Dependency of phenotype-cover used for matrix operations on the phenotype
    × gene score matrix
- name: scikit-learn
  role: Dependency of phenotype-cover; used in README examples to generate synthetic
    classification data (make_classification)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:11e31da0c8eeac120c9ff60115ec5453fdb4b736c9106cbed9aa077a8069d08d
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# phenotype-cover-marker-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the phenotype-cover (PC) formulation — equivalent to the NP-complete multiset multicover problem — to select a global set of biomarkers that discriminates every phenotype pair in a phenotype × feature score matrix. Provides two polynomial-time approximators, G-PC (extended greedy set cover) and CEM-PC (cross-entropy method), with CEM-PC finalizing the marker set from features whose probability score exceeds 0.98 after convergence.

## When to use

Use when you must select a small global feature/gene set that jointly distinguishes every pair of phenotypes (e.g., cell type × tissue combinations) from a phenotype × gene score matrix, where per-group DE genes overlap and pairwise discrimination matters.

## When NOT to use

- When only a small number of distinct groups exist with non-overlapping differentially expressed genes — standard per-group DE marker selection suffices (the paper targets many groups with overlapping DE genes).
- When you need an optimal (not approximate) solution — G-PC is a greedy approximation and does not provide an optimal solution.
- When input is raw scRNA-seq counts without a phenotype × feature score matrix — the matrix must be constructed (and log-transformed) first.

## Inputs

- Phenotype × gene score matrix M (e.g., IPF phenotype-gene scores from scRNA-seq data; log-transform X if working with raw counts)
- Phenotype-pair coverage requirements for the phenotype cover problem
- CEM-PC hyperparameters: I = 500 iterations/samples, Rs = 1000 elite-sample parameter; probability-score threshold 0.98

## Outputs

- Selected marker set (list of genes/features) with attained phenotype-pair coverage, e.g., cem_pc_ipf_selected_markers.csv
- For GreedyPC: diagnostics such as n_elements_remaining_per_iter_, coverage_per_iter_, pairs_with_incomplete_cover_

## How to apply

Load a phenotype × gene score matrix (e.g., IPF scRNA-seq scores from Adams et al.) and construct phenotype-pair coverage requirements for the phenotype cover problem. Import GreedyPC or CEMPC from phenotype-cover and fit the solver to the score matrix, using a scoring function that rewards phenotype-pair coverage while penalizing selected-set size. For CEM-PC, configure the cross-entropy method with I = 500 iterations and Rs = 1000 as the elite-sample parameter, run the CEM optimization loop, and extract the final marker set as all features whose probability score exceeds 0.98 (Methods S1, algorithm 3). For G-PC, call select(K) with a desired coverage (e.g., coverage of 8 or 100). Compute the attained phenotype-pair coverage of the selected set and write the gene set plus coverage value to a CSV (e.g., cem_pc_ipf_selected_markers.csv); validate the file parses as CSV with genes and coverage recorded.

## Related tools

- **phenotype-cover** (Python package providing the GreedyPC and CEMPC solvers for the phenotype cover problem; CEMPC run with I=500, Rs=1000 and 0.98 score threshold) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion pip-installable package required for phenotype-cover (PC is equivalent to multiset multicover))
- **CEMPC** (Cross-entropy-method solver used to produce the final IPF marker set exported to cem_pc_ipf_selected_markers.csv) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Greedy set cover solver supporting select(K) with a target coverage, progress plotting, and pairs_with_incomplete_cover_ inspection) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Dependency of phenotype-cover used for matrix operations on the phenotype × gene score matrix)
- **scikit-learn** (Dependency of phenotype-cover; used in README examples to generate synthetic classification data (make_classification))

## Examples

```
pip install multiset-multicover; pip install phenotype-cover; from phenotype_cover import CEMPC; cem = CEMPC(I=500, Rs=1000); cem.fit(M, y); markers = [g for g, p in zip(genes, cem.proba_) if p > 0.98]
```

## Evaluation signals

- Output file (e.g., cem_pc_ipf_selected_markers.csv) exists and parses as CSV containing the selected genes and the attained phenotype-pair coverage value.
- Reported coverage reflects the coverage-rewarding scoring function used during CEM optimization; recomputing pairwise coverage of the selected set matches the recorded value.
- All returned features have CEM probability scores above the 0.98 threshold after convergence.
- For GreedyPC, pairs_with_incomplete_cover_ and coverage_per_iter_ are consistent with the requested coverage (e.g., coverage of 8 or 100).
- Sanity-check against gpc.max_coverage(): pairs that cannot be covered to the desired coverage should appear in pairs_with_incomplete_cover_, not as falsely 'covered'.

## Limitations

- PC/multiset multicover is NP-complete, so both G-PC and CEM-PC are polynomial-time approximations; G-PC does not provide an optimal solution.
- Biological analysis via overlap with existing gene marker lists may be incomplete; a more thorough analysis of selected genes relative to each cell type was not performed.
- Other coverage-optimizing approximation algorithms remain untested on biological data; alternative approximators might yield better coverage.

## Evidence

- [supplementary] CEM-PC determines its final marker set by selecting all features whose probability score exceeds 0.98 after the cross-entropy optimization converges (per Methods S1, algorithm 3).: [evidence span withheld — anchor status: not_found]
- [readme] # Pairs that could not be covered to the desired `coverage`
>>> gpc.pairs_with_incomplete_cover_: "# Pairs that could not be covered to the desired `coverage`
    >>> gpc.pairs_with_incomplete_cover_"
