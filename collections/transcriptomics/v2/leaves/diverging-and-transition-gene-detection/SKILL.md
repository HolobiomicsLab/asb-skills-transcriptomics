---
name: diverging-and-transition-gene-detection
description: Use when a STREAM elastic principal graph has been fitted to scRNA-seq
  data (e.g., AnnData expression matrix) and you need to identify which genes diverge
  at branch points and which transition along branches.
when_to_use_negative:
- The trajectory topology is cyclic or disconnected — the paper notes further improvement
  is necessary to adapt STREAM to these more general structures
- The dataset has millions of cells — the article states many trajectory inference
  methods, including STREAM, don't scale to such datasets
- You need to map new cells onto a reference trajectory where the new cells have totally
  different fates — the mapping procedure may not be optimal in that case
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Interactive pipeline that reconstructs the branching trajectory and detects
    diverging and transition genes along its branches
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm underlying STREAM's branch inference,
    on which marker detection is performed
  repo: https://github.com/Albluca/ElPiGraph.R
- name: anndata
  role: Data class STREAM is written with, holding the scRNA-seq expression matrix
    fed into marker detection
  repo: http://anndata.rtfd.io
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:fbbb09408c46bae57dc57c862e4e7515b23d5f99f3e3fb40eae3d979dcfe35ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# diverging-and-transition-gene-detection

## Summary

Detects diverging genes (fate-specific markers such as erythroid Car2 and Epx) and transition genes (e.g., Coro1a, Blvrb) along branches of a STREAM/ElPiGraph reconstructed single-cell trajectory, enabling biological interpretation of branching events.

## When to use

Use when a STREAM elastic principal graph has been fitted to scRNA-seq data (e.g., AnnData expression matrix) and you need to identify which genes diverge at branch points and which transition along branches.

## When NOT to use

- The trajectory topology is cyclic or disconnected — the paper notes further improvement is necessary to adapt STREAM to these more general structures
- The dataset has millions of cells — the article states many trajectory inference methods, including STREAM, don't scale to such datasets
- You need to map new cells onto a reference trajectory where the new cells have totally different fates — the mapping procedure may not be optimal in that case

## Inputs

- AnnData (.h5ad) scRNA-seq expression matrix as used by STREAM (e.g., Nestorowa et al. mouse HSC data, 1656 sorted cells)
- STREAM-selected informative genes and low-dimensional embedding computed before tree learning
- Fitted ElPiGraph branching trajectory (learned without prior start cell or number of branch events)
- Cell type labels for lymphoid, myeloid, and erythroid fates (for rendering)

## Outputs

- Detected diverging genes per branch (fate-specific markers, e.g., Car2, Epx on the erythroid branch)
- Detected transition genes along branches (e.g., Coro1a, Blvrb)
- Branching trajectory visualization with cell type labels, e.g., stream_nestorowa_trajectory.png
- Marker output tables consistent with Figure 1 of the STREAM paper

## How to apply

Fit the STREAM branching trajectory first — the ElPiGraph elastic principal graph algorithm reconstructs the tree from an AnnData expression matrix without supplying time points, a start cell, or a preset number of branching events. Once branches are inferred, run STREAM's marker-detection step along the reconstructed branches to obtain diverging genes (genes whose expression separates at bifurcations, e.g., expected erythroid markers Car2 and Epx) and transition genes (genes changing gradually along a branch, e.g., Coro1a and Blvrb). Follow the STREAM scRNA-seq bifurcation tutorial notebook workflow, rendering the branching trajectory with cell type labels corresponding to lymphoid, myeloid, and erythroid fates and saving the figure (e.g., stream_nestorowa_trajectory.png). Validate by checking that the reconstructed tree shows the reported bifurcations and that expected fate markers appear in the detected marker output, as in Figure 1 of the STREAM paper on the Nestorowa et al. mouse hematopoiesis dataset (1656 sorted cells).

## Related tools

- **STREAM** (Interactive pipeline that reconstructs the branching trajectory and detects diverging and transition genes along its branches) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm underlying STREAM's branch inference, on which marker detection is performed) — https://github.com/Albluca/ElPiGraph.R
- **anndata** (Data class STREAM is written with, holding the scRNA-seq expression matrix fed into marker detection) — http://anndata.rtfd.io

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- The reconstructed tree exhibits the reported lymphoid/myeloid/erythroid bifurcations with multipotent progenitors placed before the first bifurcation (Fig. 1b-d)
- Expected erythroid markers Car2 and Epx appear in the detected diverging gene output on the erythroid branch
- Transition markers Coro1a and Blvrb are detected along the reconstructed branches
- The tree was fit without supplying time points, a start cell, or a preset number of branching events
- The workflow matches the STREAM scRNA-seq bifurcation tutorial and produces a labeled figure (e.g., stream_nestorowa_trajectory.png)

## Limitations

- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for such sophisticated topologies
- STREAM is not yet adapted to properly describe cyclic or disconnected graph topologies
- Scalability to millions of cells is not established for STREAM

## Evidence

- [discussion] STREAM does not require prior knowledge such as time points, start cell, or number of branching events: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [methods] STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] STREAM is an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data.: [evidence span withheld — anchor status: not_found]
- [discussion] Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
