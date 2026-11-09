---
name: single-cell-trajectory-file-formatting
description: Use when a scRNA-seq dataset has per-cell RNA velocity vectors (spliced/unspliced-derived)
  that must be exported alongside 3D DR coordinates so CellexalVR can visualize trajectories
  as animated arrows/particles.
when_to_use_negative:
- The goal is a standard gene expression graph export without RNA velocity — plain
  DR coordinates suffice and no destination-coordinate table is needed
- Data is not destined for CellexalVR — other visualization platforms use different
  input file formats
- Only 2D embeddings are available — velocity visualization in CellexalVR expects
  3D (x, y, z) coordinates
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scvelo
  role: Python tool used to compute per-cell RNA velocity vectors (spliced/unspliced
    estimation and velocity graph) that yield each cell's destination coordinates
- name: cellexalvrR
  role: R package whose export2cellexalvr function writes the per-cell position/velocity
    table as CellexalVR .mds trajectory files
  repo: https://github.com/sonejilab/cellexalvrR
- name: CellexalVR
  role: VR platform whose InputReader loads the .mds files from the Data folder and
    renders animated RNA velocity arrows/particles
  repo: https://github.com/sonejilab/cellexalvr
- name: R
  role: Runtime (R 4.0+) in which cellexalvrR is installed and the export is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:a0b737c38940bac9ee0f28ffef250e4f3a46b80caa1f339a85d6e6f702faadca
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# single-cell-trajectory-file-formatting

## Summary

Formats per-cell origin/destination coordinate pairs (e.g. from scvelo RNA velocity) into 7-column .mds trajectory files via the cellexalvrR export pipeline so CellexalVR can render animated RNA velocity trajectories.

## When to use

Use when a scRNA-seq dataset has per-cell RNA velocity vectors (spliced/unspliced-derived) that must be exported alongside 3D DR coordinates so CellexalVR can visualize trajectories as animated arrows/particles.

## When NOT to use

- The goal is a standard gene expression graph export without RNA velocity — plain DR coordinates suffice and no destination-coordinate table is needed
- Data is not destined for CellexalVR — other visualization platforms use different input file formats
- Only 2D embeddings are available — velocity visualization in CellexalVR expects 3D (x, y, z) coordinates

## Inputs

- scRNA-seq count matrix with spliced and unspliced layers (for scvelo velocity estimation)
- Per-cell 3D dimension-reduction embedding coordinates (e.g. PHATE or UMAP x, y, z)
- Per-cell velocity destination coordinates (vx, vy, vz) derived from scvelo
- Cell identifier table pairing cell ID with position and velocity values

## Outputs

- 7-column .mds trajectory files (cell ID + 3 position + 3 velocity values, one row per cell)
- A named subfolder of CellexalVR input files under the CellexalVR 'Data' folder loadable by the InputReader

## How to apply

Compute per-cell RNA velocity with scvelo in Python (spliced/unspliced velocity estimation and velocity graph) to obtain, for each cell, a destination coordinate alongside its dimension-reduction (e.g. PHATE or UMAP) coordinates. Assemble a per-cell table pairing the cell identifier, the 3D embedding coordinates (x, y, z), and the velocity destination coordinates (vx, vy, vz). Convert the Seurat/AnnData object with as_cellexalvrR if needed, then export with export2cellexalvr from the cellexalvrR package (R 4.0+) to produce .mds trajectory files with exactly 7 columns (cell ID + 3 position + 3 velocity values), one row per cell. Place the exported files in a named subfolder of the CellexalVR 'Data' folder so the InputReader loads them for animated RNA velocity visualization. Validation is straightforward: each .mds file must contain exactly 7 columns with one row per cell and be accepted by CellexalVR's InputReader.

## Related tools

- **scvelo** (Python tool used to compute per-cell RNA velocity vectors (spliced/unspliced estimation and velocity graph) that yield each cell's destination coordinates)
- **cellexalvrR** (R package whose export2cellexalvr function writes the per-cell position/velocity table as CellexalVR .mds trajectory files) — https://github.com/sonejilab/cellexalvrR
- **CellexalVR** (VR platform whose InputReader loads the .mds files from the Data folder and renders animated RNA velocity arrows/particles) — https://github.com/sonejilab/cellexalvr
- **R** (Runtime (R 4.0+) in which cellexalvrR is installed and the export is executed)

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrR")
```

## Evaluation signals

- Each exported .mds file contains exactly 7 columns (cell ID + 3 position + 3 velocity values) with one row per cell
- The number of rows in the trajectory file equals the number of cells in the DR embedding
- Files are accepted as CellexalVR input data by the InputReader without errors
- Exported files reside in a named subfolder of the CellexalVR 'Data' folder matching the expected folder structure (Config/config.xml present, RscriptFilePath set)
- CellexalVR renders animated velocity arrows/particles for the dataset (as demonstrated on the Liver Cell Atlas PHATE reduction)

## Limitations

- Requires R 4.0.0 or later (productively used under R 4.0.2 and 4.1.0) and the devtools package to install cellexalvrR from GitHub
- Destination coordinates must be computed upstream in Python with scvelo; cellexalvrR itself does not compute RNA velocity
- Data files must be present locally in the CellexalVR 'Data' folder; they are not transmitted over RPCs in multi-user sessions, so each user needs a local copy
- Velocity destination coordinates must be supplied by the user; without them the export produces standard graphs without animated trajectories

## Evidence

- [results] To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used: "To enable RNA velocity visualization users should supply coordinates denoting the destination of each cell alongside the coordinates of the DR method used"
- [results] converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR: "converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [results] A UMAP and PHATE reduction of the Liver Cell Atlas is shown where our animated RNA velocity visualization has been activated: "A UMAP and PHATE reduction of the Liver Cell Atlas is shown where our animated RNA velocity visualization has been activated"
- [readme] Put your data inside a new folder with a sensible name in the `Data` folder.: "Put your data inside a new folder with a sensible name in the `Data` folder."
- [readme] cellexalvrR is productively used under R 4.0.2 and 4.1.0.: [evidence span withheld — anchor status: external_doc]
