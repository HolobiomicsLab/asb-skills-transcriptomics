---
name: apa-dynamics-gradient-overlay-plotting
description: Use when a PACdataset of single-cell poly(A) site counts (e.g., scPACds
  with SC/RS/ES celltype colData) has UMAP_1/UMAP_2 embeddings and you need cluster
  plots overlaid with per-gene or mean pA-count gradients.
when_to_use_negative:
- The pA count matrix has not been dimensionality-reduced and no UMAP coordinates
  exist in colData and cannot be computed (reduceDim requires a normalized count matrix
  with enough features, e.g., nfeature=2000)
- The data are bulk APA samples rather than single cells with barcodes/cell-type annotations
  — vizUMAP clusters cells and has no per-cell grouping to display
- The goal is a genome-browser-style track view of pAs per gene (use vizTracks) rather
  than a cell-cluster embedding
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: vizAPA
  role: Provides vizUMAP to render the gradient-overlay cell embedding and reduceDim
    to compute umap_norm embeddings from the PACdataset counts
  repo: https://github.com/BMILAB/vizAPA
- name: Seurat
  role: Backend used by reduceDim to build the UMAP embedding from the normalized
    pA count matrix in the PACdataset
- name: scAPAtrap
  role: Upstream identification/quantification of single-cell pAs; its output (RDA
    of pA list) is converted to the PACdataset that vizUMAP consumes
  repo: https://github.com/BMILAB/scAPAtrap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:ce18c7aa992524f51d072957e17c7f80b9ada3661ab8097d7351326f4b1c7304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Seurat
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# apa-dynamics-gradient-overlay-plotting

## Summary

Render two-dimensional cell embeddings (UMAP) of single-cell poly(A) site data with vizAPA's vizUMAP, overlaying a color gradient (e.g., green-to-red scale via setStatTheme) of mean pA counts or a specific gene's expression to link APA dynamics to cell clusters.

## When to use

Use when a PACdataset of single-cell poly(A) site counts (e.g., scPACds with SC/RS/ES celltype colData) has UMAP_1/UMAP_2 embeddings and you need cluster plots overlaid with per-gene or mean pA-count gradients.

## When NOT to use

- The pA count matrix has not been dimensionality-reduced and no UMAP coordinates exist in colData and cannot be computed (reduceDim requires a normalized count matrix with enough features, e.g., nfeature=2000)
- The data are bulk APA samples rather than single cells with barcodes/cell-type annotations — vizUMAP clusters cells and has no per-cell grouping to display
- The goal is a genome-browser-style track view of pAs per gene (use vizTracks) rather than a cell-cluster embedding

## Inputs

- PACdataset of single-cell pA counts (e.g., scPACds from scAPAtrap or Sierra output read via readPACds) with colData cell-type annotations
- 2D embedding coordinates UMAP_1/UMAP_2 in scPACds@colData, or a raw pA count matrix to compute them via reduceDim
- Optional: gene name(s) to overlay as a per-gene expression gradient

## Outputs

- UMAP scatter plot of cells colored by celltype group with a MEAN gradient overlay of mean poly(A) site read counts
- Per-gene gradient-overlay UMAP (green-to-red scale via setStatTheme) written to a figure file

## How to apply

Load the PACdataset (e.g., data(scPACds) from vizAPA — 974 pAs from 413 genes across mouse sperm SC/RS/ES stages) and reorder the colData$celltype factor levels (c('SC','RS','ES')) so clusters are labeled in the differentiation order. Check that UMAP_1/UMAP_2 columns exist in scPACds@colData; if absent, compute them with reduceDim(PACds, dims=1:10, dimLabel='umap_norm', norm=TRUE), which normalizes the pA count matrix and runs Seurat-based dimensionality reduction. Call vizUMAP(scPACds, group='celltype', xcol='UMAP_1', ycol='UMAP_2') to plot cells colored by cell-type annotation with the MEAN overlay of mean poly(A) site read counts. To sharpen the gradient for a specific gene, re-invoke vizUMAP with genes=... plus setStatTheme parameters such as scale.low.col='green' and scale.high.col='red'. Validate by confirming the output figure file exists and that the embedding separates cells into clusters matching the annotated cell types.

## Related tools

- **vizAPA** (Provides vizUMAP to render the gradient-overlay cell embedding and reduceDim to compute umap_norm embeddings from the PACdataset counts) — https://github.com/BMILAB/vizAPA
- **Seurat** (Backend used by reduceDim to build the UMAP embedding from the normalized pA count matrix in the PACdataset)
- **scAPAtrap** (Upstream identification/quantification of single-cell pAs; its output (RDA of pA list) is converted to the PACdataset that vizUMAP consumes) — https://github.com/BMILAB/scAPAtrap

## Examples

```
vizUMAP(scPACds, group='celltype', xcol='UMAP_1', ycol='UMAP_2'); # gradient for a gene: vizUMAP(scPACds, genes=genename) with setStatTheme(scale.low.col='green', scale.high.col='red')
```

## Evaluation signals

- scPACds@colData contains UMAP_1 and UMAP_2 columns (or reduceDim was run with dimLabel='umap_norm') before plotting
- colData$celltype is a factor with levels ordered c('SC','RS','ES') so cluster labels render in differentiation order
- The rendered UMAP visually separates cells into three clusters matching SC, RS, and ES cell types, consistent with the gradual 3'UTR-shortening transition during sperm differentiation
- The gradient overlay spans the expected color scale (green at low, red at high) after setStatTheme(scale.low.col='green', scale.high.col='red')
- The output figure file exists on disk and, when genes=... is used, the highlighted gene (e.g., one whose RUD decreases from SC to ES) shows gradient intensity consistent with APA dynamics

## Limitations

- Embedding quality depends on upstream normalization and reduction settings (dims=1:10, norm=TRUE, nfeature=2000); without UMAP_1/UMAP_2 in colData the plot cannot be rendered
- Only genes with sufficient pA counts/3'UTR APA support give interpretable gradients — RUD-based overlays require genes with 3'UTR APA (get3UTRAPAds)
- vizUMAP visualizes single-cell APA expression/usage profiles; it does not itself test differential APA usage (that is vizAPAmarkers)

## Evidence

- [readme] vizUMAP learns two-dimensional embeddings for visualizing clusters of cells with similar APA expression or usage profiles.: [evidence span withheld — anchor status: external_doc]
- [methods] Embeddings can be computed from PACdataset counts with Seurat-backed reduceDim.: "It is also possible to get embeddings based on the counts data in the `PACdataset`, using the `Seurat` package."
- [methods] Concrete reduceDim call with normalization and 10 dims.: "scPACds=reduceDim(scPACds, nfeature=2000, dims=1:10, dimLabel='umap_norm', norm=TRUE)"
- [methods] Gradient overlay confirms biological ordering of clusters along sperm differentiation.: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -\> RS -\> ES)."
- [methods] Demo dataset defines the concrete inputs (974 pAs, 413 genes, three sperm stages).: "there is a demo `PACdataset` object of mouse sperm cells, containing 974 pAs [poly(A) sites] from 413 genes"
