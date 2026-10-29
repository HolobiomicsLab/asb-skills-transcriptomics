---
name: scalability-curve-visualization
description: Use when you need to show a tool's runtime scales acceptably with dataset
  size — e.g., colorizing single-cell-resolution SRT datasets of 1k–100,000 cells
  within a bounded time budget.
when_to_use_negative:
- The goal is to compare colorization quality or visual clarity rather than runtime
  — use qualitative benchmarking on real SRT datasets (STARmap, Stereo-seq, seqFISH,
  IMC) instead
- The tool under test has no simulated-data generator and you only have small real
  samples — runtime scaling on few small inputs does not support a scaling-curve claim
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Spaco
  role: Tool being benchmarked; its full colorize pipeline (CI-graph construction,
    palette selection, cluster-color assignment) is timed at each dataset size
  repo: github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Source of the runtime benchmarking scripts used to reproduce the paper's scaling
    measurements
  repo: github.com/BrainStOrmics/Spaco_scripts
- name: Python
  role: Environment for generating simulated datasets, timing colorize runs, and recording
    results (python>=3.8.0)
- name: numpy
  role: Used to generate simulated cell coordinates and labels; its version (1.22.x/1.23.x)
    influences Spaco's graph-guided mode and must be recorded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:aa24a1000860cf3601e554ad466ed866adf6ce8167c77532b7e3f70c4bf62a44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  - numpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scalability-curve-visualization

## Summary

Benchmark the wall-clock runtime of a pipeline (here, the Spaco colorization pipeline) across simulated datasets of increasing size, record per-size timings, and plot a runtime-versus-scale curve to validate that the method stays practical (e.g., Spaco colorizes up to 100,000 cells in under 30 seconds).

## When to use

Use when you need to show a tool's runtime scales acceptably with dataset size — e.g., colorizing single-cell-resolution SRT datasets of 1k–100,000 cells within a bounded time budget.

## When NOT to use

- The goal is to compare colorization quality or visual clarity rather than runtime — use qualitative benchmarking on real SRT datasets (STARmap, Stereo-seq, seqFISH, IMC) instead
- The tool under test has no simulated-data generator and you only have small real samples — runtime scaling on few small inputs does not support a scaling-curve claim

## Inputs

- Simulated spatial datasets with cell coordinates and cell-type labels at increasing cell counts (1k, 5k, 10k, 50k, 100k cells), generated with numpy
- Runtime benchmarking scripts from the BrainStOrmics/Spaco_scripts repository

## Outputs

- runtime_benchmark_results.csv with per-size wall-clock runtimes
- Runtime-versus-cell-count scaling curve (plot)
- Validation statement that the 100,000-cell colorization completed in under 30 s

## How to apply

Install the tool in a pinned environment (python>=3.8.0, numpy>=1.18.0, installing Spaco via 'pip install git+https://github.com/BrainStOrmics/Spaco.git') and record the numpy version, since numpy 1.22.x/1.23.x influences the graph-guided mode. Generate simulated spatial datasets at increasing cell counts (e.g., 1k, 5k, 10k, 50k, up to 100,000 cells) with cell coordinates and cell-type labels using numpy. Time the wall-clock runtime of the full spaco.colorize pipeline — its three modules: cell cluster interlacement modeling, color palette selection, and cluster-color assignment — on each simulated size. Record per-size runtimes to runtime_benchmark_results.csv, then plot the runtime-versus-cell-count scaling curve and check the practicality bound (colorization of the 100,000-cell dataset completes in under 30 seconds).

## Related tools

- **Spaco** (Tool being benchmarked; its full colorize pipeline (CI-graph construction, palette selection, cluster-color assignment) is timed at each dataset size) — github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Source of the runtime benchmarking scripts used to reproduce the paper's scaling measurements) — github.com/BrainStOrmics/Spaco_scripts
- **Python** (Environment for generating simulated datasets, timing colorize runs, and recording results (python>=3.8.0))
- **numpy** (Used to generate simulated cell coordinates and labels; its version (1.22.x/1.23.x) influences Spaco's graph-guided mode and must be recorded)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; python -c "import time,numpy,spaco; t=time.time(); spaco.colorize(adata); print(time.time()-t)"
```

## Evaluation signals

- runtime_benchmark_results.csv contains one timing row per simulated size (1k, 5k, 10k, 50k, 100k cells) with wall-clock runtimes
- The 100,000-cell run completes in under 30 seconds, matching the reported benchmark
- The scaling curve is monotone (or near-monotone) in cell count with no size showing a runtime blow-up
- The numpy version used is recorded, since 1.22.x/1.23.x changes graph-guided mode results
- Timed runs cover the full three-module colorize pipeline (interlacement modeling, palette selection, cluster-color assignment), not a partial run

## Limitations

- Simulated datasets measure runtime only; they do not assess colorization quality on real SRT data
- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, so environment must be documented
- Spaco's visual contrast is limited when performing CVD-supporting colorization or coloring hundreds of clusters — runtime practicality does not imply output quality at extreme cluster counts

## Evidence

- [methods] Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts): "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [methods] numpy version influences graph-guided mode: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [intro] three primary modules: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [readme] README installation line: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
