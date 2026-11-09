---
name: large-point-cloud-selection-scaling
description: Use when building interactive selection over dense 3D/2D scatter plots
  of single-cell DR coordinates (>30k points) where per-cell colliders would be too
  slow and freehand per-cell capture is required.
when_to_use_negative:
- Point clouds are small enough that native per-object Unity colliders incur no performance
  cost — the custom system is unnecessary overhead
- The goal is aggregate (group/cluster-level) selection rather than per-cell freehand
  selection; CellexalVR's other selection modes may suffice
- The rendering environment is not Unity/VR (e.g. flat-screen 2D plotting libraries),
  where raycast-based octree traversal does not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: CellexalVR
  role: VR platform in which the custom collision detection system enables individual
    selection of every cell in dense DR plots; source code in C#
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package that converts processed Seurat/AnnData objects (as_cellexalvrR)
    and exports them (export2cellexalvr) into the input files whose DR coordinates
    populate the point cloud
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: VRTK
  role: Unity VR toolkit used in CellexalVR for most of the interaction logic, including
    the controller interactions driving selection
- name: OpenVR
  role: API/runtime underlying SteamVR hardware communication for the VR controllers
    used to perform selections
  repo: https://github.com/ValveSoftware/openvr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:5c9d2038a2a25ed2fd21d397be14482477f73bb6596679d492b33daad47116f3
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# large-point-cloud-selection-scaling

## Summary

A technique for making every cell in a dense VR dimension-reduction plot (tens of thousands of points) individually selectable by replacing per-object Unity colliders with a custom-engineered collision detection system built to accommodate a large number of points, as implemented in CellexalVR.

## When to use

Use when building interactive selection over dense 3D/2D scatter plots of single-cell DR coordinates (>30k points) where per-cell colliders would be too slow and freehand per-cell capture is required.

## When NOT to use

- Point clouds are small enough that native per-object Unity colliders incur no performance cost — the custom system is unnecessary overhead
- The goal is aggregate (group/cluster-level) selection rather than per-cell freehand selection; CellexalVR's other selection modes may suffice
- The rendering environment is not Unity/VR (e.g. flat-screen 2D plotting libraries), where raycast-based octree traversal does not apply

## Inputs

- CellexalVR input files generated from a processed scRNA-seq object (Seurat/AnnData) via cellexalvrR's as_cellexalvrR and export2cellexalvr, including DR coordinates (UMAP/tSNE/DDRTree) per cell
- Large single-cell datasets (>30k cells), e.g. mouse gastrulation GSE87038 or 37k-cell aging mouse brain GSE129788

## Outputs

- Individually selectable cell points in a VR dimension-reduction graph with a collision detection system that scales to tens of thousands of points
- Serialized selection of hit cells, usable for skeleton sub-graph representation (cells as nodes, neighborhood links as edges)

## How to apply

Instead of attaching Unity native colliders to each cell point, index all cell DR coordinates in a purpose-built spatial data structure (e.g. an octree implemented in C# within the Unity project) so that controller raycasts can traverse the structure and resolve which single cell is hit. Implement the selection tool so that freehand selections serialize the set of hit cells, enabling downstream operations such as building a skeleton sub-graph (cells as nodes, neighborhood links as edges). Data to visualize is prepared upstream with the cellexalvrR R package: convert Seurat or AnnData objects with as_cellexalvrR and export with export2cellexalvr into CellexalVR input files. Validate by running selection on a large dataset (e.g. the >30k-cell mouse gastrulation data, GEO GSE87038) and confirming each cell remains individually selectable without excessive lag.

## Related tools

- **CellexalVR** (VR platform in which the custom collision detection system enables individual selection of every cell in dense DR plots; source code in C#) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package that converts processed Seurat/AnnData objects (as_cellexalvrR) and exports them (export2cellexalvr) into the input files whose DR coordinates populate the point cloud) — https://www.github.com/sonejilab/cellexalvrr
- **VRTK** (Unity VR toolkit used in CellexalVR for most of the interaction logic, including the controller interactions driving selection)
- **OpenVR** (API/runtime underlying SteamVR hardware communication for the VR controllers used to perform selections) — https://github.com/ValveSoftware/openvr

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR); cellexal_obj <- as_cellexalvrR(seurat_obj); export2cellexalvr(cellexal_obj, "Data/data_set_1")
```

## Evaluation signals

- On a >30k-cell dataset (e.g. GSE87038 mouse gastrulation coordinates), every individual cell remains selectable freehand with the selection tool
- Selection over dense UMAP/tSNE graphs completes without excessive lag, indicating the collision detection system accommodates the large number of points
- Cells reached by a selection serialize correctly into the skeleton sub-graph representation (nodes = cells, edges = neighborhood links) matching the Figure 3D depiction
- No Unity native colliders are attached per cell; collision queries resolve through the engineered spatial-index system instead

## Limitations

- Backend calculations triggered by selections (e.g. identifying differentially expressed genes) take longer depending on how many cells/groups are selected
- Requires a VR-ready workstation with a suitable graphics card (NVIDIA GTX1060 or higher recommended) running Windows 10; only HTC Vive and Valve Index were used during development
- No external validation or benchmark of CellexalVR against other VR tools is reported

## Evidence

- [results] Engineered collision detection system for large point counts: "Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points"
- [results] Data preparation via cellexalvrR: [evidence span withheld — anchor status: not_found]
- [discussion] Backend scaling caveat: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
- [discussion] C# source availability: "The source code for the CellexalVR is available at https://github.com/sonejilab/cellexalvr which is coded in C#"
- [readme] Interaction logic built on VRTK: "We use [VRTK](https://vrtoolkit.readme.io/) for most of the interaction logic in CellexalVR and [SteamVR](https://steamcommunity.com/steamvr) for the hardware communication."
