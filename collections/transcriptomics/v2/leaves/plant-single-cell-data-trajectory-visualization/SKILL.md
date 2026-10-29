---
name: plant-single-cell-data-trajectory-visualization
description: Use when you have a preprocessed plant Seurat object (e.g. Arabidopsis
  root cells) and need to infer and visualize developmental trajectories/pseudotime,
  especially to compare consistency across multiple trajectory tools.
when_to_use_negative:
- Input is raw count data that has not yet been processed — scPlant expects a preprocessed
  Seurat object (run QCfun/RunSeurat/readData first)
- Goal is gene regulatory network construction or cross-species integration (use pySCENIC/MINI-EX
  or crossSpecies_integrate instead)
- Only a single trajectory tool is acceptable and root placement must be guaranteed
  — CytoTRACE rooted incorrectly (xylem) on the Arabidopsis root atlas
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Framework providing RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE
    trajectory wrappers on Seurat objects
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying Seurat object/preprocessing that supplies the input to scPlant
    trajectory functions
- name: R
  role: Implementation language for running scPlant trajectory functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:3d27a1fdeb8e0ad69cc00ddf61262d3c0efbb0a4ab9272d5ea856e99835ffa3a
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
yes_rate: 0.9
n_claims: 10
---

# plant-single-cell-data-trajectory-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run pseudotime trajectory inference on a plant scRNA-seq Seurat object using scPlant's wrappers around Monocle2, Monocle3, Slingshot, and CytoTRACE, saving trajectory plots per tool and cross-comparing predicted pseudotimes (as demonstrated on the Arabidopsis thaliana root atlas, Zhang et al., 2019).

## When to use

Use when you have a preprocessed plant Seurat object (e.g. Arabidopsis root cells) and need to infer and visualize developmental trajectories/pseudotime, especially to compare consistency across multiple trajectory tools.

## When NOT to use

- Input is raw count data that has not yet been processed — scPlant expects a preprocessed Seurat object (run QCfun/RunSeurat/readData first)
- Goal is gene regulatory network construction or cross-species integration (use pySCENIC/MINI-EX or crossSpecies_integrate instead)
- Only a single trajectory tool is acceptable and root placement must be guaranteed — CytoTRACE rooted incorrectly (xylem) on the Arabidopsis root atlas

## Inputs

- Preprocessed Seurat object (e.g. Arabidopsis thaliana root scRNA-seq, Zhang et al., 2019, PRJNA517021)
- Cell-type labels (e.g. meristem root cells) for specifying the trajectory root
- Auxiliary data and codes for scPlant from figshare (deposit 23498402)

## Outputs

- Trajectory plots stored in ./output/RunMonocle2
- Trajectory plots stored in ./output/RunMonocle3
- Trajectory plots stored in ./output/RunSlingshot
- Trajectory plots stored in ./output/CytoTRACEoutput
- Per-cell pseudotime values from Monocle2, Monocle3, Slingshot, and CytoTRACE for cross-tool correlation (cf. Figure 4E–4F)

## How to apply

Load the preprocessed Seurat object (e.g. the Arabidopsis thaliana root dataset, PRJNA517021, from the figshare auxiliary deposit 23498402) and call scPlant's RunMonocle2 and RunMonocle3 with save_plot = TRUE, rooting trajectories at meristem root cells (Monocle2 with max_components = 2 by default). Then run RunSlingshot and RunCytoTRACE with save_plot = TRUE, storing each tool's plots in a separate output directory (./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput). The rationale, per the scPlant paper, is that although cell ordering differs among CytoTRACE, Monocle2, and Monocle3, their predicted pseudotimes show an overall positive correlation, so running all four lets you sanity-check trajectory consistency (Figure 4E–4F). Note the known failure mode: on the Arabidopsis root data, CytoTRACE predicted an incorrect xylem-cell root, so verify root placement rather than trusting any single tool. Judge success by the existence of plot files in all four directories and qualitative positive correlation among the four pseudotime estimates.

## Related tools

- **scPlant** (Framework providing RunMonocle2, RunMonocle3, RunSlingshot, and RunCytoTRACE trajectory wrappers on Seurat objects) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying Seurat object/preprocessing that supplies the input to scPlant trajectory functions)
- **R** (Implementation language for running scPlant trajectory functions)

## Examples

```
result <- RunMonocle2(SeuratObj, save_plot = TRUE)
```

## Evaluation signals

- Plot files exist in all four output directories (./output/RunMonocle2, ./output/RunMonocle3, ./output/RunSlingshot, ./output/CytoTRACEoutput)
- Predicted pseudotimes from Monocle2, Monocle3, Slingshot, and CytoTRACE show the overall positive correlation reported in Figure 4E–4F (qualitative check on the correlation figure)
- CytoTRACE reproduction matches the paper's reported (incorrect) xylem-cell root on the Arabidopsis root data
- Monocle2 run uses the default max_components = 2 and trajectories are rooted at meristem root cells

## Limitations

- Cell ordering along inferred trajectories differs among CytoTRACE, Monocle2, and Monocle3, even though pseudotimes are positively correlated overall
- CytoTRACE predicted an incorrect xylem-cell root on the Arabidopsis root atlas, so root placement is tool-dependent and must be verified
- Validation in the source workflow is qualitative (visual check of the correlation figure), not a quantitative correlation threshold

## Evidence

- [results] Although the ordering of cells along the inferred trajectories differed among CytoTRACE, Monocle2, and Monocle3, the pseudotimes predicted by these methods show an overall positive correlation (Figure 4F).: [evidence span withheld — anchor status: not_found]
