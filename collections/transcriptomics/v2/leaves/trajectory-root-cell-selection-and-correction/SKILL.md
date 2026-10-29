---
name: trajectory-root-cell-selection-and-correction
description: Use when inferring developmental pseudotime on a plant scRNA-seq atlas
  with scPlant (RunMonocle2, RunMonocle3, RunSlingshot, RunCytoTRACE) and a biologically
  plausible origin — such as meristem cells — must be chosen or validated against
  a tool's automatic root prediction.
when_to_use_negative:
- The dataset has no identifiable progenitor/stem-cell population (e.g., meristem
  cells) to serve as a biologically justified trajectory root
- The goal is trajectory inference on non-plant single-cell data outside the scPlant
  framework
- Only one pseudotime tool will be run, so the cross-tool root consistency check (Figure
  4E-4F) cannot be performed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPlant
  role: Framework providing RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE
    wrappers for trajectory inference with a user-specified root cell type
  repo: https://github.com/compbioNJU/scPlant
- name: R
  role: Language in which the scPlant trajectory-inference workflow is executed
- name: Seurat
  role: Produces the preprocessed Seurat object that is the input to scPlant's trajectory-inference
    functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:9f439070ea697aaddabb45af3d92ad42b19b2c2e4f8fcb41623c3d9c5f989daf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - R
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# trajectory-root-cell-selection-and-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select a biologically justified root cell type (e.g., meristem root cells in the Arabidopsis thaliana root atlas of Zhang et al., 2019) when running pseudotime inference in scPlant, and sanity-check the resulting trajectory root — as demonstrated by CytoTRACE incorrectly rooting at xylem cells despite overall positive pseudotime correlation with Monocle2, Monocle3, and Slingshot.

## When to use

Use when inferring developmental pseudotime on a plant scRNA-seq atlas with scPlant (RunMonocle2, RunMonocle3, RunSlingshot, RunCytoTRACE) and a biologically plausible origin — such as meristem cells — must be chosen or validated against a tool's automatic root prediction.

## When NOT to use

- The dataset has no identifiable progenitor/stem-cell population (e.g., meristem cells) to serve as a biologically justified trajectory root
- The goal is trajectory inference on non-plant single-cell data outside the scPlant framework
- Only one pseudotime tool will be run, so the cross-tool root consistency check (Figure 4E-4F) cannot be performed

## Inputs

- Preprocessed Seurat object of a plant scRNA-seq dataset (e.g., Arabidopsis thaliana root atlas, Zhang et al., 2019, PRJNA517021, from the figshare auxiliary deposit for scPlant)
- Cell-type annotations identifying the developmentally earliest population (meristem root cells)
- Preprocessed Seurat object passed to scPlant's RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE

## Outputs

- Per-cell pseudotime values from Monocle2, Monocle3, Slingshot, and CytoTRACE rooted at meristem cells
- Trajectory plots saved in ./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, and ./output/CytoTRACEoutput (save_plot = TRUE)
- Cross-tool pseudotime correlation assessment reproducing the positive correlation of Figure 4E-4F

## How to apply

Load the preprocessed Seurat object (e.g., the Arabidopsis thaliana root dataset, Zhang et al., 2019, PRJNA517021, from the figshare auxiliary deposit) and identify the developmentally youngest cell population — here meristem root cells — to designate as the trajectory root. Run scPlant's RunMonocle2 (max_components = 2, save_plot = TRUE) and RunMonocle3 with the root set to meristem cells, storing plots in ./output/RunMonocle2 and ./output/RunMonocle3, and run RunSlingshot with save_plot = TRUE. Also run RunCytoTRACE with save_plot = TRUE and explicitly verify its automatically predicted root: on the Arabidopsis root data CytoTRACE roots the trajectory incorrectly at xylem cells, so its output requires biological correction/interpretation rather than blind acceptance. Finally, compare pseudotimes across the four tools; although cell orderings differ, the pseudotimes show an overall positive correlation (Figure 4F), which serves as the cross-tool consistency check.

## Related tools

- **scPlant** (Framework providing RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE wrappers for trajectory inference with a user-specified root cell type) — https://github.com/compbioNJU/scPlant
- **R** (Language in which the scPlant trajectory-inference workflow is executed)
- **Seurat** (Produces the preprocessed Seurat object that is the input to scPlant's trajectory-inference functions)

## Examples

```
result <- RunMonocle2(SeuratObj, save_plot = TRUE); result <- RunMonocle3(SeuratObj, save_plot = TRUE); result <- RunSlingshot(SeuratObj, save_plot = TRUE)
```

## Evaluation signals

- Plot files exist in all four output directories (./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput)
- Pseudotimes from Monocle2, Monocle3, Slingshot, and CytoTRACE show the overall positive correlation reported for Figure 4E-4F (qualitative check on the correlation figure)
- CytoTRACE's predicted trajectory root on the Arabidopsis root data is confirmed to be the (incorrect) xylem-cell root, documenting the need for manual correction
- Monocle2 and Monocle3 trajectories are rooted at meristem cells as specified, not at an automatically chosen root
- Cell orderings may differ among CytoTRACE, Monocle2, and Monocle3, but per-cell pseudotime values remain positively correlated across tools

## Limitations

- CytoTRACE can predict an incorrect trajectory root (xylem cells on the Arabidopsis root atlas), requiring biological correction/interpretation
- Cell orderings along inferred trajectories differ among CytoTRACE, Monocle2, and Monocle3, so individual cell-level orderings are not directly comparable; only overall pseudotime correlation is expected
- The root must be specified biologically (meristem cells); automatic rooting should not be trusted without verification

## Evidence
