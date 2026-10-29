---
name: cross-entropy-method-optimization
description: Use when you need a small global biomarker set that distinguishes between
  all pairs of phenotypes (e.g., cell type × tissue combinations in scRNA-seq) from
  a phenotype-feature score matrix, and a greedy approximation (G-PC) is insufficient.
when_to_use_negative:
- You only need per-phenotype differentially-expressed markers rather than a global
  set discriminating every phenotype pair
- You require an optimal solution — G-PC and CEM-PC are polynomial-time approximations
  and do not guarantee optimality
- You are doing minimal-optimal (mRMR-style) feature selection for a single ML target
  instead of covering all phenotype pairs
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CEMPC
  role: Cross-entropy-method solver for the phenotype cover marker-selection problem,
    imported from phenotype_cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Python package providing CEMPC/GreedyPC implementations; installed via pip
    alongside multiset-multicover
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion pip package for the multiset multicover formulation underlying phenotype
    cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Numerical dependency of phenotype-cover used for the optimization loop over
    the gene-set distribution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:89d91984b8421267b2b579be804e751ee4cfe74382bbb90915870b5d4d01aedb
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
  - Python
  - multiset-multicover
  - numpy
  - phenotype-cover
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-entropy-method-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Application of the cross-entropy-method variant of phenotype cover (CEM-PC) to solve the multiset multicover marker-selection problem, selecting a global gene set that discriminates every phenotype pair from a phenotype × gene score matrix, with final markers extracted where probability scores exceed 0.98.

## When to use

Use when you need a small global biomarker set that distinguishes between all pairs of phenotypes (e.g., cell type × tissue combinations in scRNA-seq) from a phenotype-feature score matrix, and a greedy approximation (G-PC) is insufficient.

## When NOT to use

- You only need per-phenotype differentially-expressed markers rather than a global set discriminating every phenotype pair
- You require an optimal solution — G-PC and CEM-PC are polynomial-time approximations and do not guarantee optimality
- You are doing minimal-optimal (mRMR-style) feature selection for a single ML target instead of covering all phenotype pairs

## Inputs

- Phenotype × gene score matrix (e.g., IPF phenotype x gene scores from scRNA-seq)
- Phenotype-pair coverage requirements for the phenotype cover problem

## Outputs

- Selected gene/marker set (features with probability score > 0.98)
- Attained phenotype-pair coverage value (e.g., cem_pc_ipf_selected_markers.csv)

## How to apply

Load a phenotype × gene score matrix (e.g., IPF scRNA-seq differentially-expressed scores) and construct phenotype-pair coverage requirements for the phenotype cover problem, which is equivalent to the NP-complete multiset multicover problem. Initialize the CEMPC solver from the phenotype-cover Python package (pip install phenotype-cover; depends on numpy, matplotlib, scikit-learn), with a scoring function that rewards phenotype-pair coverage while penalizing the size of the selected gene set. Run the cross-entropy optimization loop (Methods S1, algorithm 3), configured with I = 500 iterations/samples and Rs = 1000 as the elite-sample parameter. After convergence, extract the final marker set by selecting all features whose probability score exceeds 0.98, then compute the attained phenotype-pair coverage of the selected set and record it alongside the genes (e.g., in cem_pc_ipf_selected_markers.csv).

## Related tools

- **CEMPC** (Cross-entropy-method solver for the phenotype cover marker-selection problem, imported from phenotype_cover) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Python package providing CEMPC/GreedyPC implementations; installed via pip alongside multiset-multicover) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion pip package for the multiset multicover formulation underlying phenotype cover) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Numerical dependency of phenotype-cover used for the optimization loop over the gene-set distribution)

## Examples

```
from phenotype_cover import CEMPC; cempc = CEMPC(); cempc.fit(X, y); features = cempc.select(coverage)
```

## Evaluation signals

- Output file (e.g., cem_pc_ipf_selected_markers.csv) exists and is parseable as CSV with selected genes plus the attained coverage value recorded
- All reported markers have cross-entropy probability scores above the 0.98 threshold at convergence
- Reported coverage reflects the coverage-rewarding, size-penalizing scoring function used during optimization
- Coverage configuration matches the run parameters (I = 500 iterations, Rs = 1000 elite samples)
- Solution coverage can be compared against G-PC's attained coverage on the same phenotype-pair requirements

## Limitations

- CEM-PC, like G-PC, is a polynomial-time approximation and does not provide an optimal solution to the NP-complete multiset multicover problem
- The greedy (G-PC) implementation was reported to outperform competitors, so CEM-PC may not always yield the best set for a given task
- Raw count matrices may need log-transformation before fitting (per README guidance)

## Evidence

- [supplementary] CEM-PC determines its final marker set by selecting all features whose probability score exceeds 0.98 after the cross-entropy optimization converges (per Methods S1, algorithm 3).: [evidence span withheld — anchor status: not_found]
- [intro] The first is based on the extended greedy algorithm to set cover (G-PC),33, and the second is based on the cross-entropy method (CEM-PC): [evidence span withheld — anchor status: not_found]
- [readme] This repository provides two algorithms for the phenotype cover (PC)
biomarker selection problem introduced in the paper: "This repository provides two algorithms for the phenotype cover (PC)
biomarker selection problem introduced in the paper"
