---
name: synthetic-benchmark-dataset-evaluation
description: Use when a gold-standard synthetic single-cell dataset with known ground-truth
  trajectory topology (e.g. two bifurcations) and true pseudotime is available, and
  multiple trajectory-inference methods must be compared on branching detection and
  pseudotime ordering.
when_to_use_negative:
- No ground truth (known bifurcations or true pseudotime) exists, so accuracy cannot
  be computed against reference topology — use real-data qualitative comparison instead
- Only one method is being run without any comparison baseline; the skill is defined
  by cross-method comparison against a benchmark
- The dataset topology is not tree-like (e.g. cyclic or disconnected graphs), which
  STREAM's authors flag as unsupported territory
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Method under test and reference trajectory-inference pipeline; installed via
    bioconda stream=1.0 in a Python 3.7 conda environment
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data container used by STREAM to load the synthetic benchmark dataset
- name: conda
  role: Environment and dependency manager for installing STREAM from the bioconda
    channel
- name: ElPiGraph
  role: Core principal-graph algorithm underlying STREAM's trajectory inference
  repo: sysbio-curie/ElPiGraph.R
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:7f5d18b1c9d4b0bb31b3af42fe0c24a235028f95f96501b4bd99385592239658
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# synthetic-benchmark-dataset-evaluation

## Summary

Evaluate trajectory-inference methods (e.g. STREAM versus ten competitors such as Monocle2, TSCAN, DPT, PHATE) on a gold-standard synthetic dataset with known branching structure, scoring bifurcation detection and pseudotime accuracy with rank- and distance-based correlation metrics. This mirrors the STREAM paper's Rizvi et al. synthetic benchmark with two bifurcations and three simulated time points.

## When to use

Use when a gold-standard synthetic single-cell dataset with known ground-truth trajectory topology (e.g. two bifurcations) and true pseudotime is available, and multiple trajectory-inference methods must be compared on branching detection and pseudotime ordering.

## When NOT to use

- No ground truth (known bifurcations or true pseudotime) exists, so accuracy cannot be computed against reference topology — use real-data qualitative comparison instead
- Only one method is being run without any comparison baseline; the skill is defined by cross-method comparison against a benchmark
- The dataset topology is not tree-like (e.g. cyclic or disconnected graphs), which STREAM's authors flag as unsupported territory

## Inputs

- Gold-standard synthetic single-cell dataset with known trajectory topology (two bifurcations) and simulated time points (e.g. Rizvi et al. synthetic dataset)
- Ground-truth pseudotime ordering for all cells
- anndata-formatted expression matrix for methods written on anndata such as STREAM

## Outputs

- Per-method branching-event detection counts versus the two-bifurcation ground truth
- Four pseudotime correlation metrics per method: rank-based Pearson, distance-based Pearson, Spearman correlation, Kendall's tau
- Consolidated benchmark table (benchmark_fig4b.csv) with one complete metric row per method, mirroring Fig. 4b

## How to apply

Set up each method in its reference implementation environment — for STREAM, create a conda env with Python 3.7 and stream=1.0 from bioconda (`conda create -n env_stream python=3.7 stream=1.0 jupyter`). Load the synthetic dataset (two bifurcations, three simulated time points) into an anndata object, since STREAM is written using the `anndata` class. Run all eleven methods (STREAM, Monocle2, scTDA, Wishbone, TSCAN, SLICER, DPT, GPfates, Mpath, SCUBA, PHATE) on the same data using each tool's default recommended settings. For every method, count detected branching events against the known two-bifurcation ground truth, and compute pseudotime accuracy as rank-based Pearson correlation, distance-based Pearson correlation, Spearman correlation, and Kendall's tau between inferred and true pseudotime orderings over all cells. Assemble branching counts and the four correlation metrics for all methods into a single table (benchmark_fig4b.csv) mirroring the layout of Fig. 4b of the STREAM paper.

## Related tools

- **STREAM** (Method under test and reference trajectory-inference pipeline; installed via bioconda stream=1.0 in a Python 3.7 conda environment) — https://github.com/pinellolab/STREAM
- **anndata** (Data container used by STREAM to load the synthetic benchmark dataset)
- **conda** (Environment and dependency manager for installing STREAM from the bioconda channel)
- **ElPiGraph** (Core principal-graph algorithm underlying STREAM's trajectory inference) — sysbio-curie/ElPiGraph.R

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- The STREAM row reports correct detection of both synthetic bifurcations (2/2 against ground truth)
- All four correlation metrics (rank-based Pearson, distance-based Pearson, Spearman, Kendall's tau) are computed over every cell
- Each of the eleven methods appears with a complete metric row in benchmark_fig4b.csv — no missing values
- Metric layout matches Fig. 4b of the paper, allowing direct comparison of STREAM's relative ranking (best on distance-based Pearson, second-best on rank-based metrics)

## Limitations

- STREAM does not adapt to cyclic or disconnected graph topologies; the authors note 'further improvement is necessary to adapt STREAM to properly describing these more general structures'
- Scalability to millions of cells is not established for many trajectory inference methods including STREAM
- Benchmark rankings depend on default recommended settings per method, which may not be optimal for every tool
- STREAM does not require prior knowledge such as time points, start cell, or number of branching events, but competitors' behavior under the same assumptions may vary

## Evidence

- [abstract] Here we present STREAM, an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data.: "Here we present STREAM, an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic"
- [methods] STREAM is written using the class anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [methods] conda environment creation for STREAM: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [discussion] STREAM requires no prior knowledge of branching events: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [abstract] Synthetic and real benchmark testing claim: "We have tested STREAM on several synthetic and real datasets generated with different single-cell technologies."
