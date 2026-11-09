---
name: cross-entropy-method-feature-selection
description: Use when you have a phenotype x feature score matrix M from scRNA-seq
  data (e.g., IPF, mouse cortex, or human cell atlas datasets) and need a discriminative
  marker set covering all phenotype pairs at a fixed coverage, especially when comparing
  against or substituting for the greedy G-PC solution.
when_to_use_negative:
- When only a small number of distinct groups with non-overlapping DE genes exist
  — standard per-group differential expression marker selection suffices
- When an exact optimal marker set is required — G-PC and CEM-PC are polynomial-time
  approximations to the NP-complete multiset multicover problem and do not provide
  an optimal solution
- When raw untransformed counts are passed directly — the README notes X may need
  log-transformation before fitting
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CEMPC
  role: The cross-entropy-method solver for the phenotype cover problem; the core
    algorithm of this skill, imported from phenotype_cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Python package providing CEMPC and GreedyPC implementations for marker selection
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion package implementing the multiset multicover formulation underlying
    PC; installed alongside phenotype-cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: The greedy alternative (extended greedy set cover) used as the baseline solver
    that CEM-PC is compared against
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Dependency of phenotype-cover used for matrix handling of the phenotype x
    feature score matrix
- name: scikit-learn
  role: Dependency of phenotype-cover; also supplies competitor methods (decision
    trees, F-value, ReliefF, mutual information) for runtime benchmarking
- name: mrmr_selection
  role: mRMR feature selection package (smazzanti/mrmr) used as a runtime competitor
    in Table 2
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:185de3dbc0ce690ebcd50ea5213d1de1473f9c1601bf46bc7219a01f19929c48
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
  - mrmr_selection
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cross-entropy-method-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies CEM-PC, the cross-entropy-method approximation to the phenotype cover (PC) problem, to select a global marker set from a phenotype x feature score matrix so that every phenotype pair is covered by discriminative features. It is the sampling-based alternative to the greedy G-PC solver shipped in the phenotype-cover package.

## When to use

Use when you have a phenotype x feature score matrix M from scRNA-seq data (e.g., IPF, mouse cortex, or human cell atlas datasets) and need a discriminative marker set covering all phenotype pairs at a fixed coverage, especially when comparing against or substituting for the greedy G-PC solution.

## When NOT to use

- When only a small number of distinct groups with non-overlapping DE genes exist — standard per-group differential expression marker selection suffices
- When an exact optimal marker set is required — G-PC and CEM-PC are polynomial-time approximations to the NP-complete multiset multicover problem and do not provide an optimal solution
- When raw untransformed counts are passed directly — the README notes X may need log-transformation before fitting

## Inputs

- Phenotype x feature score matrix M (e.g., derived from scRNA-seq count matrices of the IPF, MC, or HCA datasets; log-transform raw counts)
- Phenotype/cell-type labels for each row of M
- Target coverage value or desired number of selected features (e.g., 178 for IPF, 66 for MC, 121 for HCA)

## Outputs

- Selected marker set S (list of feature indices/gene identifiers covering phenotype pairs at the chosen coverage)
- Coverage diagnostics (coverage per iteration, pairs with incomplete cover)
- Runtime measurements for benchmarking against G-PC and competing methods (Table 2)

## How to apply

Preprocess the scRNA-seq data (log-transform raw counts if needed) into a phenotype x feature score matrix M, where entry Mi,s reflects how discriminative feature s is for phenotype i. Install via 'pip install multiset-multicover' and 'pip install phenotype-cover', then import CEMPC from phenotype_cover and fit it on the score matrix with phenotype labels. Choose the coverage target (the paper uses values such as a coverage of 8 for GSEA marker extraction and per-dataset feature counts of 178 for IPF, 66 for MC, and 121 for HCA when matching feature budgets across methods). Run CEM-PC to obtain the selected feature set S, and benchmark its runtime against G-PC (C++ implementation with Python bindings) and competitors such as CEM-PC vs decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information, and mRMR. CEM-PC is one of the two polynomial-time approximations to the NP-complete multiset multicover formulation, so it trades optimality guarantees for tractability; validate coverage using attributes like pairs_with_incomplete_cover_ and max_coverage().

## Related tools

- **CEMPC** (The cross-entropy-method solver for the phenotype cover problem; the core algorithm of this skill, imported from phenotype_cover) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Python package providing CEMPC and GreedyPC implementations for marker selection) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion package implementing the multiset multicover formulation underlying PC; installed alongside phenotype-cover) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (The greedy alternative (extended greedy set cover) used as the baseline solver that CEM-PC is compared against) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Dependency of phenotype-cover used for matrix handling of the phenotype x feature score matrix)
- **scikit-learn** (Dependency of phenotype-cover; also supplies competitor methods (decision trees, F-value, ReliefF, mutual information) for runtime benchmarking)
- **mrmr_selection** (mRMR feature selection package (smazzanti/mrmr) used as a runtime competitor in Table 2) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import CEMPC; cempc = CEMPC(); cempc.fit(X, y); features = cempc.select(8)
```

## Evaluation signals

- Selected set S covers every phenotype pair to the requested coverage; GreedyPC-style diagnostics such as pairs_with_incomplete_cover_ should be empty (or the incomplete pairs documented)
- Runtime of CEM-PC on IPF/MC/HCA with matched feature counts (178/66/121) is recorded and compared against G-PC (C++), which runs under 2 seconds on all three datasets
- Feature count of the output matches the pre-specified budget per dataset (178 IPF, 66 MC, 121 HCA) when mirroring Table 2
- Downstream validation: train a logistic regression on a subset with the selected features and evaluate on left-out data, and/or compute a stability index over 5 runs sampling half the data
- Selected genes show enrichment against the HuBMAP ASCT+B marker set when analyzed at coverage of 8

## Limitations

- CEM-PC is a polynomial-time approximation; neither G-PC nor CEM-PC provides an optimal solution to the NP-complete multiset multicover problem
- Raw counts may need log-transformation before fitting, or results will be distorted
- Biological analysis of selected genes relied on overlap with existing marker lists that may be incomplete, limiting interpretation of marker quality

## Evidence

- [intro] CEM-PC is one of two polynomial-time approximations to PC: "We show that PC is equivalent to multiset multicover, which is nondeterministic polynomial-time (NP)-complete32 and propose two algorithms that can approximate it in polynomial time"
- [intro] CEM-PC defined as cross-entropy method: "The first is based on the extended greedy algorithm to set cover (G-PC),33 and the second is based on the cross-entropy method (CEM-PC)"
- [readme] Installation and import instructions: "pip install multiset-multicover
    pip install phenotype-cover"
