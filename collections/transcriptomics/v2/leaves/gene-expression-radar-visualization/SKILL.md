---
name: gene-expression-radar-visualization
description: Use when you have a Seurat query object projected onto a reference TIL
  atlas and need to compare gene-expression profiles (e.g. exhaustion vs naive-like
  markers) across predicted cell states, or to verify subtype assignments.
when_to_use_negative:
- When the query has not been projected/label-transferred onto a reference atlas —
  radar plots group genes by predicted reference states and require state assignments
- For quantitative cell-cell distance or density analysis — UMAP embeddings distort
  distances and are for visual exploration only
- When you need genome-wide differential expression rather than a curated marker panel
  — use find.discriminant.genes instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjecTILs
  role: Provides plot.states.radar for radar plots of gene expression per predicted
    cell state, plus find.discriminant.genes to select discriminating genes from a
    projected query
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure (Seurat object) holding the projected query whose expression
    levels are visualized
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:8513c298f4a862b61421de224d6c57ef23c1b9a1dbf841d0176b028565868266
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-expression-radar-visualization

## Summary

Visualize the expression level of selected genes (e.g. Gzmb, Mki67, Tnfrsf9, Ifng) across predicted reference cell states of a projected query using the ProjecTILs `plot.states.radar` radar-plot function, typically after label transfer with Run.ProjecTILs and optionally after find.discriminant.genes.

## When to use

Use when you have a Seurat query object projected onto a reference TIL atlas and need to compare gene-expression profiles (e.g. exhaustion vs naive-like markers) across predicted cell states, or to verify subtype assignments.

## When NOT to use

- When the query has not been projected/label-transferred onto a reference atlas — radar plots group genes by predicted reference states and require state assignments
- For quantitative cell-cell distance or density analysis — UMAP embeddings distort distances and are for visual exploration only
- When you need genome-wide differential expression rather than a curated marker panel — use find.discriminant.genes instead

## Inputs

- Seurat object returned by Run.ProjecTILs / cellstate.predict with assigned cell state labels
- Named vector or character list of genes to plot (state-defining markers or find.discriminant.genes output)
- Reference map used for the projection (e.g. the default murine TIL atlas from load.reference.map)

## Outputs

- Radar plot of gene expression levels across predicted reference cell states
- Optionally, per-state expression values used to confirm differential marker direction between conditions (e.g. KO vs WT)

## How to apply

First project the query expression matrix onto the reference map with Run.ProjecTILs (or make.projection followed by cellstate.predict), which embeds cells in the reference PCA/UMAP space and assigns cell state labels via nearest-neighbor prediction. Optionally run find.discriminant.genes to identify which genes discriminate the projected query from the reference or a control sample. Then call plot.states.radar with the projected object and the chosen gene set to produce a radar plot of mean expression per predicted cell state. Choose genes with state-defining biology, e.g. Gzmb and Mki67 to profile exhausted/cycling B16 tetramer+ CD8 TILs, or Tnfrsf9, Ifng, Tcf7 and Ccr7 to show the naive-like shift of miR-155 KO TILs. Interpret radar profiles jointly with cell-state composition (plot.statepred.composition); remember quantifications are made in the PCA latent space, and UMAP is for visual exploration only.

## Related tools

- **ProjecTILs** (Provides plot.states.radar for radar plots of gene expression per predicted cell state, plus find.discriminant.genes to select discriminating genes from a projected query) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure (Seurat object) holding the projected query whose expression levels are visualized)

## Examples

```
library(ProjecTILs); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); plot.states.radar(query.projected, genes=c("Gzmb","Mki67"))
```

## Evaluation signals

- The radar plot renders non-empty and shows distinct expression profiles per predicted state (e.g. high Gzmb in Tex, elevated Tcf7/Ccr7 in naive-like states)
- Marker direction matches expectations: Tnfrsf9 and Ifng reduced, Tcf7 and Ccr7 elevated in naive-like shifted KO TILs vs WT
- Genes plotted correspond to find.discriminant.genes output or biologically defined state markers (Gzmb, Mki67, Tnfrsf9, Ifng)
- State groupings in the radar plot agree with subtype frequencies from plot.statepred.composition (e.g. ~90.2% Tex for B16 tetramer+ TILs)

## Limitations

- Radar plots summarize mean expression per predicted state, not single-cell resolution; low-confidence state assignments will blur gene-state profiles
- UMAP-based visual exploration should be interpreted with caution as cell-cell distances and densities in UMAP are highly distorted; quantitative comparisons belong in the PCA latent space
- Genes absent from the query (e.g. cross-species or low-detection markers) yield misleading radar lobes; check gene detection before plotting

## Evidence

- [methods] plot.states.radar: radar plot of marker expression across predicted states: "`plot.states.radar`   Makes a radar plot of the expression level of a specified set of genes"
- [methods] find.discriminant.genes supports marker selection: "`find.discriminant.genes`   Performs differential expression analysis between a projected query and a control (either the reference map or a control sample)"
- [methods] projection step precedes visualization: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [readme] PCA vs UMAP caveat for interpreting visualizations: [evidence span withheld — anchor status: external_doc]
- [readme] default projection invocation that produces the object visualized: [evidence span withheld — anchor status: external_doc]
