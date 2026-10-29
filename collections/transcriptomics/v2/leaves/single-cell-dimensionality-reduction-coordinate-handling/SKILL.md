---
name: single-cell-dimensionality-reduction-coordinate-handling
description: Use when dense dimension-reduction plots (UMAP/tSNE/DDRTree of scRNA-seq
  data, e.g. >30k cells) must support freehand per-cell selection or per-point raycast
  hit detection in Unity, and attaching native colliders to every point is infeasible.
when_to_use_negative:
- Datasets with few points, where attaching standard Unity native colliders per cell
  is sufficient
- Data lacking 3D dimension-reduction coordinates (the octree partitions the 3D space
  of cell DR coordinates)
- Non-Windows environments or non-HTC Vive/Valve Index hardware — the platform requires
  Windows 10 and compatibility for other VR systems was not yet added
edam_operation: http://edamontology.org/operation_0337
tools:
- name: CellexalVR
  role: Unity/C# VR platform in which the octree collision detection system is engineered
    and the selection is serialized as a skeleton sub-graph
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package that converts Seurat/AnnData objects with DR coordinates into CellexalVR
    input files (as_cellexalvrR, export2cellexalvr)
  repo: https://github.com/sonejilab/cellexalvrR
- name: VRTK
  role: Provides the controller interaction logic through which the raycast selection
    queries are issued
  repo: https://vrtoolkit.readme.io/
- name: OpenVR/SteamVR
  role: Hardware communication layer for the HTC Vive/Valve Index controllers used
    for selection
  repo: https://github.com/ValveSoftware/openvr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:6aa9790d945950dabea4a044836f5ab91addb1008afe771436cf6eaca4ccf4ca
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-dimensionality-reduction-coordinate-handling

## Summary

A purpose-built C# collision detection system (octree spatial partitioning over dimension-reduction coordinates) that lets CellexalVR make every cell in dense 3D scatter plots — tens of thousands of points — individually selectable with the freehand selection tool, instead of attaching a Unity native collider to each cell.

## When to use

Use when dense dimension-reduction plots (UMAP/tSNE/DDRTree of scRNA-seq data, e.g. >30k cells) must support freehand per-cell selection or per-point raycast hit detection in Unity, and attaching native colliders to every point is infeasible.

## When NOT to use

- Datasets with few points, where attaching standard Unity native colliders per cell is sufficient
- Data lacking 3D dimension-reduction coordinates (the octree partitions the 3D space of cell DR coordinates)
- Non-Windows environments or non-HTC Vive/Valve Index hardware — the platform requires Windows 10 and compatibility for other VR systems was not yet added

## Inputs

- Processed scRNA-seq object with dimension-reduction (DR) coordinates (Seurat or AnnData)
- CellexalVR input files produced by cellexalvrR's export2cellexalvr (placed in the Data folder)
- 3D DR coordinate table of cell positions (e.g. UMAP/tSNE/DDRTree coordinates)

## Outputs

- Octree spatial index over cell DR coordinates for per-cell collision detection
- Individually selectable 3D graph of cells supporting freehand selection of all points
- Skeleton sub-graph of a selection (cells as nodes, neighborhood links as edges)

## How to apply

First convert the processed scRNA-seq object (Seurat or AnnData) with its DR coordinates into CellexalVR input files using the cellexalvrR R package via as_cellexalvrR followed by export2cellexalvr. In the C#/Unity codebase, build an octree that partitions the 3D space of cell DR coordinates and indexes every individual cell point, replacing per-point Unity colliders. Implement controller raycast querying that traverses the octree to determine which individual cell is hit, so all >30k points remain selectable freehand. When a selection is made, serialize the selected cells as a skeleton sub-graph representation — cells as nodes, neighborhood links as edges — which corresponds to the Figure 3D depiction. Validate by running selection on the >30k-cell mouse gastrulation GSE87038 coordinates and confirming each cell is captured individually with no excessive lag.

## Related tools

- **CellexalVR** (Unity/C# VR platform in which the octree collision detection system is engineered and the selection is serialized as a skeleton sub-graph) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package that converts Seurat/AnnData objects with DR coordinates into CellexalVR input files (as_cellexalvrR, export2cellexalvr)) — https://github.com/sonejilab/cellexalvrR
- **VRTK** (Provides the controller interaction logic through which the raycast selection queries are issued) — https://vrtoolkit.readme.io/
- **OpenVR/SteamVR** (Hardware communication layer for the HTC Vive/Valve Index controllers used for selection) — https://github.com/ValveSoftware/openvr

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrr")
```

## Evaluation signals

- On the >30k-cell GSE87038 mouse gastrulation DR coordinates, every individual cell remains selectable freehand with no excessive lag
- The implementation indexes cell points in an octree rather than attaching Unity native colliders to each cell
- A freehand selection serializes the reached cells as a skeleton sub-graph (cells as nodes, neighborhood links as edges)
- CellexalVR finds the data: folders named per dataset exist under the Data folder and RscriptFilePath in Config/config.xml points to the installed Rscript.exe

## Limitations

- Backend calculations triggered by selections (e.g. differentially expressed genes for heatmaps) take longer as the number of cells/groups selected grows
- Requires a VR-ready Windows 10 workstation with a suitable graphics card (Steam recommends NVIDIA GTX1060 or higher); only HTC Vive and Valve Index were used during development
- In multi-user mode each participant must hold a local copy of the data; data files are not transmitted over RPCs

## Evidence

- [results] collision detection system enabling individual selection: "Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points"
- [results] cellexalvrR produces CellexalVR input files: "converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [discussion] CellexalVR source code is C#: "The source code for the CellexalVR is available at https://github.com/sonejilab/cellexalvr which is coded in C#"
- [readme] VRTK provides interaction logic in the Unity project: "We use [VRTK](https://vrtoolkit.readme.io/) for most of the interaction logic in CellexalVR and [SteamVR](https://steamcommunity.com/steamvr) for the hardware communication."
- [readme] Data folder layout for CellexalVR input files: "Create a folder called `Data` where CellexalVR is installed.
* Put your data inside a new folder with a sensible name in the `Data` folder."
