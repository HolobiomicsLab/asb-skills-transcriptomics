---
name: seurat-object-downstream-analysis
description: Use when a preprocessed Seurat object from a plant single-cell experiment
  (e.g., Arabidopsis root scRNA-seq) is available and trajectory inference, cell-type
  annotation, deconvolution, or cross-species integration is the next analysis step.
when_to_use_negative:
- Input is raw count data that has not yet passed scPlant's basic processing (QCfun,
  RunSeurat clustering) — build the Seurat object first
- The goal is cell-type-specific gene regulatory network construction or cisTarget
  database generation (pySCENIC/MINI-EX), which use separate scPlant add-on modules
- The dataset is not plant-derived or lacks a suitable plant knowledgebase for annotation
  (e.g., no TAIR10/EnsemblPlants/Phytozome reference applicable)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Framework providing RunMonocle2, RunMonocle3, RunSlingshot, RunCytoTRACE,
    RunSeurat, QCfun, and readData functions that operate on Seurat objects
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying single-cell object and processing methods integrated by scPlant
    for basic data processing
- name: R
  role: Implementation language; scPlant functions are called in R with save_plot
    options
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:9ef43533e97d615d13984c393bcfbb3c551aa94649a07a5b432e0774f6ccb676
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
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# seurat-object-downstream-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run scPlant's advanced analysis modules (RunMonocle2, RunMonocle3, RunSlingshot, RunCytoTRACE and related functions) on a preprocessed Seurat object to infer trajectories and other downstream structure in plant scRNA-seq data, as demonstrated on the Arabidopsis thaliana root atlas (Zhang et al., 2019).

## When to use

Use when a preprocessed Seurat object from a plant single-cell experiment (e.g., Arabidopsis root scRNA-seq) is available and trajectory inference, cell-type annotation, deconvolution, or cross-species integration is the next analysis step.

## When NOT to use

- Input is raw count data that has not yet passed scPlant's basic processing (QCfun, RunSeurat clustering) — build the Seurat object first
- The goal is cell-type-specific gene regulatory network construction or cisTarget database generation (pySCENIC/MINI-EX), which use separate scPlant add-on modules
- The dataset is not plant-derived or lacks a suitable plant knowledgebase for annotation (e.g., no TAIR10/EnsemblPlants/Phytozome reference applicable)

## Inputs

- Preprocessed Seurat object (SeuratObj) from plant scRNA-seq data, e.g., expression matrix loaded via readData(data = example_Ath, type = 'expMatrix')
- Cell-type labels (e.g., meristem root cells) used to define the trajectory root
- Auxiliary deposit from figshare (Auxiliary data and codes for scPlant) containing the preprocessed Arabidopsis root Seurat object

## Outputs

- Trajectory plots and pseudotime results saved via save_plot = TRUE into per-tool directories (e.g., ./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput)
- Per-cell pseudotime values from Monocle2, Monocle3, Slingshot, and CytoTRACE for cross-tool correlation comparison
- Correlation figure comparing pseudotimes across the four tools (cf. Figure 4E-4F)

## How to apply

Start from a Seurat object produced by scPlant's readData/QCfun/RunSeurat pipeline (e.g., resolution = 0.3 clustering) or load the preprocessed Arabidopsis thaliana root object (Zhang et al., 2019, PRJNA517021) from the figshare auxiliary deposit. For trajectory inference, call scPlant's RunMonocle2 (max_components = 2, save_plot = TRUE), RunMonocle3 (save_plot = TRUE), and RunSlingshot (save_plot = TRUE), rooting trajectories at meristem root cells, plus RunCytoTRACE (save_plot = TRUE) for developmental potential; store plots in dedicated output directories (e.g., ./output/RunMonocle2). Compare predicted pseudotimes across tools — CytoTRACE, Monocle2, Monocle3, and Slingshot — since orderings can differ while pseudotimes still show overall positive correlation. Validation combines existence of plot files in each output directory with qualitative inspection of the pseudotime correlation figure.

## Related tools

- **scPlant** (Framework providing RunMonocle2, RunMonocle3, RunSlingshot, RunCytoTRACE, RunSeurat, QCfun, and readData functions that operate on Seurat objects) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying single-cell object and processing methods integrated by scPlant for basic data processing)
- **R** (Implementation language; scPlant functions are called in R with save_plot options)

## Examples

```
result <- RunMonocle2(SeuratObj, save_plot = TRUE); result <- RunMonocle3(SeuratObj, save_plot = TRUE); result <- RunSlingshot(SeuratObj, save_plot = TRUE)
```

## Evaluation signals

- Plot files exist in all four output directories (./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput)
- CytoTRACE reproduction confirms its predicted (incorrect) xylem-cell root for the Arabidopsis root data
- Pseudotimes from Monocle2, Monocle3, Slingshot, and CytoTRACE show the overall positive correlation reported for Figure 4E-4F (qualitative check on the correlation figure)
- Trajectories rooted at meristem cells run to completion without errors when max_components = 2 is used for Monocle2

## Limitations

- Cell ordering along inferred trajectories differs among CytoTRACE, Monocle2, and Monocle3; only overall positive correlation of pseudotimes is expected, not identical orderings
- CytoTRACE predicted an incorrect xylem-cell root on the Arabidopsis root atlas, so its developmental-potential root assignment should be treated with caution
- Validation of the correlation is qualitative (visual check on the figure) rather than a quantitative threshold in the reproduction workflow

## Evidence
