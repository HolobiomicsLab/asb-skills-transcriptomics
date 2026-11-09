---
name: memory-profiling-of-r-packages
description: Use when you must quantitatively compare computational efficiency (elapsed
  runtime and RAM usage) of R-based single-cell tools on identical count-matrix inputs,
  e.g. reproducing a Table 1-style runtime/memory benchmark.
when_to_use_negative:
- The goal is to compare biological pluripotency-score accuracy across methods, not
  computational performance — use the scoring outputs directly instead of timing/profiling
  them.
- The tool under test is not available as an R package with a loadable library (e.g.
  it is a Python implementation such as PyOrigins), so system.time()-based profiling
  does not apply.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Benchmarked R package whose activity function (run as activity(expression_matrix,
    differentiation_edges) and activity HVF) is timed and memory-profiled against
    competitors
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Competitor tool whose SR computation is timed and memory-profiled for comparison
    (reported 3.43 h runtime, 6.4 Gb extra RAM on breast data)
- name: CytoTRACE
  role: Competitor tool timed and memory-profiled for comparison (reported 3.00 min
    runtime, 5.8 Gb extra RAM on breast data)
- name: remotes
  role: Used to install ORIGINS from GitHub before benchmarking via remotes::install_github("danielasenraoka/ORIGINS")
- name: R
  role: Execution environment; system.time() and gc() are used to capture elapsed
    time and extra RAM allocated per tool call
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:4bcec81ff3bbc650a87017136e632943ebf1a7841a4b2029187c4af76fa8f825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# memory-profiling-of-r-packages

## Summary

Benchmark the elapsed runtime and extra RAM allocated by R packages on the same scRNA-seq inputs, as done when ORIGINS was compared with LandSCENT (SR) and CytoTRACE on four GEO datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383).

## When to use

Use when you must quantitatively compare computational efficiency (elapsed runtime and RAM usage) of R-based single-cell tools on identical count-matrix inputs, e.g. reproducing a Table 1-style runtime/memory benchmark.

## When NOT to use

- The goal is to compare biological pluripotency-score accuracy across methods, not computational performance — use the scoring outputs directly instead of timing/profiling them.
- The tool under test is not available as an R package with a loadable library (e.g. it is a Python implementation such as PyOrigins), so system.time()-based profiling does not apply.

## Inputs

- GEO scRNA-seq count matrices (GSE113197 breast, GSE125970 colon, GSE117498 hematopoietic, GSE161383 lung), genes as rows and cells as columns
- differentiation_edges data frame of PPIN edges (bundled in the ORIGINS package data directory)
- Installed R packages: ORIGINS, LandSCENT, CytoTRACE

## Outputs

- runtime_memory_benchmark.csv with 16 tool-by-dataset rows containing elapsed runtime and extra RAM columns
- Per-call timing objects from system.time() and memory profiles from gc()/memory profiling

## How to apply

Load each dataset (e.g. GSE113197 breast counts) as an expression_matrix with genes as rows and cells as columns, then run every competitor on the same matrix: SR via LandSCENT, CytoTRACE, and ORIGINS' activity(expression_matrix, differentiation_edges) — the differentiation PPIN edges ship inside the ORIGINS package data directory. Wrap each call with system.time() to capture elapsed runtime and use gc()/memory profiling to record extra RAM allocated, one measurement per tool-per-dataset. Assemble a results table with one row per tool-dataset pair alongside reported reference values (breast: 3.43 h SR, 28.27 s activity, 2.95 h activity HVF, 3.00 min CytoTRACE; extra RAM 6.4 Gb LandSCENT, 5.8 Gb CytoTRACE, only 26.6 Kb for the ORIGINS activity vector). The rationale is that ORIGINS stores only a compact activity vector per cell, so its memory footprint is orders of magnitude smaller than competitors that materialize full diffusion/network objects.

## Related tools

- **ORIGINS** (Benchmarked R package whose activity function (run as activity(expression_matrix, differentiation_edges) and activity HVF) is timed and memory-profiled against competitors) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Competitor tool whose SR computation is timed and memory-profiled for comparison (reported 3.43 h runtime, 6.4 Gb extra RAM on breast data))
- **CytoTRACE** (Competitor tool timed and memory-profiled for comparison (reported 3.00 min runtime, 5.8 Gb extra RAM on breast data))
- **remotes** (Used to install ORIGINS from GitHub before benchmarking via remotes::install_github("danielasenraoka/ORIGINS"))
- **R** (Execution environment; system.time() and gc() are used to capture elapsed time and extra RAM allocated per tool call)

## Examples

```
Rscript -e 'library(ORIGINS); t <- system.time(diff_activity <- activity(expression_matrix, differentiation_edges)); print(t); print(gc())'
```

## Evaluation signals

- runtime_memory_benchmark.csv exists with exactly 16 rows (4 tools x 4 datasets) and both runtime and RAM columns populated
- Measured ORIGINS activity values fall within the same order of magnitude as the reported values
- ORIGINS extra RAM (26.6 Kb activity vector on breast) is orders of magnitude smaller than LandSCENT (6.4 Gb) and CytoTRACE (5.8 Gb)
- Each measurement corresponds to a single tool call on an identical expression_matrix, confirmed by the profiling wrapper (system.time / gc)

## Limitations

- Reference benchmark values are reported for four healthy human datasets only (breast, colon, hematopoietic, lung); other tissues or organisms are not covered by the published Table 1 values.
- The ORIGINS paper benchmarks activity and activity HVF variants; runtime will differ if the extended ORIGINS2 package or the Python PyOrigins implementation is used instead.
- Memory profiling via gc() captures extra RAM allocated during the call and may miss memory held before or freed differently across R versions and BLAS backends.

## Evidence

- [abstract] ORIGINS is more efficient than LandSCENT and requires less RAM memory than LandSCENT and CytoTRACE: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [abstract] Performance was benchmarked against LandSCENT and CytoTRACE on four healthy human datasets (breast, colon, hematopoietic, lung): "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
- [methods] Installation path used to set up the benchmarked package: "Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS")"
- [readme] Package purpose and data shipped for benchmarking: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [readme] Concrete call whose runtime and RAM are profiled: "diff_activity <- activity(expression_matrix, differentiation_edges)"
