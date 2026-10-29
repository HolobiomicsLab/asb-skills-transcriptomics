---
name: cross-tool-pseudotime-correlation-analysis
description: Use when you need to benchmark or validate trajectory-inference consistency
  across multiple pseudotime tools on a preprocessed plant scRNA-seq Seurat object
  with known root-state cell types (e.g., meristem).
when_to_use_negative:
- The input is a raw count matrix with no preprocessed Seurat object or cell-type
  annotations to root the trajectory at meristem cells
- The goal is a single-tool trajectory analysis rather than cross-tool consistency
  benchmarking of pseudotime predictions
- The dataset is bulk transcriptomic data rather than single-cell data, since trajectory
  inference in scPlant operates on single-cell objects
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Framework providing the RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE
    wrappers used to infer and compare pseudotime trajectories
  repo: https://github.com/compbioNJU/scPlant
- name: R
  role: Implementation language in which the scPlant trajectory-inference functions
    are executed
- name: Seurat
  role: Provides the preprocessed Seurat object format that is the input to each scPlant
    trajectory-inference wrapper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:9846a611cdd969769488744c0010163fbc11de1ade111bdf495ae982ed1d1c51
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-tool-pseudotime-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and compare developmental pseudotime trajectories on the Arabidopsis root scRNA-seq atlas (Zhang et al., 2019, PRJNA517021) using scPlant's four trajectory-inference wrappers (CytoTRACE, Slingshot, Monocle2, Monocle3), then verify that the predicted pseudotimes show the overall positive correlation reported in Figure 4E-4F despite differing cell orderings.

## When to use

Use when you need to benchmark or validate trajectory-inference consistency across multiple pseudotime tools on a preprocessed plant scRNA-seq Seurat object with known root-state cell types (e.g., meristem).

## When NOT to use

- The input is a raw count matrix with no preprocessed Seurat object or cell-type annotations to root the trajectory at meristem cells
- The goal is a single-tool trajectory analysis rather than cross-tool consistency benchmarking of pseudotime predictions
- The dataset is bulk transcriptomic data rather than single-cell data, since trajectory inference in scPlant operates on single-cell objects

## Inputs

- Preprocessed Arabidopsis thaliana root Seurat object (Zhang et al., 2019, PRJNA517021) from the scPlant figshare auxiliary deposit
- Cell-type annotations with meristem root cells identified as the trajectory root

## Outputs

- Trajectory plots saved in ./output/RunMonocle2
- Trajectory plots saved in ./output/RunMonocle3
- Trajectory plots saved in ./output/RunSlingshot
- CytoTRACE output plots saved in ./output/CytoTRACEoutput
- Cross-tool pseudotime correlation assessment reproducing Figure 4E-4F (qualitative positive-correlation check)

## How to apply

Load the preprocessed Arabidopsis thaliana root Seurat object from the figshare auxiliary deposit for scPlant (https://figshare.com/articles/dataset/Auxiliary_data_and_codes_for_scPlant/23498402). Run scPlant's RunMonocle2 with save_plot = TRUE (max_components = 2) and RunMonocle3 with save_plot = TRUE, rooting both at meristem root cells, storing plots in ./output/RunMonocle2 and ./output/RunMonocle3. Run RunSlingshot with save_plot = TRUE into ./output/RunSlingshot, and RunCytoTRACE with save_plot = TRUE into ./output/CytoTRACEoutput, verifying that CytoTRACE reproduces its predicted (incorrect) xylem-cell root. Finally, compare pseudotimes across the four tools: although cell orderings differ among CytoTRACE, Monocle2, and Monocle3, their pseudotimes should show an overall positive correlation as in Figure 4F; the rationale is that agreement across independent methods increases confidence in the inferred developmental axis.

## Related tools

- **scPlant** (Framework providing the RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE wrappers used to infer and compare pseudotime trajectories) — https://github.com/compbioNJU/scPlant
- **R** (Implementation language in which the scPlant trajectory-inference functions are executed)
- **Seurat** (Provides the preprocessed Seurat object format that is the input to each scPlant trajectory-inference wrapper)

## Examples

```
result <- RunMonocle2(SeuratObj, save_plot = TRUE)
```

## Evaluation signals

- Plot files exist in all four output directories (./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput)
- Predicted pseudotimes from Monocle2, Monocle3, Slingshot, and CytoTRACE show the overall positive correlation reported for Figure 4E-4F (qualitative check on the correlation figure)
- CytoTRACE's predicted root reproduces its known (incorrect) xylem-cell root state on the Arabidopsis root atlas
- Monocle2 and Monocle3 trajectories are rooted at meristem root cells as specified

## Limitations

- Cell ordering along inferred trajectories differs among CytoTRACE, Monocle2, and Monocle3, so pseudotimes are only qualitatively comparable via correlation, not cell-by-cell
- CytoTRACE predicted an incorrect root (xylem cells rather than meristem) on this dataset, so it should not be trusted for rooting decisions here

## Evidence
