---
name: cross-tool-performance-comparison
description: Use when comparing the runtime and RAM footprint of ORIGINS, LandSCENT,
  and CytoTRACE on the same GEO scRNA-seq count matrices to reproduce or extend the
  paper's Table 1 performance benchmark.
when_to_use_negative:
- When the goal is biological accuracy of pluripotency ordering rather than computational
  cost — the paper's benchmark measures runtime and RAM, not whether methods recover
  the classical hierarchy (e.g. GMP outscored HSC in all methods).
- When input is a precomputed potency score table rather than a raw gene-by-cell count
  matrix; the benchmark requires running each tool end-to-end on expression data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ORIGINS
  role: Tool under benchmark; computes the differentiation activity score per cell
    via activity(expression_matrix, differentiation_edges) and is measured for runtime
    and extra RAM
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Competitor tool whose SR (single-cell purity) computation is timed and memory-profiled
    as the efficiency reference point
- name: CytoTRACE
  role: Competitor tool whose potency scoring is timed and memory-profiled alongside
    LandSCENT and ORIGINS
- name: R
  role: Environment in which all three tools are installed and benchmarked with system.time()
    and gc()/memory profiling
- name: remotes
  role: Used to install ORIGINS from GitHub via remotes::install_github("danielasenraoka/ORIGINS")
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:ac9ef4cebaa051caf75826c71c09d22511c81498dcbc641e9b0bfad1f57d4a98
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-tool-performance-comparison

## Summary

Benchmark the computational efficiency (elapsed runtime) and extra RAM usage of ORIGINS activity scoring against LandSCENT (SR) and CytoTRACE across the four healthy human scRNA-seq datasets (breast, colon, hematopoietic, lung), reproducing the Table 1 runtime/memory comparison that motivated ORIGINS as a lightweight pluripotency tool.

## When to use

Use when comparing the runtime and RAM footprint of ORIGINS, LandSCENT, and CytoTRACE on the same GEO scRNA-seq count matrices to reproduce or extend the paper's Table 1 performance benchmark.

## When NOT to use

- When the goal is biological accuracy of pluripotency ordering rather than computational cost — the paper's benchmark measures runtime and RAM, not whether methods recover the classical hierarchy (e.g. GMP outscored HSC in all methods).
- When input is a precomputed potency score table rather than a raw gene-by-cell count matrix; the benchmark requires running each tool end-to-end on expression data.

## Inputs

- GEO scRNA-seq count matrices: GSE113197 (breast), GSE125970 (colon), GSE117498 (hematopoietic), GSE161383 (lung), as gene-by-cell expression matrices
- differentiation_edges data frame (differentiation PPIN loaded within the ORIGINS package data directory)

## Outputs

- runtime_memory_benchmark.csv with 16 tool-by-dataset rows containing elapsed runtime and extra RAM columns
- Per-dataset potency/activity score vectors from ORIGINS activity, activity HVF, LandSCENT SR, and CytoTRACE

## How to apply

Download the four GEO count datasets (GSE113197 breast, GSE125970 colon, GSE117498 hematopoietic, GSE161383 lung) and load each as an expression_matrix with genes in rows and cells in columns. Install ORIGINS via remotes::install_github("danielasenraoka/ORIGINS") plus the published LandSCENT and CytoTRACE packages in the same R environment. For each tool-dataset pair, run SR (LandSCENT), CytoTRACE, activity(expression_matrix, differentiation_edges), and the activity HVF variant, wrapping each call with system.time() to capture elapsed runtime and using gc()/memory profiling to record extra RAM allocated. Assemble a results table with one row per tool per dataset (16 rows total) alongside the paper's reference values, e.g. breast dataset runtimes of 3.43 h (LandSCENT), 28.27 s (CytoTRACE), 2.95 h (activity), 3.00 min (activity HVF), and extra RAM of 6.4 Gb (LandSCENT), 5.8 Gb (CytoTRACE) versus only 26.6 Kb for the ORIGINS activity vector. The rationale is the paper's claim that ORIGINS is more efficient than LandSCENT and requires far less RAM than both competitors, since it only stores a per-cell activity vector.

## Related tools

- **ORIGINS** (Tool under benchmark; computes the differentiation activity score per cell via activity(expression_matrix, differentiation_edges) and is measured for runtime and extra RAM) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Competitor tool whose SR (single-cell purity) computation is timed and memory-profiled as the efficiency reference point)
- **CytoTRACE** (Competitor tool whose potency scoring is timed and memory-profiled alongside LandSCENT and ORIGINS)
- **R** (Environment in which all three tools are installed and benchmarked with system.time() and gc()/memory profiling)
- **remotes** (Used to install ORIGINS from GitHub via remotes::install_github("danielasenraoka/ORIGINS"))

## Examples

```
library(ORIGINS); t <- system.time(diff_activity <- activity(expression_matrix, differentiation_edges)); gc()
```

## Evaluation signals

- runtime_memory_benchmark.csv exists and contains 16 tool-by-dataset rows (4 tools x 4 datasets) with both runtime and extra-RAM columns populated
- Measured breast-dataset extra RAM reproduces the paper's pattern: LandSCENT ~6.4 Gb and CytoTRACE ~5.8 Gb versus only 26.6 Kb for the ORIGINS activity vector
- Measured ORIGINS activity runtimes are within the same order of magnitude as reported values (e.g. 2.95 h for activity and 3.00 min for activity HVF on the breast dataset)
- Each tool call on the same expression matrix yields a per-cell score vector, confirming all methods were run on identical inputs
- ORIGINS install succeeds via remotes::install_github and library(ORIGINS) loads with differentiation_edges available in the package data directory

## Limitations

- Benchmark is limited to four healthy human datasets (breast, colon, hematopoietic, lung); performance may differ on other tissues or diseased samples
- CytoTRACE was faster than ORIGINS in wall-clock runtime on some datasets (28.27 s vs 2.95 h on breast), so ORIGINS' advantage is primarily RAM usage rather than elapsed time
- Runtime and RAM measurements depend on the benchmarking machine and environment; absolute values may not transfer, only the relative ordering of tools

## Evidence

- [abstract] ORIGINS is more efficient than LandSCENT and requires less RAM memory than LandSCENT and CytoTRACE: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [abstract] Performance benchmark scope: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
- [methods] ORIGINS installation command: "remotes::install_github("danielasenraoka/ORIGINS")"
- [methods] Core activity computation call: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] Differentiation PPIN availability in package: "The differentiation PPIN is loaded within the package in the data directory."
- [supplementary] RAM comparison reference values from source card workflow: [evidence span withheld — anchor status: not_found]
