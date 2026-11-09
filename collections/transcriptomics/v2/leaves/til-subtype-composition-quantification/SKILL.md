---
name: til-subtype-composition-quantification
description: Use when a scRNA-seq query of T cells (e.g. tetramer+ CD8 TILs or LCMV
  P14 cells) needs cell subtype frequencies, or when comparing perturbation effects
  on T cell state proportions (e.g. KO vs WT).
when_to_use_negative:
- The reference atlas does not cover the biology of interest — the atlases under-represent
  CD4+ TILs and contain no γδ T cells, so compositions for those lineages would be
  unreliable
- The goal is only fast label transfer without reference-space embedding — use ProjecTILs.classifier
  mode instead of full Run.ProjecTILs projection
- Quantifying composition of non-T cell types (e.g. stromal/CAF or DC queries) without
  switching to the corresponding cell-type-specific reference map
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ProjecTILs
  role: Projects the query onto the reference atlas (Run.ProjecTILs wrapping make.projection
    and cellstate.predict) and quantifies subtype composition with plot.statepred.composition
  repo: https://github.com/carmonalab/ProjecTILs
- name: STACAS
  role: Integrates heterogeneous scRNA-seq studies when building a custom reference
    atlas (make.reference) prior to composition quantification
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Data structure (Seurat object) for the query; ProjecTILs is integrated with
    Seurat for downstream analyses
  repo: https://github.com/carmonalab/ProjecTILs
- name: R
  role: Language in which the ProjecTILs package is implemented and executed
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:c77ff3031057ddbc577b37bbde6064a9a5debe273e279040f07a67fa30699308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# til-subtype-composition-quantification

## Summary

Quantify the subtype composition of a query single-cell T cell dataset by projecting it onto a reference atlas with ProjecTILs and computing frequencies of predicted cell states (e.g. Tex, Tpex, EM, SLEC) with plot.statepred.composition barplots.

## When to use

Use when a scRNA-seq query of T cells (e.g. tetramer+ CD8 TILs or LCMV P14 cells) needs cell subtype frequencies, or when comparing perturbation effects on T cell state proportions (e.g. KO vs WT).

## When NOT to use

- The reference atlas does not cover the biology of interest — the atlases under-represent CD4+ TILs and contain no γδ T cells, so compositions for those lineages would be unreliable
- The goal is only fast label transfer without reference-space embedding — use ProjecTILs.classifier mode instead of full Run.ProjecTILs projection
- Quantifying composition of non-T cell types (e.g. stromal/CAF or DC queries) without switching to the corresponding cell-type-specific reference map

## Inputs

- Query scRNA-seq expression matrix (10x/hdf5/raw or log counts) loadable into a Seurat object via read.sc.query
- Reference atlas: default murine TIL map (load.reference.map) or a custom reference built with make.reference (e.g. LCMV viral CD8 atlas from multiple studies via STACAS)
- Optional gene marker sets (e.g. cell.cycle.obj) for corroborating state assignments

## Outputs

- Projected Seurat object with query cells embedded in reference PCA/UMAP space and assigned cell state labels
- Barplot of cell state subtype frequencies per condition (plot.statepred.composition)
- Per-condition subtype composition percentages (e.g. Tex/Tpex/EM fractions) for comparison across conditions

## How to apply

Load a reference atlas (load.reference.map; mouse TIL atlas for B16-derived queries, or build a viral CD8 atlas from multiple LCMV studies with make.reference using STACAS integration into seven functional CD8 T cell clusters). Read the query expression matrix with read.sc.query into a Seurat object (raw counts expected; use skip.normalize=TRUE if data are pre-normalized). Run Run.ProjecTILs to embed the query in the reference PCA/UMAP space and assign cell state labels via nearest-neighbor prediction (cellstate.predict). Quantify subtype frequencies with plot.statepred.composition, which produces a barplot of cell state frequencies per condition; validate against expected compositions (e.g. ~90.2% Tex, 4.4% Tpex, 3.9% EM for B16-OVA tetramer+ TILs; ~20% Tex in WT vs >50% in Ptpn2-KO P14 cells). Optionally corroborate assignments with gene expression (plot.states.radar for Gzmb, Klrg1, Mki67) and cell.cycle.obj cycling scores.

## Related tools

- **ProjecTILs** (Projects the query onto the reference atlas (Run.ProjecTILs wrapping make.projection and cellstate.predict) and quantifies subtype composition with plot.statepred.composition) — https://github.com/carmonalab/ProjecTILs
- **STACAS** (Integrates heterogeneous scRNA-seq studies when building a custom reference atlas (make.reference) prior to composition quantification) — https://github.com/carmonalab/STACAS
- **Seurat** (Data structure (Seurat object) for the query; ProjecTILs is integrated with Seurat for downstream analyses) — https://github.com/carmonalab/ProjecTILs
- **R** (Language in which the ProjecTILs package is implemented and executed) — https://github.com/carmonalab/ProjecTILs

## Examples

```
query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); plot.statepred.composition(query.projected)
```

## Evaluation signals

- Projected subtype frequencies reproduce reported values within a few percent: B16-OVA tetramer+ CD8 TILs ~90.2% Tex, 4.4% Tpex, 3.9% EM on the murine TIL map
- Perturbation shifts are recapitulated: Ptpn2-KO P14 cells show Tex >50% versus ~20% in WT; Tox-KO shows SLEC expansion at the expense of Tpex with high Klrg1
- plot.statepred.composition produces a non-empty barplot of cell state frequencies for every condition
- All projected Seurat objects (query embedded in reference PCA/UMAP space with predicted labels) are generated without errors
- Corroborating marker plots (e.g. Gzmb/Mki67 radar via plot.states.radar; cycling score with cell.cycle.obj) are non-empty and consistent with assigned states

## Limitations

- Cell state quantifications are performed in the PCA latent space; UMAP is for visual exploration only since cell-cell distances and densities in UMAP are highly distorted
- Reference atlases do not encompass the full diversity of T cell transcriptional states, especially for CD4+ TILs (under-represented compared to CD8+), and γδ T cells are not represented at all
- Query cells whose states deviate from the reference may be forced into the nearest reference subtype; use find.discriminant.genes/dimensions to characterize deviations

## Evidence

- [other] Quantify subtype frequencies with plot.statepred.composition barplots: [evidence span withheld — anchor status: not_found]
- [methods] Run.ProjecTILs returns query embedded in reference space with predicted labels: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] cellstate.predict nearest-neighbor labeling: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [methods] plot.statepred.composition description: "Makes a barplot of the frequency of cell states in a query object"
- [readme] PCA used for quantification; UMAP caution: [evidence span withheld — anchor status: external_doc]
