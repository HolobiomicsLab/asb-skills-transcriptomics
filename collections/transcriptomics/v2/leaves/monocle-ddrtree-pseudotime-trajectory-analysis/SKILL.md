---
name: monocle-ddrtree-pseudotime-trajectory-analysis
description: Use when scRNAseq clusters (e.g., robustSingleCell TIL subpopulations)
  need pseudotime/trajectory ordering, and you have overexpressed gene sets per group
  to seed Monocle DDRTree analysis.
when_to_use_negative:
- Input data has not been independently preclustered and robust subpopulations/overexpressed
  gene sets have not been defined — DDRTree should be run on defined TIL groups, not
  raw cells
- The goal is a pooled t-SNE similarity comparison across compartments (TILs vs dLN
  vs Arm cells) rather than pseudotime ordering within TIL groups
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Monocle
  role: Learns the DDRTree trajectory (max_components = 2, method = DDRTree) on TIL
    groups I and II using their overexpressed gene sets and plots the trajectory figure
- name: robustSingleCell
  role: Upstream pipeline that identifies the robust cell subpopulations and overexpressed
    gene sets on which the DDRTree trajectory is run
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Execution environment for Monocle v2.9.0 and robustSingleCell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:8ce28dc85bf4cecb1f14fe219a47e82498f78630e6432d6967957d19e4ffef3b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# monocle-ddrtree-pseudotime-trajectory-analysis

## Summary

Runs Monocle v2.9.0 DDRTree trajectory learning on T cell scRNAseq subpopulations (e.g., TIL groups I and II) using their overexpressed gene sets, to resolve subpopulation structure and lineage relationships in a 2D embedding.

## When to use

Use when scRNAseq clusters (e.g., robustSingleCell TIL subpopulations) need pseudotime/trajectory ordering, and you have overexpressed gene sets per group to seed Monocle DDRTree analysis.

## When NOT to use

- Input data has not been independently preclustered and robust subpopulations/overexpressed gene sets have not been defined — DDRTree should be run on defined TIL groups, not raw cells
- The goal is a pooled t-SNE similarity comparison across compartments (TILs vs dLN vs Arm cells) rather than pseudotime ordering within TIL groups

## Inputs

- Overexpressed gene sets (marker genes) from robustSingleCell clusters for TIL groups I and II
- Preclustered scRNAseq expression data for tumor-infiltrating CD4+ T cells (TILs)
- Monocle v2.9.0 installed in R

## Outputs

- DDRTree trajectory figure (plot of the learned 2D trajectory graph) saved in the analysis figs folder
- 2D trajectory embedding resolving TIL subpopulation structure

## How to apply

After robustSingleCell clustering has identified robust subpopulations (e.g., TIL groups I and II from the tumor-specific CD4+ T cell analysis), build a Monocle CellDataSet from the overexpressed gene sets of each TIL group. Run Monocle v2.9.0 with max_components = 2 and method = DDRTree so the learned graph embeds cells in 2D while preserving branched trajectory structure. Plot the learned DDRTree trajectory figure into the analysis figs folder alongside the pooled t-SNE outputs (tsne.png, CD4/CD8 contour overlays). Rationale: DDRTree resolves the distinct TIL subpopulation structure that the pooled t-SNE shows has little transcriptomic overlap between TILs, dLN cells, and Arm cells.

## Related tools

- **Monocle** (Learns the DDRTree trajectory (max_components = 2, method = DDRTree) on TIL groups I and II using their overexpressed gene sets and plots the trajectory figure)
- **robustSingleCell** (Upstream pipeline that identifies the robust cell subpopulations and overexpressed gene sets on which the DDRTree trajectory is run) — github.com/asmagen/robustSingleCell
- **R** (Execution environment for Monocle v2.9.0 and robustSingleCell)

## Examples

```
cds <- newCellDataSet(overexpr_ges); cds <- reduceDimension(cds, max_components = 2, method = 'DDRTree'); plot_cell_trajectory(cds)
```

## Evaluation signals

- The DDRTree trajectory figure is generated in the analysis figs folder alongside tsne.png and the CD4/CD8 contour t-SNE overlays
- Trajectory embedding resolves the distinct TIL subpopulation structure rather than collapsing groups into one undifferentiated state
- Monocle is run with the specified parameters: version 2.9.0, max_components = 2, method = DDRTree
- Trajectory input is restricted to TIL groups I and II using their overexpressed gene sets

## Limitations

- robustSingleCell (the upstream subpopulation-identification pipeline) is not currently maintained, per its README
- The pooled t-SNE of TILs, dLN cells, and Arm cells shows little transcriptomic overlap, so trajectory structure learned within TILs should not be assumed to connect dLN or Arm cells

## Evidence

- [other] Run Monocle v2.9.0 with max_components = 2 and method = DDRTree on TIL groups I and II using their overexpressed gene sets: [evidence span withheld — anchor status: not_found]
- [readme] robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data: "robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data"
- [abstract] identify TIL transcriptomic patterns strikingly distinct from acute and chronic anti-viral responses: "identify TIL transcriptomic patterns strikingly distinct from acute and chronic anti-viral responses and dominated by diversity among T-bet-expressing T helper type 1 (Th1)-like cells"
