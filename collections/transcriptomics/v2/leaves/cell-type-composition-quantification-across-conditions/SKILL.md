---
name: cell-type-composition-quantification-across-conditions
description: Use when comparing cell-state or cell-type proportions between conditions
  (e.g., KO vs WT) in scRNA-seq data where cells have been projected onto a reference
  atlas and assigned discrete cell-state labels.
when_to_use_negative:
- Data has not been projected onto a reference atlas and has no predicted cell-state
  labels — quantify composition only after make.projection/cellstate.predict
- Query cells belong to cell types or states absent from the reference atlas (e.g.,
  γδ T cells, under-represented CD4+ TILs), where frequency estimates would be misleading
- The goal is de novo clustering and annotation without a reference — use unsupervised
  Seurat clustering instead of reference-based state frequencies
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: ProjecTILs
  role: Projects query scRNA-seq data into the reference atlas space, assigns cell-state
    labels, and provides plot.statepred.composition for quantifying state frequencies
    per condition
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Hosts the query count matrices as Seurat objects (from 10x data via read.sc.query)
    and provides the object structure ProjecTILs operates on
- name: STACAS
  role: Integration tool underlying the multi-study construction of ProjecTILs reference
    atlases
  repo: https://github.com/carmonalab/STACAS
- name: ggplot2
  role: R plotting package used to render composition barplots and radar plots of
    state frequencies
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:51c9841b9ffdaac79418d95741fa27fa0a81729baa23cd8b37fccc77710693ac
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cell-type-composition-quantification-across-conditions

## Summary

Quantifies the frequency of reference-defined cell states in single-cell RNA-seq query datasets (e.g., miR-155 KO vs WT B16 TILs) after projection onto a reference atlas, enabling direct comparison of cell-type composition between biological conditions using ProjecTILs and plot.statepred.composition.

## When to use

Use when comparing cell-state or cell-type proportions between conditions (e.g., KO vs WT) in scRNA-seq data where cells have been projected onto a reference atlas and assigned discrete cell-state labels.

## When NOT to use

- Data has not been projected onto a reference atlas and has no predicted cell-state labels — quantify composition only after make.projection/cellstate.predict
- Query cells belong to cell types or states absent from the reference atlas (e.g., γδ T cells, under-represented CD4+ TILs), where frequency estimates would be misleading
- The goal is de novo clustering and annotation without a reference — use unsupervised Seurat clustering instead of reference-based state frequencies

## Inputs

- ProjecTILs-projected Seurat query object(s) with predicted cell-state labels (one per condition, e.g., miR-155 KO and WT CD45+ B16 TIL count matrices in 10x format)
- Loaded reference atlas (e.g., default mouse TIL atlas from load.reference.map, 16,803 transcriptomes from B16 and MC38 tumors)

## Outputs

- Barplot of cell-state frequency composition per condition (plot.statepred.composition)
- Quantitative cell-state fractions per condition (e.g., 16% KO vs 8% WT T-cell fraction of CD45+ cells)
- Radar plots of marker gene expression per state (plot.states.radar) and discriminant genes (find.discriminant.genes)

## How to apply

First project each condition's Seurat query object into the reference atlas space using make.projection or Run.ProjecTILs, which embeds the query in the reference PCA/UMAP space and assigns cell-state labels via cellstate.predict (nearest-neighbor voting with default k, nn.decay and min.confidence). Then quantify the fraction of cells assigned to each reference state per condition with plot.statepred.composition, which generates a barplot of cell-state frequencies in the query object. In the miR-155 KO vs WT B16 TIL case, this verified the reported 16% (KO) vs 8% (WT) T-cell fraction of CD45+ cells and revealed the naive-like shift of KO TILs. Follow up with plot.states.radar on marker genes (Tnfrsf9, Ifng, Tcf7, Ccr7) and find.discriminant.genes to confirm that frequency shifts correspond to biologically consistent expression differences. Interpret cell-state quantifications in the PCA latent space rather than UMAP densities, as UMAP distances are distorted.

## Related tools

- **ProjecTILs** (Projects query scRNA-seq data into the reference atlas space, assigns cell-state labels, and provides plot.statepred.composition for quantifying state frequencies per condition) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Hosts the query count matrices as Seurat objects (from 10x data via read.sc.query) and provides the object structure ProjecTILs operates on)
- **STACAS** (Integration tool underlying the multi-study construction of ProjecTILs reference atlases) — https://github.com/carmonalab/STACAS
- **ggplot2** (R plotting package used to render composition barplots and radar plots of state frequencies)

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_object, ref=ref); plot.statepred.composition(query.projected)
```

## Evaluation signals

- KO vs WT T-cell fraction of CD45+ cells approximates the reported 16% (KO) vs 8% (WT)
- KO TILs show a dominant naive-like reference state instead of the exhausted state dominant in WT
- plot.states.radar shows reduced Tnfrsf9 and Ifng in KO, with elevated Tcf7 and Ccr7 relative to WT
- find.discriminant.genes confirms the differential direction of activation/exhaustion markers between KO and WT projected cells
- Query cells embed in the reference without altering its structure (ProjecTILs invariant), with sensible confidence scores from cellstate.predict

## Limitations

- UMAP embeddings distort cell-cell distances and densities, so cell-state quantifications should be interpreted in the PCA latent space (Chari and Pachter 2023)
- Reference atlases do not encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented) and γδ T cells (not represented at all), limiting frequency estimates for such states
- Query objects are expected to contain raw counts, which ProjecTILs internally log-normalizes; using a different normalization requires skip.normalize=TRUE, and mismatched normalization can bias state assignment

## Evidence

- [methods] plot.statepred.composition: "`plot.statepred.composition`   Makes a barplot of the frequency of cell states in a query object"
- [methods] Run.ProjecTILs embedding and label transfer: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] read.sc.query input loading: "`read.sc.query`   Load a query expression matrix to be projected onto the reference atlas"
- [readme] stable reference coordinates rationale: "projecTILs is a computational method to project scRNA-seq data into reference single-cell atlases, enabling their direct comparison in a stable, annotated system of coordinates."
- [readme] UMAP distortion caveat: [evidence span withheld — anchor status: external_doc]
- [discussion] reference atlas coverage gap: "they do not yet encompass the full diversity of transcriptional states that can be acquired by T cells, especially for CD4+ TILs (which were under-represented compared to CD8+ among available data)"
