---
name: subgraph-skeleton-representation
description: Use when a VR visualization must make every cell individually selectable
  in dimension-reduction plots containing >30k points, or when a selection must be
  captured as a cell/neighborhood sub-graph for downstream analysis.
when_to_use_negative:
- Datasets so small that native Unity colliders per point would not cause lag — the
  octree collision system is engineered specifically for large numbers of points
- When the goal is 2D flat-screen visualization of single-cell data rather than interactive
  VR selection (CellexalVR requires a VR-ready workstation running Windows 10)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform in which the octree-based collision detection and sub-graph serialization
    of selected cells are implemented in C#
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package converting processed scRNA-seq objects (via as_cellexalvrR and export2cellexalvr)
    into the CellexalVR input files consumed by the selection system
  repo: https://github.com/sonejilab/cellexalvrR
- name: VRTK
  role: VR toolkit used for most of the interaction logic, including controller-based
    selection in CellexalVR
  repo: https://github.com/ExtendRealityLtd/VRTK
- name: OpenVR/SteamVR
  role: Hardware communication layer for the VR headset and controllers driving the
    raycast selection
  repo: https://github.com/ValveSoftware/openvr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:e86d3f8844caf825c1858aa2ddb55c530952f77b3a5198a26e90977fac23d477
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - OpenVR/SteamVR
  - R
  - VRTK
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# subgraph-skeleton-representation

## Summary

Serializes cells reached by a freehand VR selection in CellexalVR as a skeleton sub-graph (cells as nodes, neighborhood links as edges), enabling per-cell selection in dense dimension-reduction plots of tens of thousands of points via an octree-based collision detection system rather than per-cell Unity colliders.

## When to use

Use when a VR visualization must make every cell individually selectable in dimension-reduction plots containing >30k points, or when a selection must be captured as a cell/neighborhood sub-graph for downstream analysis.

## When NOT to use

- Datasets so small that native Unity colliders per point would not cause lag — the octree collision system is engineered specifically for large numbers of points
- When the goal is 2D flat-screen visualization of single-cell data rather than interactive VR selection (CellexalVR requires a VR-ready workstation running Windows 10)

## Inputs

- CellexalVR input files derived from a processed scRNA-seq object (DR coordinates for each cell, e.g. UMAP/tSNE/DDRTree)
- cellexalvrR object produced by as_cellexalvrR from Seurat or AnnData objects
- Controller raycast interaction events from VR selection tool (HTC Vive / Valve Index via VRTK and SteamVR)

## Outputs

- Skeleton sub-graph representation of the selected cells (cells as nodes, neighborhood links as edges, per Figure 3D)
- Set of individually selected cells from a >30k-point dense dimension-reduction plot

## How to apply

First prepare scRNA-seq data (e.g., mouse gastrulation GSE87038) as CellexalVR input files by running as_cellexalvrR on a Seurat/AnnData object and exporting with export2cellexalvr from cellexalvrR. In the Unity/C# CellexalVR codebase, implement an octree spatial data structure that partitions the 3D space of cell DR coordinates, indexing every individual cell point instead of attaching Unity native colliders to each of the >30k points. Implement controller raycast querying that traverses the octree to detect which individual cell is hit during freehand selection with the selection tool. Serialize the cells reached by a selection as the skeleton sub-graph representation, with cells as nodes and neighborhood links as edges, corresponding to the Figure 3D depiction. The rationale is that a purpose-built collision detection system is needed because default per-object colliders cannot scale to large point counts while preserving per-cell selectability.

## Related tools

- **CellexalVR** (VR platform in which the octree-based collision detection and sub-graph serialization of selected cells are implemented in C#) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package converting processed scRNA-seq objects (via as_cellexalvrR and export2cellexalvr) into the CellexalVR input files consumed by the selection system) — https://github.com/sonejilab/cellexalvrR
- **VRTK** (VR toolkit used for most of the interaction logic, including controller-based selection in CellexalVR) — https://github.com/ExtendRealityLtd/VRTK
- **OpenVR/SteamVR** (Hardware communication layer for the VR headset and controllers driving the raycast selection) — https://github.com/ValveSoftware/openvr

## Examples

```
library(cellexalvrR); cellexal_obj <- export2cellexalvr(as_cellexalvrR(seurat_obj))  # produces CellexalVR input files consumed by the octree selection system
```

## Evaluation signals

- Run freehand selection on the >30k-cell mouse gastrulation (GSE87038) DR coordinates and confirm each individual cell remains selectable with no excessive lag
- Verify the octree indexes every cell point (no point left without an index) instead of Unity native colliders per cell
- Check that the serialized selection is a valid sub-graph: every reached cell appears as a node and neighborhood links appear as edges
- Confirm the selection result matches the Figure 3D-style skeleton sub-graph depiction for a known selection region
- Confirm the CellexalVR input files were correctly produced by as_cellexalvrR followed by export2cellexalvr before testing selection

## Limitations

- Requires a VR-ready workstation with a suitable graphics card (NVIDIA GTX1060 or higher recommended) running Windows 10; backend calculations take longer depending on how many cells/groups are selected
- Only HTC Vive and Valve Index were used during development; compatibility for other VR systems was not yet available
- The octree/selection approach is tied to the CellexalVR Unity codebase (C#) and its Data folder input-file layout

## Evidence

- [results] Per-cell selection via engineered collision detection: "Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points"
- [results] cellexalvrR converts data into CellexalVR input files: "the resulting object is converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [discussion] CellexalVR source code is C# on GitHub: "The source code for the CellexalVR is available at https://github.com/sonejilab/cellexalvr which is coded in C#"
- [readme] CellexalVR is a VR platform for single-cell data: "CellexalVR is a virtual reality platform for the visualisation and analysis of single-cell gene expression data."
