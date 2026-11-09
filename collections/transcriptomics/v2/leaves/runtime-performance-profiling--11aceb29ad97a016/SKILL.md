---
name: runtime-performance-profiling
description: Use when you need to show a pipeline's runtime scales practically with
  dataset size, e.g. validating that Spaco colorization completes within bounded time
  on single-cell-resolution SRT datasets up to 100,000 cells.
when_to_use_negative:
- The goal is to assess colorization quality or visual contrast rather than computational
  cost — use visual/benchmark comparison workflows instead
- The pipeline requires real biological data with specific spatial structure; simulated
  coordinate/label datasets only support runtime scaling claims, not biological validity
- 3D colorization is the target, since the article notes parameter fine-tuning to
  distinguish 'sparse' from 'dense' cell types makes 3D behavior differ from the 2D
  scaling benchmark
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spaco
  role: Tool whose full colorization pipeline (CI-graph modeling, palette selection,
    cluster-color assignment) is timed via spaco.colorize
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: Execution environment (python>=3.8.0) in which spaco.colorize runs are timed
- name: numpy
  role: Used to generate simulated cell coordinates and cell-type labels; version
    (1.22.x/1.23.x) influences graph-guided mode results and must be recorded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:1ccb37733332247d0fdb71ac1bb7cc18df4d4803c51324701803f06fb8896712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - numpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# runtime-performance-profiling

## Summary

Benchmark wall-clock runtime of a computational pipeline (here, the Spaco spatial colorization pipeline) on simulated datasets of increasing size to characterize how runtime scales with input size and verify it stays within a practical bound (under 30 s for up to 100,000 cells).

## When to use

Use when you need to show a pipeline's runtime scales practically with dataset size, e.g. validating that Spaco colorization completes within bounded time on single-cell-resolution SRT datasets up to 100,000 cells.

## When NOT to use

- The goal is to assess colorization quality or visual contrast rather than computational cost — use visual/benchmark comparison workflows instead
- The pipeline requires real biological data with specific spatial structure; simulated coordinate/label datasets only support runtime scaling claims, not biological validity
- 3D colorization is the target, since the article notes parameter fine-tuning to distinguish 'sparse' from 'dense' cell types makes 3D behavior differ from the 2D scaling benchmark

## Inputs

- Simulated spatial transcriptomics datasets with cell coordinates and cell-type labels at increasing sizes (1k, 5k, 10k, 50k, up to 100,000 cells), generated with numpy
- Runtime benchmarking scripts from the BrainStOrmics/Spaco_scripts repository

## Outputs

- runtime_benchmark_results.csv with per-size wall-clock runtimes
- Runtime-versus-cell-count scaling curve (plot)

## How to apply

Install the tool in a pinned environment (Spaco via pip install git+https://github.com/BrainStOrmics/Spaco.git, python>=3.8.0, numpy>=1.18.0) and record the numpy version, since numpy 1.22.x/1.23.x influences the graph-guided mode results. Generate simulated spatial datasets at increasing cell counts (e.g. 1k, 5k, 10k, 50k, 100,000 cells) with cell coordinates and cell-type labels using numpy. Run the full spaco.colorize pipeline — cell cluster interlacement modeling, color palette selection, and cluster-color assignment — on each size, timing wall-clock runtime per run, then write per-size runtimes to runtime_benchmark_results.csv and plot the runtime-versus-cell-count scaling curve. Judge success by whether the 100,000-cell colorization completes in under 30 seconds, matching the reported benchmark behavior.

## Related tools

- **Spaco** (Tool whose full colorization pipeline (CI-graph modeling, palette selection, cluster-color assignment) is timed via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **Python** (Execution environment (python>=3.8.0) in which spaco.colorize runs are timed)
- **numpy** (Used to generate simulated cell coordinates and cell-type labels; version (1.22.x/1.23.x) influences graph-guided mode results and must be recorded)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; python -c "import time, spaco; t=time.time(); m=spaco.colorize(adata, rad_cutoff=50); print(time.time()-t)"
```

## Evaluation signals

- The 100,000-cell colorization completes in under 30 seconds of wall-clock time
- Per-size runtimes are recorded for every simulated size (1k through 100k) in runtime_benchmark_results.csv
- The runtime-versus-cell-count scaling curve is plotted and shows no explosive growth with dataset size
- The numpy version is recorded, since 1.22.x/1.23.x can change graph-guided mode results
- Each timed run executed all three Spaco modules: cell cluster interlacement modeling, color palette selection, and cluster-color assignment

## Limitations

- numpy versions 1.22.x and 1.23.x can influence the result of the graph-guided mode of Spaco, so timings across environments may not be directly comparable
- Runtimes are measured on simulated datasets and may not transfer to real SRT data with different cluster structures
- The benchmark supports only the 'under 30 s at 100,000 cells' claim; behavior beyond 100,000 cells is untested here

## Evidence

- [results] Spaco's full colorization pipeline completes in under 30 seconds on simulated datasets of up to 100,000 cells, indicating that the additional computations do not introduce a substantial time burden.: [evidence span withheld — anchor status: not_found]
- [intro] the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts): "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository"
- [methods] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
