---
name: branching-trajectory-visualization
description: Use when single-cell data (transcriptomic or chromatin-accessibility)
  is expected to contain branching developmental structure, and you need to learn,
  assign cells to, and visualize a branching trajectory graph with pseudotime.
when_to_use_negative:
- Data has cyclic or disconnected topology — the paper states STREAM needs further
  improvement to properly describe these more general structures
- A pre-computed feature table or trajectory is already available and only visualization
  is needed; simpler plotting tools suffice
- Datasets with millions of cells — the article notes many trajectory inference methods,
  including STREAM, do not yet scale to that size
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: End-to-end pipeline that learns branching trajectories, estimates pseudotime,
    and provides density-level trajectory visualization; written using the anndata
    class
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the elastic principal graph supporting
    the branching trajectory visualization
  repo: https://github.com/pinellolab/STREAM
- name: conda
  role: Environment manager used to install stream and stream_atac (Bioconda) for
    running the analysis
- name: STREAM_web
  role: Interactive web application offering the same trajectory visualization and
    exploration panels, runnable locally via Docker
  repo: https://github.com/pinellolab/STREAM_web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:c806c36ba7b1f816272aeedfe2d4cd45bc5afc5779695d12193026d264f1ed90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - STREAM_web
  - conda
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# branching-trajectory-visualization

## Summary

Visualize single-cell branching trajectories and pseudotime using STREAM's density-level trajectory plots built on the ElPiGraph principal-graph algorithm, applicable to both scRNA-seq and scATAC-seq (k-mer, peak, or motif feature) data.

## When to use

Use when single-cell data (transcriptomic or chromatin-accessibility) is expected to contain branching developmental structure, and you need to learn, assign cells to, and visualize a branching trajectory graph with pseudotime.

## When NOT to use

- Data has cyclic or disconnected topology — the paper states STREAM needs further improvement to properly describe these more general structures
- A pre-computed feature table or trajectory is already available and only visualization is needed; simpler plotting tools suffice
- Datasets with millions of cells — the article notes many trajectory inference methods, including STREAM, do not yet scale to that size

## Inputs

- Single-cell feature-count matrix (gene expression, scATAC-seq k-mer counts, peaks, or motif deviations) as an anndata object
- Optional cell annotations (labels/subpopulations) for coloring and composition visualization

## Outputs

- Branching trajectory graph with pseudotime and cell-to-branch assignment
- Density-level trajectory visualization figure showing subpopulation composition along branches
- Per-branch ranked feature lists (e.g. k-mer-to-transcription-factor tables)

## How to apply

Create a conda environment `env_stream` (python=3.7, stream=1.0, plus `stream_atac` for scATAC-seq) or pull the `pinellolab/stream` Docker image, then load the feature matrix (e.g. k-mer counts for scATAC-seq, held in an `anndata` object) into STREAM. Run dimensionality reduction (e.g. MLLE for k-mer features) and let ElPiGraph, the core algorithm behind STREAM, learn an elastic principal graph that captures the branching topology without needing time points, start cells, or the number of branching events specified. Cells are assigned to branches with pseudotime, and the resulting graph is rendered as STREAM's density-level trajectory visualization, where branch thickness reflects cell subpopulation composition. Downstream branch plots rank features (genes for scRNA-seq, k-mers mapped to transcription factors for scATAC-seq) along each branch. Correct application is judged by whether the learned graph recovers known hierarchy structure (e.g. HSC→MPP into erythroid, lymphoid, and myeloid branches) and whether branch-specific top features match expected regulators.

## Related tools

- **STREAM** (End-to-end pipeline that learns branching trajectories, estimates pseudotime, and provides density-level trajectory visualization; written using the anndata class) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm behind STREAM that learns the elastic principal graph supporting the branching trajectory visualization) — https://github.com/pinellolab/STREAM
- **conda** (Environment manager used to install stream and stream_atac (Bioconda) for running the analysis)
- **STREAM_web** (Interactive web application offering the same trajectory visualization and exploration panels, runnable locally via Docker) — https://github.com/pinellolab/STREAM_web

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- The learned ElPiGraph graph recovers the expected branching hierarchy (e.g. HSC through MPP into erythroid, lymphoid, and myeloid branches) without user-specified branch counts
- Pseudotime ordering increases monotonically from the root (stem/progenitor) state toward terminal branches
- Branch-thickness/density visualization reflects known subpopulation composition transitions along the trajectory
- Branch-ranked k-mers map to transcription factors consistent with lineage biology (e.g. GATA1 for erythroid, CEBPA for myeloid)

## Limitations

- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for sophisticated topologies
- STREAM is not yet adapted to cyclic or disconnected graph topologies
- Scaling to millions of cells is not established for STREAM
- Mapping new cells onto reference trajectories may not be optimal if the new cells have totally different fates from those in the reference structure

## Evidence

- [readme] STREAM is an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data.: "STREAM (**S**ingle-cell **T**rajectories **R**econstruction, **E**xploration **A**nd **M**apping) is an interactive pipeline capable of disentangling and visualizing complex branching trajectories"
- [methods] Create an environment named env_stream, install stream, jupyter, and activate it: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it"
- [discussion] Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [intro] STREAM also provides a mapping feature and a set of interactive tools to explore and visualize both cell type composition and relevant genes: "STREAM also provides a mapping feature and a set of interactive tools to explore and visualize both cell type composition and relevant genes"
- [discussion] Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
