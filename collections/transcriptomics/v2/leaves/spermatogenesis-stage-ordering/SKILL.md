---
name: spermatogenesis-stage-ordering
description: Use when single-cell APA data from mouse sperm cells (SC, RS, ES) should
  be visualized as a RUD-score gradient on 2D UMAP embeddings to reveal 3'UTR shortening
  during differentiation.
when_to_use_negative:
- Input data are not 3'UTR poly(A) sites with at least a proximal-distal pair per
  gene — only genes with 3'UTR APA can be used for RUD calculation
- The analysis targets bulk APA dynamics across samples rather than single-cell-level
  visualization (use vizTracks/vizStats modules instead)
- Cell embeddings (UMAP_1/UMAP_2) or cell type annotations are absent from the PACdataset
  colData
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: vizAPA
  role: Provides getAPAindexPACds (choose2PA='PD') to compute per-gene, per-cell RUD
    scores and vizUMAP to plot the RUD-gradient 2D embedding; ships the scPACds mouse
    sperm demo dataset
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Underlying PACdataset infrastructure; supplies 3'UTR APA helpers (get3UTRAPApd)
    used for proximal-distal pair selection
  repo: https://github.com/BMILAB/vizAPA
- name: R
  role: Execution environment for the vizAPA workflow (R >= 4.2.2 recommended)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:5ef957dc822c747de0d7c3f64eb03d377d05283d36ca7e0ece419dc53786c648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - movAPA
  - vizAPA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# spermatogenesis-stage-ordering

## Summary

Visualizes global 3'UTR shortening dynamics across mouse spermatogenesis by overlaying per-cell mean RUD (Relative Usage of Distal poly(A) site) scores as a gradient on vizUMAP 2D embeddings of sperm cells ordered SC -> RS -> ES.

## When to use

Use when single-cell APA data from mouse sperm cells (SC, RS, ES) should be visualized as a RUD-score gradient on 2D UMAP embeddings to reveal 3'UTR shortening during differentiation.

## When NOT to use

- Input data are not 3'UTR poly(A) sites with at least a proximal-distal pair per gene — only genes with 3'UTR APA can be used for RUD calculation
- The analysis targets bulk APA dynamics across samples rather than single-cell-level visualization (use vizTracks/vizStats modules instead)
- Cell embeddings (UMAP_1/UMAP_2) or cell type annotations are absent from the PACdataset colData

## Inputs

- PACdataset of mouse sperm cell poly(A) sites (demo scPACds, 974 pAs from 413 genes, via data(scPACds, package='vizAPA'))
- celltype annotation in colData with factor levels releveled to c('SC','RS','ES')
- UMAP_1/UMAP_2 2D embeddings stored in colData (e.g., from reduceDim)

## Outputs

- Gradient-color UMAP scatter plot of mean per-cell RUD scores showing the SC -> RS -> ES 3'UTR-shortening transition
- Optional exported figure (e.g., rud_umap_overlay.png via eoffice::topptx)

## How to apply

Load the demo scPACds PACdataset of mouse sperm cells (974 pAs from 413 genes) shipped with vizAPA via data(scPACds, package='vizAPA') and relevel the celltype factor to c('SC','RS','ES') to match sperm differentiation order. Subset the PACdataset to 3'UTR features (scPACds@anno$ftr=='3UTR'), since only genes with 3'UTR APA can be used for RUD calculation, then compute the proximal-distal RUD index with vizAPA::getAPAindexPACds using choose2PA='PD'. Plot with vizUMAP(RUD, group='celltype', xcol='UMAP_1', ycol='UMAP_2'), which overlays each cell's mean RUD score as gradient color using the UMAP_1/UMAP_2 coordinates stored in colData (computed via reduceDim). Optionally export via eoffice::topptx as rud_umap_overlay.png. Correct output shows a gradual transition of decreased RUD scores (3'UTR shortening) along SC -> RS -> ES.

## Related tools

- **vizAPA** (Provides getAPAindexPACds (choose2PA='PD') to compute per-gene, per-cell RUD scores and vizUMAP to plot the RUD-gradient 2D embedding; ships the scPACds mouse sperm demo dataset) — https://github.com/BMILAB/vizAPA
- **movAPA** (Underlying PACdataset infrastructure; supplies 3'UTR APA helpers (get3UTRAPApd) used for proximal-distal pair selection) — https://github.com/BMILAB/vizAPA
- **R** (Execution environment for the vizAPA workflow (R >= 4.2.2 recommended))

## Examples

```
scPACds<-scPACds[scPACds@anno$ftr=="3UTR"]; RUD=getAPAindexPACds(scPACds, choose2PA="PD"); vizUMAP(RUD, group='celltype', xcol='UMAP_1', ycol='UMAP_2')
```

## Evaluation signals

- The rendered UMAP gradient plot exists and exhibits a gradual transition of decreased RUD scores (3'UTR shortening) along SC -> RS -> ES, as stated in the tutorial
- RUD values are computed only on 3'UTR features (scPACds@anno$ftr=='3UTR') using a proximal-distal pair selection (choose2PA='PD')
- celltype factor levels are ordered c('SC','RS','ES') so the differentiation sequence is respected in the plot
- Each cell carries a single mean RUD score mapped to gradient color on UMAP_1/UMAP_2 coordinates in colData

## Limitations

- RUD calculation requires genes with 3'UTR APA (a proximal and distal pair); genes without such pairs are excluded
- Demonstrated on the built-in mouse sperm demo dataset (974 pAs, 413 genes); new data must first be processed into a PACdataset with 2D embeddings computed (e.g., reduceDim with nfeature=2000, dims=1:10)
- The plot conveys a global average trend; per-gene APA dynamics (e.g., Asrgl1 RUD decreasing from SC to ES) require separate marker analysis

## Evidence

- [methods] The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES).: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES)."
- [methods] iPACds=getAPAindexPACds(scPACds, choose2PA = "PD"): "iPACds=getAPAindexPACds(scPACds, choose2PA = "PD")"
- [methods] there is a demo `PACdataset` object of mouse sperm cells, containing 974 pAs [poly(A) sites] from 413 genes: "there is a demo `PACdataset` object of mouse sperm cells, containing 974 pAs [poly(A) sites] from 413 genes"
- [readme] vizUMAP that learns two-dimensional embeddings for visualizing clusters of cells with simi-lar APA expression or usage profiles.: [evidence span withheld — anchor status: external_doc]
