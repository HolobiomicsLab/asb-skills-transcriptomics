---
name: branching-event-detection-against-gold-standard
description: Use when evaluating whether a trajectory inference method (e.g. STREAM,
  Monocle2, TSCAN, DPT) correctly detects known bifurcations and recovers true pseudotime
  on a synthetic single-cell dataset with ground-truth topology.
when_to_use_negative:
- The dataset is real experimental data with no known ground-truth branching topology
  or true pseudotime — correlation metrics cannot be computed against a gold standard.
- The trajectory of interest is cyclic, disconnected, or has fates absent from a reference
  structure — the paper notes STREAM requires further adaptation for such general
  graph topologies.
- The input is not single-cell omics data (e.g. bulk or spatial data without per-cell
  ordering), since the compared methods are single-cell trajectory inference tools.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Primary trajectory inference method under test; detects branching events and
    estimates pseudotime using ElPiGraph-based elastic principal graphs
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the principal graph topology used
    for branching detection
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data container class in which the synthetic dataset is loaded and passed to
    STREAM
- name: conda
  role: Environment manager used to install the STREAM 1.0 reference implementation
    from the bioconda channel
- name: Monocle2
  role: Comparator trajectory inference method run on the same synthetic benchmark
    data
- name: Jupyter Notebook
  role: Interactive environment for running the benchmark analyses per the STREAM
    tutorials
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:7cfae1cefc1e61f399b8c9351fb824f512a9ef863edf8980c1863f71fe5f1d3e
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# branching-event-detection-against-gold-standard

## Summary

Benchmark a trajectory inference method's ability to detect branching events and estimate pseudotime by comparing its output against a gold-standard synthetic dataset with known bifurcations, following the STREAM paper's Rizvi et al. benchmark design.

## When to use

Use when evaluating whether a trajectory inference method (e.g. STREAM, Monocle2, TSCAN, DPT) correctly detects known bifurcations and recovers true pseudotime on a synthetic single-cell dataset with ground-truth topology.

## When NOT to use

- The dataset is real experimental data with no known ground-truth branching topology or true pseudotime — correlation metrics cannot be computed against a gold standard.
- The trajectory of interest is cyclic, disconnected, or has fates absent from a reference structure — the paper notes STREAM requires further adaptation for such general graph topologies.
- The input is not single-cell omics data (e.g. bulk or spatial data without per-cell ordering), since the compared methods are single-cell trajectory inference tools.

## Inputs

- Rizvi et al. gold-standard synthetic scRNA-seq dataset with two known bifurcation events and three simulated time points
- anndata (.h5ad) single-cell expression matrix
- Ground-truth pseudotime ordering and branching topology for each cell

## Outputs

- Per-method branching-event detection counts against the two-bifurcation ground truth
- Four pseudotime correlation metrics per method: rank-based Pearson, distance-based Pearson, Spearman, Kendall's tau
- benchmark_fig4b.csv comparison table of all eleven methods

## How to apply

Load the Rizvi et al. gold-standard synthetic dataset (two bifurcations, three simulated time points) into an anndata object, then run each trajectory inference method — STREAM plus ten comparators (Monocle2, scTDA, Wishbone, TSCAN, SLICER, DPT, GPfates, Mpath, SCUBA, PHATE) — using each tool's default recommended settings. For every method, count correctly detected branching events against the known two-bifurcation ground truth. Compute pseudotime accuracy as rank-based Pearson correlation, distance-based Pearson correlation, Spearman correlation and Kendall's tau between inferred and true pseudotime orderings across all cells. Assemble branching counts and the four correlation metrics for all eleven methods into a single table (benchmark_fig4b.csv) mirroring the layout of Fig. 4b. STREAM should use the reference implementation installed via bioconda (stream=1.0 in a Python 3.7 conda environment), since it is built on the anndata class and ElPiGraph elastic principal graphs.

## Related tools

- **STREAM** (Primary trajectory inference method under test; detects branching events and estimates pseudotime using ElPiGraph-based elastic principal graphs) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm behind STREAM that learns the principal graph topology used for branching detection) — sysbio-curie/ElPiGraph.R
- **anndata** (Data container class in which the synthetic dataset is loaded and passed to STREAM)
- **conda** (Environment manager used to install the STREAM 1.0 reference implementation from the bioconda channel)
- **Monocle2** (Comparator trajectory inference method run on the same synthetic benchmark data)
- **Jupyter Notebook** (Interactive environment for running the benchmark analyses per the STREAM tutorials)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- STREAM's row in benchmark_fig4b.csv shows correct detection of both synthetic bifurcation events (count = 2 against known ground truth).
- All four correlation metrics (rank-based Pearson, distance-based Pearson, Spearman, Kendall's tau) are computed for all cells, not a subset.
- Every one of the eleven methods (STREAM + ten comparators) is represented by a complete metric row in the assembled table.
- Each comparator method was run with its default recommended settings rather than tuned parameters, ensuring fair comparison.
- The table layout mirrors Fig. 4b of the STREAM paper, allowing direct visual diffing against the published benchmark.

## Limitations

- STREAM does not support cyclic or disconnected graph topologies; further improvement is necessary for such general structures.
- ElPiGraph, the core algorithm behind STREAM, supports learning complex graphs but more work is needed for intuitive visualizations of sophisticated topologies.
- Benchmark conclusions may not extend to datasets with millions of cells, since many current trajectory inference methods including STREAM do not scale to such sizes.
- Comparison results depend on default/recommended settings of each method and the specific synthetic dataset; other benchmarks may rank methods differently.

## Evidence

- [results] STREAM ranked best for two of four pseudotime correlation metrics: [evidence span withheld — anchor status: not_found]
- [methods] conda environment with stream 1.0: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [methods] STREAM written using anndata class: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] ElPiGraph core algorithm: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [discussion] no prior knowledge required: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [discussion] cyclic/disconnected limitation: "Therefore, further improvement is necessary to adapt STREAM to properly describing these more general structures"
- [readme] README install instructions: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it"
