---
name: phenotype-pair-coverage-scoring
description: Use when you need to select a global biomarker set that discriminates
  every phenotype pair in a phenotype x feature score matrix, or when scoring a gene
  set by the phenotype-pair coverage it attains (e.g., CEM-PC with I=500 iterations,
  Rs=1000 elite samples).
when_to_use_negative:
- You need per-class one-vs-rest markers only, not a global set that discriminates
  every phenotype pair — ordinary DE analysis may suffice when there is a small number
  of distinct groups
- 'You need the provably optimal cover: PC is NP-complete via multiset multicover,
  and CEM-PC (like G-PC) provides an approximation, not an optimal solution'
- Input is raw counts without log transformation — the README notes X may need log-transforming
  when working with raw counts
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CEMPC
  role: Cross-entropy-method solver imported from phenotype_cover that optimizes gene-set
    distribution to maximize phenotype-pair coverage while penalizing set size
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Package providing GreedyPC/CEMPC for the phenotype cover biomarker selection
    problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Dependency of phenotype-cover used for the Python/numpy implementation of
    the solver over the score matrix
- name: GreedyPC
  role: Alternative extended-greedy approximation to the same PC problem, useful for
    comparison against the CEM-PC coverage score
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:32efb5d7a2e93cfac2cf126a85ce932a3f776a16bf7a68f87bab0e5ebf43608d
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# phenotype-pair-coverage-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score a candidate marker gene set by the phenotype-pair coverage it attains over a phenotype x gene score matrix (e.g., IPF scRNA-seq), as formulated in the phenotype cover (PC) / multiset multicover problem. This coverage-rewarding, size-penalizing score drives both the CEM-PC cross-entropy solver and final marker-set extraction (features with probability score > 0.98).

## When to use

Use when you need to select a global biomarker set that discriminates every phenotype pair in a phenotype x feature score matrix, or when scoring a gene set by the phenotype-pair coverage it attains (e.g., CEM-PC with I=500 iterations, Rs=1000 elite samples).

## When NOT to use

- You need per-class one-vs-rest markers only, not a global set that discriminates every phenotype pair — ordinary DE analysis may suffice when there is a small number of distinct groups
- You need the provably optimal cover: PC is NP-complete via multiset multicover, and CEM-PC (like G-PC) provides an approximation, not an optimal solution
- Input is raw counts without log transformation — the README notes X may need log-transforming when working with raw counts

## Inputs

- Phenotype x gene score matrix (e.g., IPF phenotype-by-gene scores; log-transform X if working with raw counts)
- Phenotype-pair coverage requirements for the phenotype cover problem
- CEM configuration parameters: I = 500 iterations/samples, Rs = 1000 elite-sample parameter

## Outputs

- Selected marker gene set (all features with CEM probability score > 0.98)
- Attained phenotype-pair coverage of the selected set
- CSV of selected genes plus attained coverage (e.g., cem_pc_ipf_selected_markers.csv)

## How to apply

Load the phenotype x gene score matrix M and construct the phenotype-pair coverage requirements of the phenotype cover problem. Initialize the CEMPC solver from the phenotype-cover package, setting the scoring function to reward phenotype-pair coverage while penalizing the size of the selected gene set. Configure the cross-entropy method with I = 500 iterations and Rs = 1000 as the elite-sample parameter, run the CEM optimization loop over the gene-set distribution, then extract the best-scoring gene set from the final CEM distribution. Per Methods S1 (algorithm 3), the final marker set is all features whose probability score exceeds 0.98 after convergence. Finally compute the attained phenotype-pair coverage of the selected set and record the genes together with that coverage value.

## Related tools

- **CEMPC** (Cross-entropy-method solver imported from phenotype_cover that optimizes gene-set distribution to maximize phenotype-pair coverage while penalizing set size) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Package providing GreedyPC/CEMPC for the phenotype cover biomarker selection problem) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Dependency of phenotype-cover used for the Python/numpy implementation of the solver over the score matrix)
- **GreedyPC** (Alternative extended-greedy approximation to the same PC problem, useful for comparison against the CEM-PC coverage score) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from phenotype_cover import CEMPC; cem = CEMPC(I=500, Rs=1000); cem.fit(X, y); features = [f for f in features if prob > 0.98]
```

## Evaluation signals

- Output file exists and parses as CSV with the selected genes and the attained coverage recorded
- Reported coverage reflects the coverage-rewarding scoring function used during optimization
- Every gene in the final marker set has a CEM probability score exceeding 0.98 (per Methods S1, algorithm 3)
- Coverage computed per the PC formulation: for phenotype i, a gene s contributes when Mi,s >= Mj,s + 1 for some phenotype j
- Compare attained coverage against gpc.max_coverage() (maximum possible coverage for every class pair) to assess completeness

## Limitations

- PC is equivalent to multiset multicover, which is NP-complete, so polynomial-time approximations (G-PC, CEM-PC) do not guarantee optimality
- Genes in the solution may be less significant than overall DE genes since they target phenotype-pair specificity (e.g., cell-type x tissue combinations)
- G-PC worked well for the analyzed data but does not provide an optimal solution; other coverage-optimizing approximation algorithms remain untested on biological data
- Coverage thresholds are user-set (e.g., coverage of 8 for HuBMAP ASCT+B enrichment analysis); some phenotype pairs may remain with incomplete cover (pairs_with_incomplete_cover_)

## Evidence

- [supplementary] CEM-PC determines its final marker set by selecting all features whose probability score exceeds 0.98 after the cross-entropy optimization converges (per Methods S1, algorithm 3).: [evidence span withheld — anchor status: not_found]
