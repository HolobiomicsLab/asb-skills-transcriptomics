---
name: runtime-performance-benchmarking
description: Use when comparing the total computational runtime of two sketching/subsampling
  methods on a large scRNA-seq expression matrix (e.g., Sphetcher-H vs geometric sketching
  on the 2,026,641-cell MOCA dataset), especially when per-stage timing breakdowns
  are needed.
when_to_use_negative:
- The goal is to assess sketch quality (accuracy of transcriptomic-landscape representation,
  rare cell-type detection) rather than computational runtime
- The dataset is small (e.g., the 465,281-cell Zeisel example or smaller) where runtime
  differences between methods are negligible and not the paper's scalability claim
- Only one method is being timed — the skill centers on a head-to-head comparison
  against the geometric sketching baseline on the same input
tools:
- name: Sphetcher
  role: Method under benchmark; run as the hybrid Sphetcher-H strategy (grid sampling
    + spherical thresholding/set cover) with stage-wise timing
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher source repository before compilation and timing
    runs
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:b2a356736df2ec836c44f09f2bfe714321f7401057f6bdd897074ca737a3e5e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# runtime-performance-benchmarking

## Summary

Benchmark the wall-clock runtime of Sphetcher's hybrid strategy (Sphetcher-H: grid sampling followed by spherical thresholding/set cover) against geometric box-based sketching on a two-million-cell single-cell matrix, timing each pipeline stage separately to validate the paper's scalability claim (~16 min vs ~30 min).

## When to use

Use when comparing the total computational runtime of two sketching/subsampling methods on a large scRNA-seq expression matrix (e.g., Sphetcher-H vs geometric sketching on the 2,026,641-cell MOCA dataset), especially when per-stage timing breakdowns are needed.

## When NOT to use

- The goal is to assess sketch quality (accuracy of transcriptomic-landscape representation, rare cell-type detection) rather than computational runtime
- The dataset is small (e.g., the 465,281-cell Zeisel example or smaller) where runtime differences between methods are negligible and not the paper's scalability claim
- Only one method is being timed — the skill centers on a head-to-head comparison against the geometric sketching baseline on the same input

## Inputs

- MOCA (Cao et al. 2019) expression matrix as CSV, rows = 2,026,641 cells, columns = features/genes/PCs
- Compiled Sphetcher binary (built via make in src, C++11 compiler required)
- Sketch size parameter (20296, i.e. 1% of cells)
- Geometric box-based sketching baseline on the identical matrix and sketch size

## Outputs

- sketch_indicator_output.csv — indicator vector of n samples with sketch_size ones
- Per-stage elapsed timings for Sphetcher-H (grid sampling, distance computation, set cover)
- Total runtime for geometric sketching baseline
- runtime_comparison.csv aggregating per-stage and total timings of both methods

## How to apply

Clone github.com/canzarlab/Sphetcher and build with 'make' in src using a compiler supporting C++11. Prepare the expression matrix as CSV with rows as cells and columns as features/PCs — here the Mouse organogenesis cell atlas (MOCA, Cao et al. 2019; 2,026,641 cells). Run Sphetcher-H at sketch size 20296 (1% of cells) and record elapsed time for the three stages separately: grid sampling, distance computation, and the spherical-thresholding set-cover step. Run the geometric box-based sketching baseline on the same matrix and sketch size, recording its total runtime. Aggregate per-stage and total timings into runtime_comparison.csv and compare against the reported values: Sphetcher-H ~16 minutes total (grid sampling ~10 s, distances ~600 s, set cover ~400 s) versus ~30 minutes for geometric sketching.

## Related tools

- **Sphetcher** (Method under benchmark; run as the hybrid Sphetcher-H strategy (grid sampling + spherical thresholding/set cover) with stage-wise timing) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher source repository before compilation and timing runs) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make; ./sphetcher moca_expression_matrix.csv 20296 sketch_indicator_output.csv
```

## Evaluation signals

- Sphetcher-H total runtime should be on the order of the paper's reported ~16 minutes for the two-million-cell dataset
- runtime_comparison.csv must contain timing entries for both methods and all three timed stages (grid sampling, distance computation, set cover)
- Sphetcher-H stage timings should be of the order: grid sampling ~10 s, distance computation ~600 s, set cover ~400 s
- The geometric sketching baseline total should be around ~30 minutes on the same matrix and sketch size
- The produced sketch_indicator_output.csv contains exactly sketch_size (20296) entries equal to 1

## Limitations

- No software version numbers are stated for Sphetcher or the downstream tools, complicating reproducibility of the timing benchmark
- The runtime comparison is reported for one dataset (MOCA, 2,026,641 cells) and one sketch size (1%); timing results on other datasets or sketch fractions are not covered
- Absolute timings are hardware-dependent, so only order-of-magnitude agreement with the reported ~16 vs ~30 minutes should be expected

## Evidence

- [intro] Scalability claim: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
- [methods] Build requirement: "A compiler that supports C++11 is needed to build sphetcher."
- [methods] MOCA dataset: "Mouse organogenesis cell atlas (MOCA) from [Cao et al. (2019)](https://www.nature.com/articles/s41586-019-0969-x) (2,026,641 cells)"
- [readme] CLI invocation format: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] Input matrix format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
