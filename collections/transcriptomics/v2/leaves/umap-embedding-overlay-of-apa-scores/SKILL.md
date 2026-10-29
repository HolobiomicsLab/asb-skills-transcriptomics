---
name: umap-embedding-overlay-of-apa-scores
description: Use when single-cell poly(A) site data (PACdataset) has computed per-cell
  APA indices like RUD and you need to check spatial coherence of those scores on
  the cell embedding.
when_to_use_negative:
- The gene has no annotated 3'UTR APA (no proximal-distal pAs) and thus no RUD index
  can be computed
- The goal is genome-browser-style visualization of pA positions and read alignments,
  which is the vizTracks module, not the UMAP overlay
- Data are bulk APA profiles without per-cell resolution — UMAP embedding of cells
  is meaningless
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: vizAPA
  role: Provides vizUMAP to render the 2D embedding overlay of RUD scores and reduceDim
    to compute the UMAP embedding from the pA count matrix
  repo: https://github.com/BMILAB/vizAPA
- name: Seurat
  role: Alternative source of UMAP embeddings from the counts data in the PACdataset
- name: movAPA
  role: Supplies the PACdataset data structure and get3UTRAPApd for selecting proximal-distal
    pA pairs used in RUD calculation
- name: ggplot2
  role: Underlying plotting engine for composing the UMAP and summary panels of the
    figure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:1f142487d2745f8429ea5021eeb9f002a3f13e6e253153f8cd97c53248adb9f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Seurat
  - ggplot2
  - movAPA
  - vizAPA
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# umap-embedding-overlay-of-apa-scores

## Summary

Overlay per-cell APA usage metrics (e.g., RUD scores) onto a 2D UMAP embedding with vizAPA::vizUMAP to visualize whether alternative polyadenylation dynamics align with single-cell clusters, as demonstrated for Asrgl1 in mouse sperm cells.

## When to use

Use when single-cell poly(A) site data (PACdataset) has computed per-cell APA indices like RUD and you need to check spatial coherence of those scores on the cell embedding.

## When NOT to use

- The gene has no annotated 3'UTR APA (no proximal-distal pAs) and thus no RUD index can be computed
- The goal is genome-browser-style visualization of pA positions and read alignments, which is the vizTracks module, not the UMAP overlay
- Data are bulk APA profiles without per-cell resolution — UMAP embedding of cells is meaningless

## Inputs

- PACdataset of single-cell poly(A) site counts (e.g., scPACds from Sierra or scAPAtrap output)
- Per-gene/per-cell RUD index from getAPAindexPACds (choose2PA='PD') restricted to 3'UTR pAs
- UMAP coordinates (UMAP_1/UMAP_2) either stored in the PACdataset or computed via reduceDim/Seurat
- Cell type annotations (e.g., SC, RS, ES factor levels)

## Outputs

- UMAP scatter plot with cells colored by per-cell RUD score of the target gene (vizUMAP output)
- Complementary per-cell-type summary plots (heatmap, dot, violin) from vizStats of the same RUD scores

## How to apply

First learn a 2D embedding for the cells, either from the PACdataset count matrix with vizAPA::reduceDim (e.g., nfeature=2000, dims=1:10, norm=TRUE) or by importing UMAP coordinates computed with Seurat. Then call vizUMAP with xcol='UMAP_1' and ycol='UMAP_2' to plot cells colored by the target gene's RUD score (e.g., Asrgl1/entrez 66514 in the scPACds mouse sperm demo). RUD scores should only be computed for genes with 3'UTR APA after subsetting to 3UTR-annotated pAs, using vizAPA::getAPAindexPACds with choose2PA='PD' for the proximal-distal pair. Interpret the overlay together with heatmap/dot/violin views from vizStats: a gradient of RUD values along cell-type clusters (SC -> RS -> ES) indicates APA dynamics during differentiation.

## Related tools

- **vizAPA** (Provides vizUMAP to render the 2D embedding overlay of RUD scores and reduceDim to compute the UMAP embedding from the pA count matrix) — https://github.com/BMILAB/vizAPA
- **Seurat** (Alternative source of UMAP embeddings from the counts data in the PACdataset)
- **movAPA** (Supplies the PACdataset data structure and get3UTRAPApd for selecting proximal-distal pA pairs used in RUD calculation)
- **ggplot2** (Underlying plotting engine for composing the UMAP and summary panels of the figure)

## Examples

```
iPACds=getAPAindexPACds(scPACds, choose2PA='PD'); vizUMAP(scPACds, iPACds, gene='66514', xcol='UMAP_1', ycol='UMAP_2')
```

## Evaluation signals

- The UMAP overlay reproduces the expected gradient: Asrgl1 RUD score decreases from SC to ES, consistent with 3'UTR shortening during sperm cell differentiation
- Cells cluster by cell type on UMAP_1/UMAP_2, and RUD colors are spatially coherent within clusters rather than randomly mixed
- RUD values are only present for genes classified as 3'UTR-APA (subset scPACds@anno$ftr=='3UTR' before index calculation)
- Figure exports correctly as asrgl1_rud_figure.png (optionally editable PPTX via eoffice::topptx) with the combined heatmap/dot/violin/UMAP panels

## Limitations

- RUD can only be computed for genes with multiple annotated 3'UTR poly(A) sites; single-pA genes yield no index
- Embedding quality depends on the reduceDim parameters (e.g., nfeature=2000, dims=1:10) or on a precomputed Seurat UMAP; poor embeddings will blur the overlay
- vizUMAP requires UMAP_1/UMAP_2 columns to be present and correctly named in the colData/embeddings

## Evidence

- [readme] vizUMAP learns two-dimensional embeddings for visualizing clusters of cells with similar APA expression or usage profiles.: [evidence span withheld — anchor status: external_doc]
- [methods] scPACds=reduceDim(scPACds, nfeature=2000, dims=1:10, dimLabel='umap_norm', norm=TRUE): "scPACds=reduceDim(scPACds, nfeature=2000, dims=1:10, dimLabel='umap_norm', norm=TRUE)"
- [methods] vizUMAP can plot per-cell RUD scores on UMAP_1/UMAP_2 coordinates: "It is also possible to get embeddings based on the counts data in the `PACdataset`, using the `Seurat` package."
- [methods] The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES).: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -\> RS -\> ES)."
- [methods] only genes with 3'UTR APA can be used for RUD calculation: "# get 3UTR PACds, only genes with 3'UTR APA can be used for RUD calculation."
