---
name: one-sided-t-test-method-comparison
description: Use when two or more quantification/clustering methods have been run
  on the same datasets and per-seed metric distributions (e.g., ARI/NMI over 10 random
  seeds) are available, and the goal is to test whether the proposed method is significantly
  better, not just different.
when_to_use_negative:
- You have only a single metric value per method (no seed-level distribution) — a
  t-test on singleton observations is undefined/meaningless
- The comparison hypothesis is non-directional (any difference matters) — use a two-sided
  test instead of a one-sided one
- Clustering metrics are not computed against the same ground-truth labels for both
  methods, making the paired distributions incomparable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MATES
  role: Candidate method whose TE quantification feeds cell clustering; its ARI/NMI
    distributions are the test numerator
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method compared against MATES via one-sided
    t-test
- name: SoloTE
  role: Baseline single-cell TE quantification method compared against MATES via one-sided
    t-test, including at locus level on shared datasets
- name: Scanpy
  role: Louvain/Leiden community detection used to generate the per-seed clustering
    assignments that produce ARI/NMI values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:fc9a6e863597572b1530d712f55c8d06ed48fbe49bb94765e5087594d304c039
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - Scanpy
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# one-sided-t-test-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical validation pattern for benchmarking a new method (here, MATES) against existing tools (scTE, SoloTE) by comparing performance-metric distributions (ARI/NMI over N=10 random seeds) with one-sided Student's t-tests, establishing directional superiority rather than mere difference.

## When to use

Use when two or more quantification/clustering methods have been run on the same datasets and per-seed metric distributions (e.g., ARI/NMI over 10 random seeds) are available, and the goal is to test whether the proposed method is significantly better, not just different.

## When NOT to use

- You have only a single metric value per method (no seed-level distribution) — a t-test on singleton observations is undefined/meaningless
- The comparison hypothesis is non-directional (any difference matters) — use a two-sided test instead of a one-sided one
- Clustering metrics are not computed against the same ground-truth labels for both methods, making the paired distributions incomparable

## Inputs

- Per-seed ARI/NMI metric vectors (N=10 random seeds) for each method and each feature perspective (Gene/Peak+TE, TE-only, multi-mapping-TE-only)
- Ground-truth cell labels for the benchmark datasets (10x mouse scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq)

## Outputs

- One-sided Student's t-test P-values for MATES vs scTE and MATES vs SoloTE on ARI and NMI distributions
- Per-perspective ranking of methods by mean ARI/NMI (e.g., MATES ranked first as in Fig. 6a, b)
- Reported effect size such as percentage ARI improvement (e.g., ~10.52% ARI gain, P ≈ 2.60e-12)

## How to apply

For each dataset (e.g., 10x mouse chemical reprogramming scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq), cluster cells from each feature matrix (Louvain/Leiden in Scanpy) and compute Adjusted Rand Index and Normalized Mutual Information against known cell labels, repeating over N=10 random seeds to obtain metric distributions per method. Then compare MATES vs scTE and MATES vs SoloTE ARI/NMI distributions with one-sided Student's t-tests, using the one-sided (greater) alternative because the hypothesis is directional: the new method outperforms the baseline. Repeat across the three feature perspectives — Gene/Peak+TE, TE-only, and multi-mapping-TE-only — and additionally perform the locus-level MATES-vs-SoloTE comparison on shared datasets. Acceptance criterion from this paper: the locus-level result should show MATES achieving ~10.52% higher ARI than SoloTE with one-sided t-test P ≈ 2.60e-12, and MATES ranking first across the three perspectives as in Fig. 6.

## Related tools

- **MATES** (Candidate method whose TE quantification feeds cell clustering; its ARI/NMI distributions are the test numerator) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method compared against MATES via one-sided t-test)
- **SoloTE** (Baseline single-cell TE quantification method compared against MATES via one-sided t-test, including at locus level on shared datasets)
- **Scanpy** (Louvain/Leiden community detection used to generate the per-seed clustering assignments that produce ARI/NMI values)

## Examples

```
from scipy import stats; res = stats.ttest_ind(mates_ari_seeds, solote_ari_seeds, alternative='greater')
```

## Evaluation signals

- Exactly N=10 ARI/NMI values per method per perspective are collected before testing; distributions (not single points) are compared
- Locus-level MATES-vs-SoloTE result reproduces ~10.52% higher ARI with one-sided t-test P ≈ 2.60e-12
- MATES ranks first across all three perspectives (Gene/Peak+TE, TE-only, multi-mapping-TE-only), matching Fig. 6a, b middle and right panels
- One-sided (greater-alternative) tests are used for the directional claim, and P-values are reported per method pair and perspective

## Limitations

- One-sided tests assume the direction of superiority a priori; they cannot detect the reverse effect and inflate significance if direction is chosen after seeing results
- Comparison requires a common evaluation protocol (same clustering algorithm, same ground-truth labels, same seeds) across methods; differing pipelines invalidate the test
- Metric distributions over seeds may not be independent or normal, which Student's t-test assumes

## Evidence

- [other] Benchmark finding that MATES outperformed baselines in TE-only and multi-mapping perspectives: [evidence span withheld — anchor status: not_found]
- [other] Statistical test specification and seed repetition: [evidence span withheld — anchor status: not_found]
- [other] Quantitative acceptance criterion for the reproduced comparison: [evidence span withheld — anchor status: not_found]
