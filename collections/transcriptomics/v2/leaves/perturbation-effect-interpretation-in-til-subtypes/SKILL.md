---
name: perturbation-effect-interpretation-in-til-subtypes
description: Use when comparing a perturbed scRNA-seq condition (e.g., Regnase-1-null
  vs WT CD8+ TILs) against a control to distinguish changes in cell-state abundance
  from subtype-specific gene-program deviations.
when_to_use_negative:
- When the goal is only fast label transfer without altering the dimensionality reduction
  — use the ProjecTILs.classifier mode instead of full reference embedding
- When interpreting UMAP-based cell-cell distances or densities for quantitative claims
  — the README cautions these are highly distorted and quantification must be done
  in the PCA latent space
- When the query data is not T-cell-derived, e.g., gamma-delta T cells, which are
  not represented at all in the reference atlases
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: 'Core framework: load.reference.map, read.sc.query, Run.ProjecTILs, plot.statepred.composition,
    find.discriminant.dimensions, find.discriminant.genes'
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Seurat object format for query data; ProjecTILs is integrated with Seurat
    so results can be combined with downstream Seurat analyses
- name: STACAS
  role: Integration method used upstream to generate the multi-study reference atlases
    onto which perturbation data are projected
  repo: https://github.com/carmonalab/STACAS
- name: R
  role: Language in which ProjecTILs is implemented and the perturbation-interpretation
    workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:6b4e7b7ec18b86295769b69f9ac3f96e9b2c57c205919a76047eb3115096e421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# perturbation-effect-interpretation-in-til-subtypes

## Summary

Interpret cell-perturbation effects (e.g., Regnase-1 knockout in CD8+ TILs) by projecting perturbed and control scRNA-seq data onto the ProjecTILs mouse TIL reference atlas, comparing cell-state composition, and using discriminant ICA to identify altered gene programs within specific subtypes such as Tex.

## When to use

Use when comparing a perturbed scRNA-seq condition (e.g., Regnase-1-null vs WT CD8+ TILs) against a control to distinguish changes in cell-state abundance from subtype-specific gene-program deviations.

## When NOT to use

- When the goal is only fast label transfer without altering the dimensionality reduction — use the ProjecTILs.classifier mode instead of full reference embedding
- When interpreting UMAP-based cell-cell distances or densities for quantitative claims — the README cautions these are highly distorted and quantification must be done in the PCA latent space
- When the query data is not T-cell-derived, e.g., gamma-delta T cells, which are not represented at all in the reference atlases

## Inputs

- Perturbed-condition scRNA-seq raw count matrix (e.g., Regnase-1-null CD8+ TIL expression matrix from Wei et al.)
- Control-condition scRNA-seq raw count matrix (e.g., WT CD8+ TILs)
- ProjecTILs reference atlas (default mouse TIL reference map loaded via load.reference.map)
- Cell-state labels of interest (e.g., Tex, Tpex) for subtype-restricted discriminant analysis

## Outputs

- Projected Seurat objects embedded in the reference PCA/UMAP space with predicted cell-state labels
- Cell-state frequency composition barplot comparing perturbed vs control (plot.statepred.composition)
- Ranked table of discriminant ICA dimensions with KS statistics per condition/subtype
- Identification of the top discriminating gene program (e.g., ICA 25 driven by Lag3 and Klrc1) with direction of regulation
- Optionally, differential-expression gene list from find.discriminant.genes

## How to apply

Load the default tumour-infiltrating lymphocyte reference atlas with load.reference.map, then load the query expression matrices (e.g., Wei et al. Regnase-1-null and WT CD8+ TIL counts) with read.sc.query as Seurat objects; ProjecTILs log-normalizes raw counts internally unless skip.normalize=TRUE. Project each condition with Run.ProjecTILs, which embeds the query in the reference PCA/UMAP space and transfers cell-state labels (Tpex, Tex, etc.). First quantify cell-state frequency shifts with plot.statepred.composition to capture abundance effects (validation: ~3-fold Tpex and ~1.8-fold Tex enrichment in Regnase-1-null). Then, within a single subtype (Tex), run find.discriminant.dimensions with reduction='ICA' to rank components where the perturbed cells deviate from the control; confirm the top component (ICA 25, KS statistic ~0.612, p = 0, driven by Lag3/Klrc1 and down-regulated in the knockout) and optionally use find.discriminant.genes for the underlying DE genes. Interpret perturbation effects as the combination of composition shifts and subtype-internal program deviations.

## Related tools

- **ProjecTILs** (Core framework: load.reference.map, read.sc.query, Run.ProjecTILs, plot.statepred.composition, find.discriminant.dimensions, find.discriminant.genes) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Seurat object format for query data; ProjecTILs is integrated with Seurat so results can be combined with downstream Seurat analyses)
- **STACAS** (Integration method used upstream to generate the multi-study reference atlases onto which perturbation data are projected) — https://github.com/carmonalab/STACAS
- **R** (Language in which ProjecTILs is implemented and the perturbation-interpretation workflow is executed)

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); find.discriminant.dimensions(query.projected, ref=ref, reduction='ICA')
```

## Evaluation signals

- Cell-state composition validates the expected perturbation phenotype: Tpex fold-change ~3 and Tex fold-change ~1.8 in Regnase-1-null vs WT CD8+ TILs
- find.discriminant.dimensions with ICA reduction ranks ICA 25 as the top component for the Tex subtype, with KS statistic 0.612 and p = 0
- The top discriminant component is driven by checkpoint molecules Lag3 and Klrc1 and is down-regulated in Regnase-1-null Tex cells
- Query cells are embedded in the reference PCA/UMAP space without altering the reference structure (reference-space integrity check)
- Query object contains raw counts that are internally log-normalized; if pre-normalized data is used, the skip.normalize=TRUE flag must have been set

## Limitations

- UMAP embeddings are for visual exploration only; cell-cell distances and densities in UMAP are highly distorted, so quantifications must use the PCA latent space
- The reference atlases do not encompass the full diversity of T cell transcriptional states, particularly for CD4+ TILs, and gamma-delta T cells are not represented at all
- The analysis depends on the quality and coverage of the reference atlas; deviations may reflect missing reference states rather than true perturbation effects

## Evidence

- [abstract] ProjecTILs accurately predicts the effects of cell perturbations and identifies gene programs that are altered in different conditions and tissues: "ProjecTILs accurately predicts the effects of cell perturbations and identifies gene programs that are altered in different conditions and tissues"
- [methods] find.discriminant.dimensions searches PCA or ICA dimensions: "Searches PCA or ICA dimensions where the query set deviates the most from a control set or from the reference map"
- [methods] Run.ProjecTILs embeds the query and predicts cell states: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [readme] README: raw counts are internally log-normalized: [evidence span withheld — anchor status: external_doc]
- [readme] README: quantification must use PCA, not UMAP: [evidence span withheld — anchor status: external_doc]
