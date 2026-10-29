---
name: four-cluster-cell-partitioning
description: 'Use when preparing selected cells in a CellexalVR session for pseudotime
  trajectory calculation: slingshot analysis requires the selected cells to first
  be partitioned into exactly four clusters before pseudotime ordering.'
when_to_use_negative:
- The goal is a standalone clustering of the data rather than the specific four-cluster
  partition required as slingshot input for CellexalVR pseudotime
- The user needs pseudotime over the full dataset rather than a user-selected cell
  subset traced in a CellexalVR session
- The VR workstation prerequisites are absent (CellexalVR requires Windows 10, an
  NVIDIA GTX1060-or-better class GPU, R >= 4.0.0, and the cellexalvrR R package installed)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellexalVR
  role: VR front end where the user selects cells and triggers the four-cluster partitioning
    + slingshot pseudotime calculation
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R backend package providing the in-session calculation functions for partitioning,
    slingshot pseudotime, and heatmap generation
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: slingshot
  role: trajectory inference method run on the four-cluster partition to produce pseudotime
    values
- name: R
  role: runtime environment (>= 4.0.0) whose Rscript executable is called by the CellexalVR
    client for backend calculations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:2e407df6d769536346c588a0ab266dcd2a0481a35a2894e45415b6edcffccf48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - cellexalvrR
  - slingshot
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# four-cluster-cell-partitioning

## Summary

Partitions a selected set of single cells into four clusters as the mandatory preprocessing step for slingshot-based pseudotime trajectory inference in CellexalVR, after which ordered cells are split into ten colour groups for visualization.

## When to use

Use when preparing selected cells in a CellexalVR session for pseudotime trajectory calculation: slingshot analysis requires the selected cells to first be partitioned into exactly four clusters before pseudotime ordering.

## When NOT to use

- The goal is a standalone clustering of the data rather than the specific four-cluster partition required as slingshot input for CellexalVR pseudotime
- The user needs pseudotime over the full dataset rather than a user-selected cell subset traced in a CellexalVR session
- The VR workstation prerequisites are absent (CellexalVR requires Windows 10, an NVIDIA GTX1060-or-better class GPU, R >= 4.0.0, and the cellexalvrR R package installed)

## Inputs

- Selected cell set from a CellexalVR session (e.g. hand-traced path defining start and stop clusters)
- Dimensionality-reduction coordinates (e.g. DDRTree/UMAP/PHATE as exported 3D reductions)
- Normalized expression matrix for the selected cells (e.g. exported via export2cellexalvr from a Seurat-processed object)

## Outputs

- Four-cluster partition of the selected cells used for slingshot analysis
- Pseudotime ordering of cells with ten balanced colour-group labels
- Pseudotime heatmap of genes correlated to the inferred pseudotime (hierarchically clustered top marker genes)

## How to apply

Take the cells the user selected (e.g. via a hand-traced path defining start and stop clusters) and partition them into four clusters, matching the CellexalVR convention that the partitioning feeds directly into a slingshot analysis. Run slingshot on this four-cluster partition to compute the trajectory and order cells by the returned pseudotime values. Split the pseudotime-ordered cells into ten even colour groups as used by CellexalVR for group-wise visualization. The clustering and pseudotime calculations run in the cellexalvrR R backend, invoked from the Unity client via Rscript. For downstream heatmap validation, remember that differentially expressed genes for heatmaps are identified with a Wilcox test (implemented in C++) and hierarchical clustering is performed on the top N marker genes (default 250).

## Related tools

- **CellexalVR** (VR front end where the user selects cells and triggers the four-cluster partitioning + slingshot pseudotime calculation) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R backend package providing the in-session calculation functions for partitioning, slingshot pseudotime, and heatmap generation) — https://www.github.com/sonejilab/cellexalvrr
- **slingshot** (trajectory inference method run on the four-cluster partition to produce pseudotime values)
- **R** (runtime environment (>= 4.0.0) whose Rscript executable is called by the CellexalVR client for backend calculations)

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- Pseudotime ordering output contains exactly one row per analyzed cell
- Colour-group labels are balanced across ten even groups over the pseudotime-ordered cells
- The four-cluster partition is the input actually consumed by the slingshot call (trajectory endpoints correspond to user-defined start/stop clusters)
- Pseudotime heatmap exists and its gene rows are ordered by hierarchical clustering of the top marker genes (default N=250, Wilcox-test selected)
- An automatically generated HTML report records the in-session actions performed

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so large selected sets slow pseudotime and heatmap generation
- Requires a VR-ready Windows 10 workstation with a suitable GPU (NVIDIA GTX1060 or higher per Steam recommendation); only HTC Vive and Valve Index were used during development
- Each user in multi-user mode must hold a local copy of the data, as data files are not transmitted over RPCs

## Evidence

- [discussion] Four-cluster partition precedes slingshot: "Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis."
- [discussion] Ten colour groups after ordering: "Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis."
- [discussion] Heatmap gene selection parameters: "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [readme] Backend role of cellexalvrR: [evidence span withheld — anchor status: external_doc]
- [discussion] DE genes for heatmaps via Wilcox in C++: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
