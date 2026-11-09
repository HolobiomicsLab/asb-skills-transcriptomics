---
name: multi-method-comparison-table-compilation
description: Use when a gold-standard dataset with known branching topology (e.g.,
  Rizvi et al. synthetic data with two bifurcations) is available and the goal is
  to rank multiple trajectory-inference methods on bifurcation detection and pseudotime
  accuracy.
when_to_use_negative:
- The dataset has no ground truth for branching events or true pseudotime, so bifurcation
  detection and correlation-to-truth cannot be scored
- Only one method is of interest and no cross-method ranking table is required
- The target topology is cyclic or disconnected rather than a branching tree, which
  STREAM and this benchmark design do not properly describe (see article's own limitations)
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Focal method being benchmarked against ten alternatives; installed via bioconda
    as the reference implementation in the benchmark environment
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data container used to load the gold-standard dataset so all methods consume
    the same input object; STREAM is written using the anndata class
  repo: http://anndata.rtfd.io
- name: conda
  role: Creates the reproducible `env_stream` environment (python=3.7, stream=1.0,
    jupyter) in which the benchmark runs
- name: Jupyter Notebook
  role: Interactive execution environment for running the methods and assembling the
    comparison table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:e154702247df355078c78f37b0c7cbcb75dcc7928a13253d23d4d412ee017c2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Jupyter Notebook
  - Monocle2
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-method-comparison-table-compilation

## Summary

Compile a side-by-side benchmark table that compares a focal trajectory-inference tool (e.g., STREAM) against ten state-of-the-art alternatives on a gold-standard dataset, scoring branching-event detection and four pseudotime correlation metrics into a single CSV mirroring the paper's benchmark figure layout.

## When to use

Use when a gold-standard dataset with known branching topology (e.g., Rizvi et al. synthetic data with two bifurcations) is available and the goal is to rank multiple trajectory-inference methods on bifurcation detection and pseudotime accuracy.

## When NOT to use

- The dataset has no ground truth for branching events or true pseudotime, so bifurcation detection and correlation-to-truth cannot be scored
- Only one method is of interest and no cross-method ranking table is required
- The target topology is cyclic or disconnected rather than a branching tree, which STREAM and this benchmark design do not properly describe (see article's own limitations)

## Inputs

- Gold-standard synthetic benchmark dataset with known trajectory topology and true pseudotime (Rizvi et al.: two bifurcations, three simulated time points), loaded as an anndata object
- Reference implementations of the 11 trajectory-inference methods (STREAM, Monocle2, scTDA, Wishbone, TSCAN, SLICER, DPT, GPfates, Mpath, SCUBA, PHATE)
- Ground-truth annotations: number and location of bifurcation events and true pseudotime ordering per cell

## Outputs

- benchmark_fig4b.csv: one complete row per method (11 rows) with branching-event count and four pseudotime correlation metrics (rank-based Pearson, distance-based Pearson, Spearman, Kendall's tau), laid out to mirror Fig. 4b
- Per-method branching detection counts against the known two-bifurcation ground truth

## How to apply

Set up a single reproducible environment (e.g., conda env `env_stream` with `python=3.7 stream=1.0 jupyter` from bioconda) and load the gold-standard dataset into an `anndata` object, since STREAM is written on the `anndata` class. Run all eleven methods (STREAM, Monocle2, scTDA, Wishbone, TSCAN, SLICER, DPT, GPfates, Mpath, SCUBA, PHATE) on the same input using each tool's default recommended settings so the comparison is settings-fair. For each method, count correctly detected branching events against the known ground truth (two bifurcations in the Rizvi synthetic benchmark). For each method, compute four pseudotime accuracy metrics against the true ordering: rank-based Pearson correlation, distance-based Pearson correlation, Spearman correlation, and Kendall's tau, over all cells. Assemble branching counts and the four correlations for all eleven methods into one table (benchmark_fig4b.csv) whose layout mirrors Fig. 4b of the STREAM paper, so every tool appears as a complete metric row and rankings (best/second-best per metric) are directly readable.

## Related tools

- **STREAM** (Focal method being benchmarked against ten alternatives; installed via bioconda as the reference implementation in the benchmark environment) — https://github.com/pinellolab/STREAM
- **anndata** (Data container used to load the gold-standard dataset so all methods consume the same input object; STREAM is written using the anndata class) — http://anndata.rtfd.io
- **conda** (Creates the reproducible `env_stream` environment (python=3.7, stream=1.0, jupyter) in which the benchmark runs)
- **Jupyter Notebook** (Interactive execution environment for running the methods and assembling the comparison table)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- All eleven methods (STREAM, Monocle2, scTDA, Wishbone, TSCAN, SLICER, DPT, GPfates, Mpath, SCUBA, PHATE) are represented by a complete metric row with no missing values
- STREAM's row shows correct detection of both synthetic bifurcations (2 of 2 against ground truth)
- All four correlation values (rank-based Pearson, distance-based Pearson, Spearman, Kendall's tau) are computed for every cell in the dataset, not a subsample
- The table layout mirrors Fig. 4b, allowing per-metric ranking (e.g., STREAM best on distance-based metrics, second-best on rank-based metrics)
- Methods were run with each tool's default recommended settings — verify no method used tuned parameters that would break settings-fairness

## Limitations

- STREAM and many current trajectory-inference methods do not scale to recent single-cell assays profiling millions of cells
- STREAM cannot properly describe cyclic or disconnected graph topologies; further improvement is necessary for these more general structures
- ElPiGraph (the core algorithm behind STREAM) supports complex graphs, but intuitive visualizations for sophisticated topologies remain insufficient

## Evidence

- [methods] STREAM benchmarked with conda reference environment: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [readme] STREAM data container is anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)](http://anndata.rtfd.io)"
- [readme] bioconda installation of reference implementation: "$ conda install -c bioconda stream"
- [discussion] STREAM validated as a trajectory and pseudotime inference tool across data types: "In this study we presented STREAM, a trajectory inference tool capable of reliably reconstructing trajectories and inferring pseudotime from different data types"
