---
name: raycast-collision-detection
description: Use when implementing per-cell selection in a VR scatter/DR plot where
  datasets with tens of thousands of points (e.g., the >30k-cell mouse gastrulation
  GSE87038 coordinates) make per-point Unity colliders infeasible.
when_to_use_negative:
- The DR plot is sparse (far fewer than tens of thousands of points), where attaching
  native Unity colliders per point would be simpler and sufficient
- The goal is data preparation or backend statistics (e.g., DE gene identification
  in cellexalvrR) rather than in-VR interaction/collision logic
- Working outside the Unity/C# VR environment (e.g., selecting cells in a 2D desktop
  plotting tool)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CellexalVR
  role: The VR platform whose C# source (github.com/sonejilab/cellexalvr) hosts the
    engineered collision detection system and octree-based per-cell selection
  repo: https://github.com/sonejilab/cellexalvr
- name: VRTK
  role: Provides most of the VR interaction logic, including controller handling used
    by the raycast-based selection
  repo: https://github.com/ExtendRealityLtd/VRTK
- name: SteamVR / OpenVR
  role: Hardware communication layer for the HTC Vive / Valve Index controllers used
    to cast selection rays
  repo: https://github.com/ValveSoftware/openvr
- name: cellexalvrR
  role: R package that converts processed scRNA-seq objects (as_cellexalvrR, export2cellexalvr)
    into the CellexalVR input files containing the DR coordinates the octree indexes
  repo: https://github.com/sonejilab/cellexalvrR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:c7c7f373788da5078b5896f61431cb95771f57ba349d28e4052ca6c0035edb3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - SteamVR / OpenVR
  - VRTK
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# raycast-collision-detection

## Summary

Engineer a purpose-built collision detection system (octree spatial indexing plus controller raycast traversal) in CellexalVR's C# codebase so that every cell in dense dimension-reduction plots containing >30k points remains individually selectable without lag.

## When to use

Use when implementing per-cell selection in a VR scatter/DR plot where datasets with tens of thousands of points (e.g., the >30k-cell mouse gastrulation GSE87038 coordinates) make per-point Unity colliders infeasible.

## When NOT to use

- The DR plot is sparse (far fewer than tens of thousands of points), where attaching native Unity colliders per point would be simpler and sufficient
- The goal is data preparation or backend statistics (e.g., DE gene identification in cellexalvrR) rather than in-VR interaction/collision logic
- Working outside the Unity/C# VR environment (e.g., selecting cells in a 2D desktop plotting tool)

## Inputs

- CellexalVR input files of cell DR coordinates (produced from a processed Seurat/AnnData object via as_cellexalvrR and export2cellexalvr in the cellexalvrR R package), e.g. the mouse gastrulation dataset GSE87038 (Pijuan-Sala et al., 2019)
- Unity scene graph containing the DR plot points (Assets/CellexalVR_Main_Scene.unity) and controller raycast input from the VR headset/controllers ([VRTK]3.3 prefab)

## Outputs

- Octree spatial index over all cell points enabling per-point raycast hits
- Individually selectable cell points in the dense DR plot
- Skeleton sub-graph serialization of a selection (cells as nodes, neighborhood links as edges)

## How to apply

In the CellexalVR Unity/C# project (Unity 2021.3.0f1 for commit 1a29d79 or later; Unity 2019.1.8f1 for commits 9790228–5693e91), implement an octree spatial data structure that partitions the 3D space of cell DR coordinates and indexes every individual cell point instead of attaching Unity native colliders to each point. Wire controller raycast querying so it traverses the octree to detect which individual cell is hit; use VRTK for most of the interaction logic and SteamVR for hardware communication (controller script aliases live under the [VRTK_Scripts] prefab in Assets/Prefabs/Environment). When a freehand selection is made, serialize the cells reached by the selection as a skeleton sub-graph representation (cells as nodes, neighborhood links as edges), matching the Figure 3D depiction. Rationale: a completely new collision detection system was engineered specifically to accommodate a large number of points so each cell is individually selectable. Validate by running selection on the >30k-cell GSE87038 DR coordinates and confirming every cell is captured with no excessive lag.

## Related tools

- **CellexalVR** (The VR platform whose C# source (github.com/sonejilab/cellexalvr) hosts the engineered collision detection system and octree-based per-cell selection) — https://github.com/sonejilab/cellexalvr
- **VRTK** (Provides most of the VR interaction logic, including controller handling used by the raycast-based selection) — https://github.com/ExtendRealityLtd/VRTK
- **SteamVR / OpenVR** (Hardware communication layer for the HTC Vive / Valve Index controllers used to cast selection rays) — https://github.com/ValveSoftware/openvr
- **cellexalvrR** (R package that converts processed scRNA-seq objects (as_cellexalvrR, export2cellexalvr) into the CellexalVR input files containing the DR coordinates the octree indexes) — https://github.com/sonejilab/cellexalvrR

## Examples

```
git clone https://github.com/sonejilab/cellexalvr  # open in Unity 2021.3.0f1 (commit 1a29d79 or later) and inspect Assets/Scripts
```

## Evaluation signals

- Every one of the >30k points in the GSE87038 mouse gastrulation DR coordinates is individually hit by the controller raycast (no cell is unselectable)
- Freehand selection over dense plots completes with no excessive lag, consistent with the engineered collision detection system reported for large numbers of points
- The octree indexes every individual cell point and no per-point Unity native colliders are attached to graph points
- A freehand selection serializes to a skeleton sub-graph with cells as nodes and neighborhood links as edges, matching the Figure 3D depiction
- Selection works in the main scene built by SceneBuilder with VRTK controller aliases under the [VRTK_Scripts] prefab

## Limitations

- Backend calculations scale with the number of cells/groups selected and take longer for large datasets
- Requires a VR-ready workstation with a suitable graphics card (Steam recommends NVIDIA GTX1060 or higher) running Windows 10
- Only the HTC Vive and Valve Index were used during development; compatibility for other VR systems was not yet available
- No external validation or benchmark of CellexalVR against other VR tools is reported in this text

## Evidence

- [results] Per-cell selection in dense plots: "Each cell is individually selectable, and to make this possible we engineered a completely new collision detection system to accommodate a large number of points"
- [readme] VRTK and SteamVR roles in the Unity project: "We use [VRTK](https://vrtoolkit.readme.io/) for most of the interaction logic in CellexalVR and [SteamVR](https://steamcommunity.com/steamvr) for the hardware communication."
- [discussion] CellexalVR implementation language and source location: "The source code for the CellexalVR is available at https://github.com/sonejilab/cellexalvr which is coded in C#"
