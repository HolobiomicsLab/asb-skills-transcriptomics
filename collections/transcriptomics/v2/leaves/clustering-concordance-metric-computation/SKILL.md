---
name: clustering-concordance-metric-computation
description: Use when you need to quantitatively benchmark whether a feature-quantification
  method (e.g., MATES vs scTE/SoloTE) yields better downstream cell clustering, given
  known cell labels and repeated clusterings over multiple seeds.
when_to_use_negative:
- Input is a single clustering result with no ground-truth labels — ARI/NMI require
  reference labels and cannot be computed
- You need TE locus quantification itself (read assignment/counting), not the downstream
  evaluation of clusterings built from those counts
- Only one random seed or one clustering run is available — the skill's t-test over
  N=10 seeds requires distributional replicates
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Source of TE-quantified feature matrices (locus-level counts via multi-mapping
    read allocation with 2001 bp coverage vectors) whose clusterings are benchmarked
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method whose feature matrices are benchmarked
    against MATES in TE-only and multi-mapping-TE-only perspectives
- name: SoloTE
  role: Baseline single-cell TE quantification method compared with MATES at both
    subfamily and locus levels; MATES showed ~10.52% higher ARI at the locus level
- name: Scanpy
  role: Used for Louvain/Leiden cell clustering of each feature matrix before computing
    ARI/NMI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:0b27fd370d2cd27d0a95d2393d7674a3c0f969fe054ab2fff44ae1b10700f134
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# clustering-concordance-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for comparing cell-clustering quality across alternative feature matrices (e.g., Gene/Peak+TE, TE-only, Multi-mapping-TE-only produced by MATES, scTE, and SoloTE) using Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) against known cell labels, with statistical comparison via one-sided Student's t-tests over multiple random seeds.

## When to use

Use when you need to quantitatively benchmark whether a feature-quantification method (e.g., MATES vs scTE/SoloTE) yields better downstream cell clustering, given known cell labels and repeated clusterings over multiple seeds.

## When NOT to use

- Input is a single clustering result with no ground-truth labels — ARI/NMI require reference labels and cannot be computed
- You need TE locus quantification itself (read assignment/counting), not the downstream evaluation of clusterings built from those counts
- Only one random seed or one clustering run is available — the skill's t-test over N=10 seeds requires distributional replicates

## Inputs

- Feature count matrices per method and perspective (Gene/Peak+TE, TE-only, Multi-mapping-TE-only) in a format loadable by Scanpy (e.g., AnnData/h5ad)
- Ground-truth cell-type labels for each benchmark dataset (e.g., 10x mouse chemical reprogramming scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq)
- Number of random seeds (N=10) for repeated clustering runs

## Outputs

- Per-dataset, per-perspective ARI and NMI distributions (one value per random seed) for each method
- Ranked method comparison across the three perspectives (e.g., MATES first, as in Fig. 6)
- One-sided Student's t-test statistics and P-values for pairwise method comparisons (e.g., P ≈ 2.60e-12 for MATES vs SoloTE locus-level ARI)

## How to apply

For each benchmark dataset (e.g., 10x mouse chemical reprogramming scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq), build feature matrices under three perspectives — Gene/Peak+TE (combined gene/peak plus TE counts), TE-only, and Multi-mapping-TE-only — for each quantification method. Cluster cells from each matrix using community detection (e.g., Louvain/Leiden in Scanpy), repeat over N=10 random seeds to control initialization variability, and compute ARI and NMI against known cell labels. Compare ARI/NMI distributions between methods (e.g., MATES vs SoloTE, MATES vs scTE) with one-sided Student's t-tests. The comparison is accepted when the method under test ranks first across perspectives and reproduces effect sizes such as MATES achieving ~10.52% higher locus-level ARI than SoloTE with one-sided t-test P ≈ 2.60e-12, as in Fig. 6 of the MATES paper.

## Related tools

- **MATES** (Source of TE-quantified feature matrices (locus-level counts via multi-mapping read allocation with 2001 bp coverage vectors) whose clusterings are benchmarked) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method whose feature matrices are benchmarked against MATES in TE-only and multi-mapping-TE-only perspectives)
- **SoloTE** (Baseline single-cell TE quantification method compared with MATES at both subfamily and locus levels; MATES showed ~10.52% higher ARI at the locus level)
- **Scanpy** (Used for Louvain/Leiden cell clustering of each feature matrix before computing ARI/NMI)

## Examples

```
mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()  # then cluster with Scanpy and compute ARI/NMI over 10 seeds
```

## Evaluation signals

- ARI and NMI are computed against known cell labels and averaged over N=10 random seeds, so per-seed values should be stable within each method/perspective combination
- MATES should rank first across the Gene/Peak+TE, TE-only, and Multi-mapping-TE-only perspectives, reproducing the ordering shown in Fig. 6a, b
- The locus-level MATES-vs-SoloTE comparison should show MATES with ~10.52% higher ARI and a one-sided Student's t-test P ≈ 2.60e-12
- MATES should consistently outperform scTE and SoloTE specifically in the TE-only and multi-mapping-TE-only perspectives (middle and right panels of Fig. 6a, b)
- All three perspectives (Gene/Peak+TE, TE-only, Multi-mapping-TE-only) must be evaluated for each method — omitting a perspective breaks the ranking validation

## Limitations

- ARI/NMI require pre-existing ground-truth cell labels; the benchmark cannot be applied to unlabeled datasets
- Results depend on seed count (N=10) and clustering algorithm choice (Louvain/Leiden in Scanpy); changing these alters the t-test distributions
- Feature-matrix construction choices upstream (e.g., trimming TE-gene overlaps) can unintentionally distort TE quantification and hence clustering scores, with no quantitative guidance on when to opt out

## Evidence

- [discussion] Benchmark workflow and metrics: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline modules for building feature matrices: [evidence span withheld — anchor status: not_found]
