---
name: runtime-benchmarking-of-feature-selection-methods
description: Use when you need to compare wall-clock runtime of feature/marker-selection
  algorithms on the same datasets at an identical number of selected features, e.g.
  reproducing or extending the paper's Table 2 speed comparison.
when_to_use_negative:
- When the goal is marker quality (accuracy, stability, deconvolution performance)
  rather than speed — the paper's Table 1 and stability analyses cover those separately
- When datasets or feature counts differ across methods, since unmatched feature-count
  settings make the runtime comparison invalid
- When working with raw counts that have not been converted to the phenotype x feature
  score matrix required by the phenotype-cover formulation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: multiset-multicover
  role: C++ implementation of G-PC with Python bindings, the fastest method timed
    in the benchmark
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Python package providing GreedyPC and CEMPC, both benchmarked on the three
    datasets
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: 'source of competitor feature selectors: decision trees, F-value, ReliefF-style,
    and mutual information'
- name: smazzanti/mrmr
  role: mRMR competitor installed via pip install mrmr_selection and timed with mrmr_classif
    at matched feature counts
  repo: https://github.com/smazzanti/mrmr
- name: CEMPC
  role: cross-entropy-method phenotype-cover variant benchmarked against G-PC
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:f1f278ca1ed9c99e9138d6cdd5b78449cadc3e565546f750d5992d2b6dbcf1a2
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
  - smazzanti/mrmr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# runtime-benchmarking-of-feature-selection-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for reproducing the runtime comparison of marker-selection methods (Table 2 of the multiset multicover paper), timing the C++ G-PC implementation against CEM-PC, decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information, and mRMR on the IPF, MC, and HCA scRNA-seq datasets at matched feature counts.

## When to use

Use when you need to compare wall-clock runtime of feature/marker-selection algorithms on the same datasets at an identical number of selected features, e.g. reproducing or extending the paper's Table 2 speed comparison.

## When NOT to use

- When the goal is marker quality (accuracy, stability, deconvolution performance) rather than speed — the paper's Table 1 and stability analyses cover those separately
- When datasets or feature counts differ across methods, since unmatched feature-count settings make the runtime comparison invalid
- When working with raw counts that have not been converted to the phenotype x feature score matrix required by the phenotype-cover formulation

## Inputs

- IPF scRNA-seq dataset (Adams et al.) preprocessed into a phenotype x feature score matrix (178 target features)
- Mouse cortex (MC) scRNA-seq dataset (Zeisel et al.) phenotype x feature score matrix (66 target features)
- Human cell atlas (HCA) scRNA-seq dataset (He et al.) phenotype x feature score matrix (121 target features)
- Feature-selection methods: GreedyPC, CEMPC, decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information, mRMR

## Outputs

- runtime_table.csv with one row per method and one column per dataset (IPF, MC, HCA), mirroring Table 2
- Wall-clock runtime measurements per method per dataset
- Validation check that C++ G-PC runtimes are < 2 seconds on all three datasets

## How to apply

Preprocess the IPF, MC, and HCA scRNA-seq datasets into phenotype x feature score matrices M as required by the phenotype-cover formulation. Run the C++ GreedyPC implementation (multiset-multicover/phenotype-cover with Python bindings) selecting 178 features for IPF, 66 for MC, and 121 for HCA, recording wall-clock time per run; run CEMPC with the same feature counts. Run the competitors — decision trees, scGeneFit, RankCorr, F-value, and mutual information via scikit-learn, plus mrmr_classif from mrmr_selection — on each dataset with identical feature-count settings and record runtimes. Assemble one runtime_table.csv row per method and one column per dataset, mirroring Table 2, and check that G-PC finishes in under 2 seconds on all three datasets, consistent with the claim that the C++ implementation is the fastest method tested. Runtime comparisons are only fair when the number of selected features matches across methods, which is why fixed feature counts are imposed.

## Related tools

- **multiset-multicover** (C++ implementation of G-PC with Python bindings, the fastest method timed in the benchmark) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Python package providing GreedyPC and CEMPC, both benchmarked on the three datasets) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (source of competitor feature selectors: decision trees, F-value, ReliefF-style, and mutual information)
- **smazzanti/mrmr** (mRMR competitor installed via pip install mrmr_selection and timed with mrmr_classif at matched feature counts) — https://github.com/smazzanti/mrmr
- **CEMPC** (cross-entropy-method phenotype-cover variant benchmarked against G-PC) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(178)
```

## Evaluation signals

- G-PC (C++ implementation) completes in under 2 seconds on IPF, MC, and HCA
- All methods are timed on identical feature-count settings: 178 (IPF), 66 (MC), 121 (HCA)
- runtime_table.csv contains one row per method and one column per dataset, matching Table 2 layout
- G-PC runtime is lower than every competitor (CEM-PC, decision trees, scGeneFit, RankCorr, F-value, ReliefF, mutual information, mRMR) on each dataset
- Runtimes are recorded as wall-clock times from the same machine/run conditions for all methods

## Limitations

- Runtimes are hardware- and implementation-dependent; Table 2 values reflect the authors' C++ implementation with Python bindings and may not transfer to other machines
- G-PC is an approximation: it works well on the analyzed data but does not provide an optimal solution
- Timing fairness depends on matching the number of selected features across methods; unmatched K values invalidate the comparison

## Evidence
