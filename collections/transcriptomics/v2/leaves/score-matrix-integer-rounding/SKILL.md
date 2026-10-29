---
name: score-matrix-integer-rounding
description: Use when you have a phenotype x feature score matrix M (e.g., from a
  scRNA-seq dataset like Tabula Muris, mouse cortex, or HCA) and need every phenotype
  pair (i,j) distinguished at least K times by selected markers.
when_to_use_negative:
- The feature-selection goal is all-relevant (identifying every feature related to
  the target) rather than a minimal set that discriminates every phenotype pair —
  mRMR-style or all-relevance methods fit better
- Raw count data has not been log-transformed and normalized first, which the README
  notes is needed when working with raw counts
- There is only a small number of distinct groups with non-overlapping differential
  genes, where ordinary per-group DE marker selection suffices
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GreedyPC
  role: Consumes the integer-rounded matrix and runs the extended greedy selection
    with O(kP^2F) complexity to cover all phenotype pairs
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Package providing the multiset multicover machinery into which the rounded
    matrix is cast
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Loading the scRNA-seq data and rounding/integer-casting the entries of matrix
    M
- name: scikit-learn
  role: Training the logistic regression used to evaluate F1 of the selected marker
    set on left-out data
- name: phenotype-cover
  role: Package from which GreedyPC/CEMPC are imported; depends on numpy, matplotlib,
    and scikit-learn
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:bb3bffba9e8f1e929c9a91a7a6637c7f17d3881dd25c1a74e16f25021adc87ab
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
  - GreedyPC
  - Python
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# score-matrix-integer-rounding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduction step in the phenotype cover (PC) marker-selection framework in which a real-valued phenotype x feature score matrix M is rounded to integer multiplicities so the PC problem becomes multiset multicover and greedy selection (G-PC) can be applied.

## When to use

Use when you have a phenotype x feature score matrix M (e.g., from a scRNA-seq dataset like Tabula Muris, mouse cortex, or HCA) and need every phenotype pair (i,j) distinguished at least K times by selected markers.

## When NOT to use

- The feature-selection goal is all-relevant (identifying every feature related to the target) rather than a minimal set that discriminates every phenotype pair — mRMR-style or all-relevance methods fit better
- Raw count data has not been log-transformed and normalized first, which the README notes is needed when working with raw counts
- There is only a small number of distinct groups with non-overlapping differential genes, where ordinary per-group DE marker selection suffices

## Inputs

- phenotype x feature score matrix M (real-valued, from scRNA-seq data such as lung, mouse cortex, or human cell atlas datasets)
- phenotype labels per cell/class
- target coverage K (integer, e.g., coverage of 8 in the GSEA analysis or 100 in the README example)

## Outputs

- integer-rounded multiplicity matrix suitable for multiset multicover
- selected marker set S with achieved phenotype-pair coverage
- F1 score of a logistic regression trained on the selected markers and evaluated on left-out data

## How to apply

Compute the phenotype x feature score matrix M from the dataset (e.g., a mouse cortex scRNA-seq matrix), then round each entry of M to an integer multiplicity per the multiset multicover formulation. Under this formulation a feature s covers phenotype pair (i,j) when Mi,s >= Mj,s + 1, which encodes the requirement that phenotype i's markers distinguish it from phenotype j. Because PC is equivalent to multiset multicover (NP-complete), feed the integer matrix into the polynomial-time approximation GreedyPC, which iteratively picks the feature with the largest corrected multiplicity-weighted coverage gain until all pairs reach the target coverage K. Validate the selected marker set S by checking achieved pair coverage against the requested K and by training a logistic regression on a subset of cells and measuring F1 on held-out data.

## Related tools

- **GreedyPC** (Consumes the integer-rounded matrix and runs the extended greedy selection with O(kP^2F) complexity to cover all phenotype pairs) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Package providing the multiset multicover machinery into which the rounded matrix is cast) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Loading the scRNA-seq data and rounding/integer-casting the entries of matrix M)
- **scikit-learn** (Training the logistic regression used to evaluate F1 of the selected marker set on left-out data)
- **phenotype-cover** (Package from which GreedyPC/CEMPC are imported; depends on numpy, matplotlib, and scikit-learn) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from phenotype_cover import GreedyPC; import numpy as np; M_int = np.rint(M).astype(int); gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100)
```

## Evaluation signals

- The selected marker set S attains the paper's reported phenotype-pair coverage on the chosen dataset (e.g., coverage of 8 for the HuBMAP ASCT+B GSEA comparison)
- Coverage of pair (i,j) is verifiable per feature via the inequality Mi,s >= Mj,s + 1 on the integer matrix
- F1 of the logistic regression trained on S matches reported performance on held-out data within a small tolerance
- GreedyPC's coverage_per_iter_ and n_elements_remaining_per_iter_ show monotone progress toward full pair coverage
- gpc.pairs_with_incomplete_cover_ is empty when the target coverage is attainable (max_coverage() upper-bounds achievable coverage)

## Limitations

- G-PC does not provide an optimal solution; it is a polynomial-time approximation to an NP-complete problem
- Raw counts must be log-transformed before fitting, per the README
- Pairs that cannot be covered to the desired coverage remain flagged in pairs_with_incomplete_cover_; max_coverage() bounds what is achievable

## Evidence
