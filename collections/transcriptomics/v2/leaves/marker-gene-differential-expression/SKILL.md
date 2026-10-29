---
name: marker-gene-differential-expression
description: Use when a scRNA-seq query has been projected onto a reference atlas
  (e.g. murine TIL atlas) and you need subtype-level marker genes or perturbation-altered
  gene programs, such as KO-vs-WT shifts in Tnfrsf9/Ifng/Tcf7/Ccr7.
when_to_use_negative:
- When the query has not yet been projected onto a reference atlas — find.discriminant.genes
  requires a projected query and a control in a shared feature space
- When an unbiased, reference-free cell-type annotation is needed; this skill interprets
  states relative to predefined reference subtypes
- When analyzing cell types absent from the reference (e.g. gamma-delta T cells, which
  are not represented in the TIL atlases)
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ProjecTILs
  role: Provides find.discriminant.genes for DE between projected query and control,
    plus plot.states.radar and celltype.heatmap for visualization
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure holding the projected query; ProjecTILs is integrated with
    Seurat so DE results feed downstream Seurat analyses
- name: STACAS
  role: Used upstream to integrate heterogeneous datasets when building custom references
    against which DE is computed
  repo: https://github.com/carmonalab/STACAS
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:d4e139710c1f91a2b2b586e68decc469f99b997bcf5fb36521ea125af1e39650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  - TILPRED
  - ggplot2
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-gene-differential-expression

## Summary

Performs differential expression (marker-gene) analysis between a query dataset projected onto a ProjecTILs reference atlas and a control (the reference map itself or a control sample), using find.discriminant.genes to identify gene programs altered in the query's T cell states.

## When to use

Use when a scRNA-seq query has been projected onto a reference atlas (e.g. murine TIL atlas) and you need subtype-level marker genes or perturbation-altered gene programs, such as KO-vs-WT shifts in Tnfrsf9/Ifng/Tcf7/Ccr7.

## When NOT to use

- When the query has not yet been projected onto a reference atlas — find.discriminant.genes requires a projected query and a control in a shared feature space
- When an unbiased, reference-free cell-type annotation is needed; this skill interprets states relative to predefined reference subtypes
- When analyzing cell types absent from the reference (e.g. gamma-delta T cells, which are not represented in the TIL atlases)

## Inputs

- Projected Seurat object (query embedded in reference PCA/UMAP space with predicted cell state labels)
- Control set: the reference map (e.g. murine TIL atlas) or a control sample Seurat object
- Optional: Hs2Mm.convert.table human-to-mouse ortholog table for human query data
- Gene set of interest for radar plots (e.g. Tnfrsf9, Ifng, Tcf7, Ccr7)

## Outputs

- Per-subtype differential expression (marker gene) tables from find.discriminant.genes
- Averaged gene expression heatmap (celltype.heatmap)
- Radar plots of expression of specified genes per state (plot.states.radar)

## How to apply

After embedding the query with make.projection/Run.ProjecTILs and assigning states with cellstate.predict (nearest-neighbor voting with default k and min.confidence), run find.discriminant.genes per reference subtype, comparing each projected query against a control set — either the murine reference map or a control sample — to obtain subtype-level differential expression tables. For cross-species comparisons, first convert human genes to mouse orthologs with Hs2Mm.convert.table before projection. Visualize and confirm the differential direction with plot.states.radar (radar plots of specified genes) and summarize averaged expression per subtype with celltype.heatmap. Accept the result when the identified markers are consistent with known phenotype signatures of the predicted states (e.g. Pdcd1/Havcr2/Tox/Gzmb for Tex; Tcf7/Ccr7 for naive-like or Tpex).

## Related tools

- **ProjecTILs** (Provides find.discriminant.genes for DE between projected query and control, plus plot.states.radar and celltype.heatmap for visualization) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure holding the projected query; ProjecTILs is integrated with Seurat so DE results feed downstream Seurat analyses)
- **STACAS** (Used upstream to integrate heterogeneous datasets when building custom references against which DE is computed) — https://github.com/carmonalab/STACAS

## Examples

```
library(ProjecTILs); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); DE.genes <- find.discriminant.genes(query.projected, ref=ref); plot.states.radar(query.projected, genes=c("Tnfrsf9","Ifng","Tcf7","Ccr7"))
```

## Evaluation signals

- find.discriminant.genes returns non-empty per-subtype DE tables with genes ranked by differential expression between query and control
- For miR-155 KO vs WT B16 TILs, Tnfrsf9 and Ifng are reduced while Tcf7 and Ccr7 are elevated in KO relative to WT, confirming the naive-like shift
- celltype.heatmap of averaged subtype-study expression profiles clusters by TIL reference subtype rather than by study or species (human-mouse co-clustering p < 3e-6 in the meta-analysis)
- Marker direction matches reference-state biology: exhausted states show inhibitory receptors and cytotoxic molecules; naive-like/Tpex states show Tcf7

## Limitations

- The reference atlases do not encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented) and gamma-delta T cells (not represented), limiting DE interpretation for those populations
- UMAP embeddings are for visual exploration only; cell-cell distances and densities in UMAP are highly distorted, so DE should be interpreted in the PCA latent space
- DE comparisons assume the query states are adequately represented in the reference; genuinely novel states that deviate from all reference subtypes may be misassigned

## Evidence

- [methods] find.discriminant.genes: "`find.discriminant.genes`   Performs differential expression analysis between a projected query and a control (either the reference map or a control sample)"
- [methods] celltype.heatmap: "`celltype.heatmap`   Generate a averaged gene expression heatmap from a Seurat object"
- [methods] plot.states.radar: "`plot.states.radar`   Makes a radar plot of the expression level of a specified set of genes"
- [abstract] projection preserves reference: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure"
- [abstract] perturbation prediction: "ProjecTILs accurately predicts the effects of cell perturbations and identifies gene programs that are altered in different conditions and tissues"
- [discussion] Seurat integration: "Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools for up- and down-stream analyses."
- [discussion] ortholog mapping: "ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas"
- [readme] UMAP caution: [evidence span withheld — anchor status: external_doc]
