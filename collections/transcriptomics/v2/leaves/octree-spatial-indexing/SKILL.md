---
name: octree-spatial-indexing
description: Use when you must make every cell individually selectable (e.g. freehand/controller
  selection) in VR dimension-reduction plots containing tens of thousands of scRNA-seq
  points, where attaching Unity native colliders per point is infeasible.
when_to_use_negative:
- Plot contains few enough points that Unity native colliders per cell are performant
  — the custom collision system targets large numbers of points
- You only need 2D flat-screen visualization without interactive per-cell selection;
  CellexalVR's collision/selection system is built for VR interaction
- Input data has not been converted to CellexalVR input files via cellexalvrR (no
  DR coordinates available to index)
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform in which the octree collision detection and per-cell selection
    system is implemented (C#/Unity); source at github.com/sonejilab/cellexalvr
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package that converts processed Seurat/AnnData objects (as_cellexalvrR,
    export2cellexalvr) into the CellexalVR input files containing the DR coordinates
    that the octree indexes
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: VRTK
  role: Virtual Reality Toolkit used for most of the interaction logic, including
    controller raycast interaction with the octree-indexed graphs
  repo: https://github.com/ExtendRealityLtd/VRTK
- name: OpenVR
  role: API/runtime for VR hardware communication underlying SteamVR device input
    during selection
  repo: https://github.com/ValveSoftware/openvr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:db8fdc1a00993e48114306eb83e7a6ecdd606cbbb8d3074a74d64040bdd449ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - OpenVR
  - R
  - VRTK
  - cellexalvrR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# octree-spatial-indexing

## Summary

An octree spatial data structure implemented in C# within CellexalVR that partitions the 3D space of cell dimension-reduction (DR) coordinates and indexes every individual cell point, enabling freehand per-cell selection in dense plots (>30k points) via a purpose-built collision detection system instead of Unity native colliders.

## When to use

Use when you must make every cell individually selectable (e.g. freehand/controller selection) in VR dimension-reduction plots containing tens of thousands of scRNA-seq points, where attaching Unity native colliders per point is infeasible.

## When NOT to use

- Plot contains few enough points that Unity native colliders per cell are performant — the custom collision system targets large numbers of points
- You only need 2D flat-screen visualization without interactive per-cell selection; CellexalVR's collision/selection system is built for VR interaction
- Input data has not been converted to CellexalVR input files via cellexalvrR (no DR coordinates available to index)

## Inputs

- Processed single-cell expression object (e.g. Seurat or AnnData) with DR coordinates (UMAP/tSNE/DDRTree)
- CellexalVR input files exported via cellexalvrR (export2cellexalvr)
- Unity project (CellexalVR main scene) with controller input via VRTK/SteamVR

## Outputs

- Octree spatial index over cell DR coordinates replacing per-point colliders
- Per-cell raycast hit detection supporting freehand selection of >30k points
- Serialized skeleton sub-graph of selected cells (cells as nodes, neighborhood links as edges)

## How to apply

Convert the processed scRNA-seq object (e.g. Seurat/AnnData) to CellexalVR input files with the cellexalvrR R package (as_cellexalvrR then export2cellexalvr), obtaining per-cell DR coordinates. In C# (CellexalVR is coded in C# on Unity, using VRTK for interaction logic and SteamVR/OpenVR for hardware), implement an octree that recursively partitions the 3D space of the DR coordinates, indexing each cell point rather than attaching Unity native colliders. Implement controller raycast querying that traverses the octree to detect which individual cell is hit, supporting freehand selection of all >30k points. Serialize the cells reached by a selection as a skeleton sub-graph (cells as nodes, neighborhood links as edges) for downstream reporting/analysis. Validate by running selection on the >30k-cell GSE87038 (Pijuan-Sala et al., 2019) gastrulation coordinates and confirming each cell remains individually selectable without excessive lag.

## Related tools

- **CellexalVR** (VR platform in which the octree collision detection and per-cell selection system is implemented (C#/Unity); source at github.com/sonejilab/cellexalvr) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package that converts processed Seurat/AnnData objects (as_cellexalvrR, export2cellexalvr) into the CellexalVR input files containing the DR coordinates that the octree indexes) — https://www.github.com/sonejilab/cellexalvrr
- **VRTK** (Virtual Reality Toolkit used for most of the interaction logic, including controller raycast interaction with the octree-indexed graphs) — https://github.com/ExtendRealityLtd/VRTK
- **OpenVR** (API/runtime for VR hardware communication underlying SteamVR device input during selection) — https://github.com/ValveSoftware/openvr

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- Every cell in a dense (>30k point) DR plot, e.g. GSE87038 mouse gastrulation coordinates, remains individually selectable with the selection tool
- No excessive lag during freehand selection, consistent with the engineered collision detection system built to accommodate a large number of points
- No Unity native colliders are attached per cell; the octree indexes each cell point instead
- Raycast traversal of the octree returns the correct individual cell hit for a given controller aim
- Serialized skeleton sub-graph (nodes = selected cells, edges = neighborhood links) matches the cells reached by a selection

## Limitations

- Requires a VR-ready workstation with a suitable graphics card (NVIDIA GTX1060 or higher recommended) running Windows 10; other OS support not stated
- Only HTC Vive and Valve Index were used during development; compatibility for other VR systems not yet available
- Backend calculations scale with the number of cells/groups selected and take longer for large datasets
- Unity version must match the checked-out commit (e.g. Unity 2021.3.0f1 for commit 1a29d79 or later)

## Evidence

- [results] Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points: "Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points"
- [discussion] Source code is C# on GitHub: "The source code for the CellexalVR is available at https://github.com/sonejilab/cellexalvr which is coded in C#"
- [results] cellexalvrR converts objects to CellexalVR input files: "converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [readme] README: VRTK handles interaction logic in CellexalVR: "We use [VRTK](https://vrtoolkit.readme.io/) for most of the interaction logic in CellexalVR and [SteamVR](https://steamcommunity.com/steamvr) for the hardware communication."
