---
name: scrna-seq-runtime-benchmarking
description: Use when comparing the runtime and memory footprint of pluripotency/scRNA-seq
  scoring tools (ORIGINS, LandSCENT SR, CytoTRACE) on GEO count matrices (breast GSE113197,
  colon GSE125970, hematopoietic GSE117498, lung GSE161383).
when_to_use_negative:
- The goal is to compare biological pluripotency scores (accuracy of stem-cell ranking)
  rather than computational performance — use the scoring skills, not this benchmarking
  skill.
- The input is not a scRNA-seq count matrix from the benchmark datasets/organism scope;
  runtime figures will not be comparable to the published reference values.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ORIGINS
  role: Benchmarked tool whose activity/activity-HVF runtime and RAM footprint (26.6
    Kb activity vector) are measured
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Competitor method (SR) benchmarked for elapsed time and extra RAM (6.4 Gb
    on breast dataset)
- name: CytoTRACE
  role: Competitor method benchmarked for elapsed time and extra RAM (5.8 Gb on breast
    dataset)
- name: R
  role: Execution environment; system.time() captures elapsed runtime and gc()/memory
    profiling records extra RAM allocated
- name: remotes
  role: 'Used to install ORIGINS from GitHub: remotes::install_github("danielasenraoka/ORIGINS")'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:245a2c3cf2cf007d2ad4040ec5eefca681b37f14ef67a174e82d24d2438281f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - SkewC
  - docker
  - remotes
  - singularity
  - udocker
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scrna-seq-runtime-benchmarking

## Summary

Benchmark computational efficiency (elapsed runtime via system.time() and extra RAM via gc()/memory profiling) of ORIGINS activity scoring against LandSCENT (SR) and CytoTRACE across scRNA-seq datasets, reproducing the paper's Table 1 performance comparison.

## When to use

Use when comparing the runtime and memory footprint of pluripotency/scRNA-seq scoring tools (ORIGINS, LandSCENT SR, CytoTRACE) on GEO count matrices (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383).

## When NOT to use

- The goal is to compare biological pluripotency scores (accuracy of stem-cell ranking) rather than computational performance — use the scoring skills, not this benchmarking skill.
- The input is not a scRNA-seq count matrix from the benchmark datasets/organism scope; runtime figures will not be comparable to the published reference values.

## Inputs

- GEO scRNA-seq count datasets: GSE113197 (breast), GSE125970 (colon), GSE117498 (hematopoietic), GSE161383 (lung)
- Expression matrix (genes as rows, cells as columns)
- differentiation_edges data frame (PPIN edges bundled within the ORIGINS package)

## Outputs

- runtime_memory_benchmark.csv with 16 tool-by-dataset rows containing elapsed runtime and extra RAM columns
- Per-tool per-dataset runtime and memory measurements comparable to Table 1 (e.g., breast: LandSCENT 3.43 h, CytoTRACE 2.95 h, activity 28.27 s, activity HVF 3.00 min)

## How to apply

Download the four GEO scRNA-seq count datasets and load each as an expression_matrix with genes as rows and cells as columns. Install the ORIGINS R package via remotes::install_github("danielasenraoka/ORIGINS") alongside LandSCENT and CytoTRACE. For each dataset, run SR (LandSCENT), CytoTRACE, activity(expression_matrix, differentiation_edges), and activity HVF, wrapping each call with system.time() for elapsed time and gc()/memory profiling for extra RAM allocated. Assemble a 16-row tool-by-dataset table (runtime_memory_benchmark.csv) alongside the reported reference values (breast: 3.43 h / 28.27 s / 2.95 h / 3.00 min; extra RAM 6.4 Gb LandSCENT, 5.8 Gb CytoTRACE, 26.6 Kb for the activity vector). The rationale is to demonstrate that ORIGINS is more efficient than LandSCENT and requires far less RAM than both competitors, since it only needs 26.6 Kb to store the activity vector.

## Related tools

- **ORIGINS** (Benchmarked tool whose activity/activity-HVF runtime and RAM footprint (26.6 Kb activity vector) are measured) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Competitor method (SR) benchmarked for elapsed time and extra RAM (6.4 Gb on breast dataset))
- **CytoTRACE** (Competitor method benchmarked for elapsed time and extra RAM (5.8 Gb on breast dataset))
- **R** (Execution environment; system.time() captures elapsed runtime and gc()/memory profiling records extra RAM allocated)
- **remotes** (Used to install ORIGINS from GitHub: remotes::install_github("danielasenraoka/ORIGINS"))

## Examples

```
library(ORIGINS); t <- system.time(diff_activity <- activity(expression_matrix, differentiation_edges)); gc()
```

## Evaluation signals

- runtime_memory_benchmark.csv exists and contains 16 tool-by-dataset rows with both runtime and RAM columns populated
- Measured ORIGINS activity runtimes are within the same order of magnitude as reported values (e.g., breast activity ≈ 28.27 s, activity HVF ≈ 3.00 min)
- ORIGINS requires far less RAM than competitors (26.6 Kb activity vector vs 6.4 Gb LandSCENT and 5.8 Gb CytoTRACE on breast)
- All four GEO datasets (GSE113197, GSE125970, GSE117498, GSE161383) appear in the results table
- Each timing capture uses a consistent environment (same R session setup) so elapsed-time comparisons are fair

## Limitations

- Reference runtime values are hardware- and environment-dependent; absolute timings will differ on other machines, so only relative ordering (ORIGINS faster and lighter) is robust.
- Memory profiling via gc() captures R-level allocations and may not reflect total process memory for tools with external dependencies.

## Evidence

- [abstract] We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs.: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [abstract] We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets"
- [methods] Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS"): "Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS")"
- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
