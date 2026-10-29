---
name: simulated-spatial-dataset-generation
description: Use when benchmarking the scalability or runtime of a spatial colorization
  method (e.g., Spaco) and you need controlled synthetic SRT inputs with known cell
  counts and labels instead of real tissue sections.
when_to_use_negative:
- You need biologically realistic tissue structure or real SRT measurements — this
  skill produces synthetic coordinates/labels for benchmarking only
- Your goal is to evaluate colorization visual quality or spatial contrast rather
  than runtime/scalability — use real benchmark datasets (e.g., STARmap mouse brain,
  seqFISH mouse embryo) instead
- The dataset exceeds the benchmarked range or is 3D, where parameter fine-tuning
  is required and runtime behavior is not established
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spaco
  role: Target pipeline whose colorize() runtime is being benchmarked on simulated
    datasets
  repo: https://github.com/BrainStOrmics/Spaco
- name: numpy
  role: Used to generate simulated cell coordinates and cell-type labels at each dataset
    size; version 1.22.x/1.23.x influences graph-guided mode
- name: Python
  role: Runtime environment (python>=3.8.0) for simulation and benchmarking scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:303cdb9ca10bc0c2608f36099f09f86e1e982e5b5cf8836301e87a33c5eaf819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - numpy
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# simulated-spatial-dataset-generation

## Summary

Generates synthetic spatially resolved transcriptomics datasets — cell coordinates plus cell-type labels — at increasing cell counts (1k up to 100,000 cells) using numpy, so that the wall-clock runtime of the Spaco colorization pipeline (cluster interlacement modeling, palette selection, cluster-color assignment) can be benchmarked for scalability.

## When to use

Use when benchmarking the scalability or runtime of a spatial colorization method (e.g., Spaco) and you need controlled synthetic SRT inputs with known cell counts and labels instead of real tissue sections.

## When NOT to use

- You need biologically realistic tissue structure or real SRT measurements — this skill produces synthetic coordinates/labels for benchmarking only
- Your goal is to evaluate colorization visual quality or spatial contrast rather than runtime/scalability — use real benchmark datasets (e.g., STARmap mouse brain, seqFISH mouse embryo) instead
- The dataset exceeds the benchmarked range or is 3D, where parameter fine-tuning is required and runtime behavior is not established

## Inputs

- Simulated spatial dataset parameters: target cell counts (1k, 5k, 10k, 50k, up to 100,000 cells)
- Cell coordinate arrays and cell-type label vectors generated with numpy
- Spaco runtime benchmarking scripts from the Spaco_scripts repository

## Outputs

- runtime_benchmark_results.csv with per-size wall-clock runtimes of spaco.colorize
- Runtime-versus-cell-count scaling curve (plot)
- Validation check that the 100,000-cell colorization completes in under 30 seconds

## How to apply

Install Spaco (pip install git+https://github.com/BrainStOrmics/Spaco.git) in a Python>=3.8.0 environment with numpy>=1.18.0, recording the numpy version because 1.22.x/1.23.x influences graph-guided mode results. Use the runtime benchmarking scripts from the Spaco_scripts repository and numpy to generate simulated spatial datasets at increasing cell counts (e.g., 1k, 5k, 10k, 50k, 100,000 cells), each containing cell coordinates and cell-type labels. For each size, run the full spaco.colorize pipeline — which comprises cell cluster interlacement modeling, color palette selection, and cluster-color assignment — while timing the wall-clock runtime. Record per-size runtimes (e.g., into runtime_benchmark_results.csv), then plot the runtime-versus-cell-count scaling curve to assess whether runtime remains practical; the reference benchmark validates that colorization of 100,000 cells completes in under 30 seconds.

## Related tools

- **Spaco** (Target pipeline whose colorize() runtime is being benchmarked on simulated datasets) — https://github.com/BrainStOrmics/Spaco
- **numpy** (Used to generate simulated cell coordinates and cell-type labels at each dataset size; version 1.22.x/1.23.x influences graph-guided mode)
- **Python** (Runtime environment (python>=3.8.0) for simulation and benchmarking scripts)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata_cell, cell_type_key=...)
```

## Evaluation signals

- runtime_benchmark_results.csv contains one wall-clock runtime entry per simulated cell count (1k, 5k, 10k, 50k, 100k)
- The 100,000-cell spaco.colorize run completes in under 30 seconds, matching the reported benchmark
- Each simulated dataset contains both cell coordinates and cell-type labels
- The runtime-versus-cell-count scaling curve is plotted and shows no explosive (non-practical) growth with dataset size
- The numpy version is recorded, since 1.22.x/1.23.x influences graph-guided mode results

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco
- Simulated datasets probe runtime only; they do not assess colorization visual quality on real tissue
- Fine-tuning of parameters is crucial for efficient 3D colorization, so 3D or extremely sparse/dense synthetic configurations are out of scope

## Evidence

- [results] Spaco's full colorization pipeline completes in under 30 seconds on simulated datasets of up to 100,000 cells: [evidence span withheld — anchor status: not_found]
- [intro] Three primary modules of Spaco colorization: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] numpy version influences graph-guided mode: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] Spaco installation via pip: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
- [readme] Benchmarking scripts repository: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
