---
name: phenotype-cover-optimization
description: Use when marker selection must produce a single global feature set that
  discriminates between many overlapping phenotypes (e.g., cell-type/tissue combinations
  in IPF, mouse cortex, or HCA scRNA-seq data), not just per-group DE genes.
when_to_use_negative:
- The goal is per-group differentially expressed genes rather than a single global
  marker set discriminating between many overlapping phenotypes (e.g., a small number
  of distinct major cell types)
- An optimal (not approximate) solution to the covering problem is required — G-PC
  provides no optimality guarantee
- The input is already a curated marker list or a feature table unrelated to phenotype-pair
  discrimination
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: phenotype-cover
  role: Python package providing GreedyPC and CEMPC solvers for the phenotype cover
    marker-selection problem
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Companion package installed alongside phenotype-cover for the equivalent multiset
    multicover formulation
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Extended greedy set-cover approximation algorithm used to select the marker
    set
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy-method approximation algorithm, alternative solver to GreedyPC
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Dependency used for data generation in examples and logistic regression evaluation
    of selected features
- name: mrmr_selection
  role: Competitor minimum-redundancy maximum-relevance feature selection method used
    for runtime benchmarking
  repo: https://github.com/smazzanti/mrmr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:b7e2f408c50dd8a00ea0abde4389db1c1e562b9396e3c69e2203e17aac503987
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phenotype-cover-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formulates discriminative marker selection as the phenotype cover (PC) problem — finding the smallest feature set such that every phenotype pair is covered — and solves it via the G-PC (greedy) and CEM-PC (cross-entropy) polynomial-time approximations from Hasanaj et al. 2022.

## When to use

Use when marker selection must produce a single global feature set that discriminates between many overlapping phenotypes (e.g., cell-type/tissue combinations in IPF, mouse cortex, or HCA scRNA-seq data), not just per-group DE genes.

## When NOT to use

- The goal is per-group differentially expressed genes rather than a single global marker set discriminating between many overlapping phenotypes (e.g., a small number of distinct major cell types)
- An optimal (not approximate) solution to the covering problem is required — G-PC provides no optimality guarantee
- The input is already a curated marker list or a feature table unrelated to phenotype-pair discrimination

## Inputs

- Phenotype x feature score matrix M (e.g., derived from preprocessed scRNA-seq datasets such as IPF (Adams et al.), mouse cortex (Zeisel et al.), or HCA (He et al.))
- Phenotype/label vector y for each sample (as in make_classification-style inputs)
- Coverage parameter (integer threshold a marker must exceed for each phenotype pair)
- Target number of features K to select

## Outputs

- A global marker set S (ranked feature list) covering all phenotype pairs at the chosen coverage
- Coverage-per-iteration diagnostics (gpc.coverage_per_iter_, gpc.n_elements_remaining_per_iter_)
- List of phenotype pairs with incomplete cover (gpc.pairs_with_incomplete_cover_)
- Wall-clock runtime measurements for benchmarking against competitor methods

## How to apply

Build a phenotype x feature score matrix M (e.g., from preprocessed scRNA-seq data such as the IPF lung, Zeisel mouse cortex, or He HCA datasets); entries quantify how well each feature separates a phenotype pair. Formulate PC as combinatorial optimization equivalent to multiset multicover (NP-complete) and run either GreedyPC, the extended greedy set-cover approximation, or CEMPC, the cross-entropy method, both from the phenotype-cover package. Choose a coverage parameter (e.g., coverage of 8 in the GSEA analysis) and call select() to obtain the marker set at the desired number of features (178 for IPF, 66 for MC, 121 for HCA in the runtime benchmark). For speed-critical runs, use the C++ G-PC implementation with Python bindings, which was the fastest method tested, running under 2 seconds on all three datasets. Evaluate the resulting set by training logistic regression on a subset and testing on left-out data, and by feature-stability analysis over 5 resampled runs.

## Related tools

- **phenotype-cover** (Python package providing GreedyPC and CEMPC solvers for the phenotype cover marker-selection problem) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Companion package installed alongside phenotype-cover for the equivalent multiset multicover formulation) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Extended greedy set-cover approximation algorithm used to select the marker set) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy-method approximation algorithm, alternative solver to GreedyPC) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Dependency used for data generation in examples and logistic regression evaluation of selected features)
- **mrmr_selection** (Competitor minimum-redundancy maximum-relevance feature selection method used for runtime benchmarking) — https://github.com/smazzanti/mrmr

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(8)
```

## Evaluation signals

- G-PC runtimes fall under 2 seconds on the IPF, MC, and HCA datasets when using the C++ implementation with Python bindings, consistent with Table 2's claim it is the fastest method tested
- Selected marker set S achieves the requested coverage for all phenotype pairs; check pairs_with_incomplete_cover_ is empty (or as expected given max_coverage)
- Logistic regression trained on the selected feature subset performs well on left-out data across the phenotype classes
- Feature selection is stable: the stability index computed over 5 runs of randomly sampling half the data shows consistent selected features
- coverage_per_iter_ monotonically increases and n_elements_remaining_per_iter_ decreases across greedy iterations

## Limitations

- G-PC does not provide an optimal solution to the phenotype cover problem
- Coverage to the desired threshold may be unattainable for some phenotype pairs (pairs_with_incomplete_cover_ may be non-empty)
- Input matrices may need log-transformation when working with raw counts
- Other coverage-optimizing approximation algorithms remain untested on biological data

## Evidence
