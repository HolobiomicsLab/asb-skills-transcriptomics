---
name: rna-velocity-estimation-spliced-unspliced
description: Use when single-cell scRNA-seq data with spliced and unspliced count
  layers is available and the goal is to visualize predicted transcriptional trajectories
  as animated arrows/particles in a 3D embedding (e.g., PHATE/UMAP) inside CellexalVR.
when_to_use_negative:
- The input data lacks unspliced transcript counts, so spliced/unspliced velocity
  estimation cannot be performed
- The goal is static visualization of cell clusters rather than animated RNA velocity
  trajectories
- The target platform is not CellexalVR and cannot consume cellexalvrR-exported input
  files
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scvelo
  role: Python library for spliced/unspliced velocity estimation and velocity graph,
    producing per-cell destination coordinates
- name: Python
  role: Language used to run the scvelo velocity estimation workflow
- name: R
  role: Language required by cellexalvrR for exporting data; CellexalVR requires at
    least R 4.0.0 and Rscript path configuration
- name: cellexalvrR
  role: R package whose export2cellexalvr function converts the velocity table into
    CellexalVR input files
  repo: https://github.com/sonejilab/cellexalvrR
- name: CellexalVR
  role: VR platform that loads the .mds trajectory files via the InputReader and renders
    animated RNA velocity visualization
  repo: https://github.com/sonejilab/cellexalvr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:7a5a2d469142d09c009f39f06771442f1f93e8be915aaae48f735a68058e44fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - Python
  - R
  - cellexalvrR
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rna-velocity-estimation-spliced-unspliced

## Summary

Computes per-cell RNA velocity from spliced/unspliced transcript counts (scvelo velocity graph) and exports paired origin/destination 3D coordinates so CellexalVR can render animated RNA velocity trajectories, as demonstrated on the Liver Cell Atlas PHATE reduction.

## When to use

Use when single-cell scRNA-seq data with spliced and unspliced count layers is available and the goal is to visualize predicted transcriptional trajectories as animated arrows/particles in a 3D embedding (e.g., PHATE/UMAP) inside CellexalVR.

## When NOT to use

- The input data lacks unspliced transcript counts, so spliced/unspliced velocity estimation cannot be performed
- The goal is static visualization of cell clusters rather than animated RNA velocity trajectories
- The target platform is not CellexalVR and cannot consume cellexalvrR-exported input files

## Inputs

- scRNA-seq count matrix with spliced and unspliced layers (e.g., AnnData object)
- 3D dimension-reduction embedding of the cells (e.g., PHATE or UMAP coordinates)
- Per-cell velocity destination coordinates computed with scvelo (vx, vy, vz)
- Cell identifiers shared across the count matrix, DR embedding, and velocity vectors

## Outputs

- 7-column .mds trajectory files (cell ID + 3 position + 3 velocity values, one row per cell)
- Named subfolder of CellexalVR 'Data' folder containing the exported files, loadable by the InputReader

## How to apply

Load the scRNA-seq dataset together with its dimension-reduction embeddings (e.g., the Liver Cell Atlas PHATE and UMAP reductions), then compute per-cell velocity vectors with scvelo's spliced/unspliced velocity estimation and velocity graph in Python, yielding each cell's destination coordinates (vx, vy, vz) alongside its DR coordinates (x, y, z). Assemble a per-cell table pairing the cell identifier, the 3D embedding coordinates, and the velocity destination coordinates. Export this table with the cellexalvrR R package via export2cellexalvr as 7-column .mds trajectory files (cell ID + 3 position + 3 velocity values), because the article states that to enable RNA velocity visualization users must supply coordinates denoting each cell's destination alongside the coordinates of the DR method. Place the exported files in a named subfolder of the CellexalVR 'Data' folder (one folder per data set) so the InputReader loads them for animated RNA velocity visualization.

## Related tools

- **scvelo** (Python library for spliced/unspliced velocity estimation and velocity graph, producing per-cell destination coordinates)
- **Python** (Language used to run the scvelo velocity estimation workflow)
- **R** (Language required by cellexalvrR for exporting data; CellexalVR requires at least R 4.0.0 and Rscript path configuration)
- **cellexalvrR** (R package whose export2cellexalvr function converts the velocity table into CellexalVR input files) — https://github.com/sonejilab/cellexalvrR
- **CellexalVR** (VR platform that loads the .mds trajectory files via the InputReader and renders animated RNA velocity visualization) — https://github.com/sonejilab/cellexalvr

## Examples

```
devtools::install_github("sonejilab/cellexalvrR")
```

## Evaluation signals

- Each exported .mds file contains exactly 7 columns (cell ID + 3 position + 3 velocity values)
- Each .mds file has one row per cell and is accepted as CellexalVR input data by the InputReader
- CellexalVR renders animated RNA velocity arrows/particles when the data set folder is loaded (as shown with the Liver Cell Atlas PHATE reduction)
- The exported files reside in a named subfolder of the CellexalVR 'Data' folder, following the one-folder-per-dataset structure
- The RscriptFilePath in Config/config.xml points to a valid Rscript.exe so CellexalVR's back-end R functions run

## Limitations

- CellexalVR requires a VR-ready workstation with a suitable graphics card (NVIDIA GTX1060 or higher recommended) running Windows 10
- Only HTC Vive and Valve Index were used during development; compatibility for other VR systems will be added
- cellexalvrR is productively used under R 4.0.2 and 4.1.0, and CellexalVR requires at least R 4.0.0
- The article's velocity demonstration is validated only on the Liver Cell Atlas with PHATE/UMAP reductions; no other velocity benchmarks are reported

## Evidence

- [results] velocity destination coordinates required for export: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
- [results] velocity visualization demonstrated on Liver Cell Atlas: "A UMAP and PHATE reduction of the Liver Cell Atlas is shown where our animated RNA velocity visualization has been activated"
- [readme] Data folder structure for loading datasets: "Put your data inside a new folder with a sensible name in the `Data` folder."
- [readme] InputReader consumes cellexalvrR output: "reads the data from the CellexalVR R package and calls the appropriate functions to create graphs"
