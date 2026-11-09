---
name: apa-usage-heatmap-visualization-across-cell-types
description: Use when you have a PACdataset of poly(A) site counts with cell-type
  annotations (e.g., sperm cell stages SC/RS/ES) and want to display which genes show
  differential APA usage (RUD/avg_log2FC) across cell types as a heatmap or companion
  rich plots.
when_to_use_negative:
- You want usage of a single specific poly(A) site or gene across groups — use vizStats
  (violin/box/bubble/dot for a given pA) rather than the marker heatmap
- The data are bulk APA samples with no cell-level grouping or no computable proximal-distal
  3'UTR pA pairs, so no RUD index or APA markers can be derived
- The PACdataset has not been subset to 3'UTR features — only genes with 3'UTR APA
  can be used for RUD calculation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: vizAPA
  role: Provides getAPAindexPACds, getAPAmarkers, and vizAPAMarkers (figType='heatmap')
    plus vizStats/vizUMAP to compute and plot cell-type APA markers
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Underlying PACdataset/movAPA utilities (e.g., get3UTRAPApd) used with vizAPA
    for RUD computation
  repo: https://github.com/BMILAB/movAPA
- name: Sierra
  role: Upstream detection and quantification of poly(A) sites from scRNA-seq that
    generate the PAC data visualized here
  repo: https://github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream tool for identifying and quantifying pAs from 3' tag-based
    scRNA-seq; its results feed the PACdataset
  repo: https://github.com/BMILAB/scAPAtrap
- name: ggplot2
  role: Underlying plotting engine for vizAPA figure rendering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:a3cba9f4ba1e6e074f2786858e6a4af749f1f232c30a629ba7d1e17cd7448002
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Sierra
  - ggplot2
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# apa-usage-heatmap-visualization-across-cell-types

## Summary

Visualize cell-type-specific alternative polyadenylation (APA) usage by detecting APA markers (genes with differential APA usage, e.g., via RUD index) and rendering them with vizAPA's vizAPAMarkers heatmap (plus violin, bubble, dot, and UMAP variants) across cell groups.

## When to use

Use when you have a PACdataset of poly(A) site counts with cell-type annotations (e.g., sperm cell stages SC/RS/ES) and want to display which genes show differential APA usage (RUD/avg_log2FC) across cell types as a heatmap or companion rich plots.

## When NOT to use

- You want usage of a single specific poly(A) site or gene across groups — use vizStats (violin/box/bubble/dot for a given pA) rather than the marker heatmap
- The data are bulk APA samples with no cell-level grouping or no computable proximal-distal 3'UTR pA pairs, so no RUD index or APA markers can be derived
- The PACdataset has not been subset to 3'UTR features — only genes with 3'UTR APA can be used for RUD calculation

## Inputs

- PACdataset object of poly(A) site counts with cell-type colData (e.g., scPACds from Sierra or scAPAtrap output converted via readPACds)
- 3'UTR-restricted PACdataset and RUD APA index (iPACds) from getAPAindexPACds(choose2PA='PD')
- APA marker list (rowid gene entries with avg_log2FC) from getAPAmarkers with min.pct=0.1, logFC=0.12

## Outputs

- Heatmap of RUD scores for the selected APA markers across cell types (vizAPAMarkers figType='heatmap')
- Companion violin, bubble (incl. axis-swapped statTheme=list(xgroup=FALSE) variant), per-marker dot plot (vizStats), and UMAP overlay figures
- Exported figures (e.g., via eoffice::topptx to figures.pptx)

## How to apply

Load or build a PACdataset (e.g., data(scPACds, package='vizAPA') for the mouse sperm demo with 974 pAs from 413 genes) and order the celltype factor to match the biology (SC < RS < ES). Subset to 3'UTR poly(A) sites (scPACds[scPACds@anno$ftr=='3UTR']) and compute a proximal–distal APA index with getAPAindexPACds(choose2PA='PD') to obtain RUD (relative usage of distal pA) scores. Detect APA markers between cell-type pairs with getAPAmarkers(RUD, group='celltype', everyPair=TRUE, min.pct=0.1, logFC=0.12) using the wilcoxon rank sum test, and take the top rowid entries as the marker list. Render the markers with vizAPAMarkers(figType='heatmap') for all cell types, optionally restricted to the two stages involved, and complement with figType='violin', 'bubble' (including statTheme=list(xgroup=FALSE) axis-swapped variant), and per-marker vizStats(figType='dot', gene=marker) or figType='umap' overlays. Positive avg_log2FC markers indicate higher RUD (longer 3'UTR usage) in the first group; negative values indicate 3'UTR shortening.

## Related tools

- **vizAPA** (Provides getAPAindexPACds, getAPAmarkers, and vizAPAMarkers (figType='heatmap') plus vizStats/vizUMAP to compute and plot cell-type APA markers) — https://github.com/BMILAB/vizAPA
- **movAPA** (Underlying PACdataset/movAPA utilities (e.g., get3UTRAPApd) used with vizAPA for RUD computation) — https://github.com/BMILAB/movAPA
- **Sierra** (Upstream detection and quantification of poly(A) sites from scRNA-seq that generate the PAC data visualized here) — https://github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream tool for identifying and quantifying pAs from 3' tag-based scRNA-seq; its results feed the PACdataset) — https://github.com/BMILAB/scAPAtrap
- **ggplot2** (Underlying plotting engine for vizAPA figure rendering)

## Examples

```
iPACds <- getAPAindexPACds(scPACds[scPACds@anno$ftr=='3UTR',], choose2PA='PD'); markers <- getAPAmarkers(iPACds, group='celltype', everyPair=TRUE, min.pct=0.1, logFC=0.12); vizAPAMarkers(figType='heatmap', markers=markers)
```

## Evaluation signals

- Top markers between RS/SC show clearly higher RUD scores in SC than in RS/ES (positive avg_log2FC), consistent with the reported stage-specific APA dynamics
- Markers between ES and SC are predominantly negative (avg_log2FC<0), i.e., lower RUD/shorter 3'UTR in ES than in SC
- Heatmap and violin renderings of the same marker list show concordant cell-type patterns (e.g., RUD in SC significantly different from other sperm differentiation stages)
- Celltype factor levels ordered SC < RS < ES before plotting so the heatmap columns reflect the differentiation trajectory
- All figures export cleanly (eoffice::topptx to figures.pptx) and the bubble plot axis-swapped variant (statTheme=list(xgroup=FALSE)) renders without error

## Limitations

- RUD calculation requires genes with at least two annotated 3'UTR pAs; only genes with 3'UTR APA can be used (get3UTRAPAds / choose2PA='PD' filtering)
- Marker detection thresholds (min.pct=0.1, logFC=0.12, wilcoxon rank sum test) determine the marker set; sensitivity of the heatmap to these cutoffs is not benchmarked
- Demo dataset is limited (974 pAs from 413 genes in mouse sperm cells); other tissues require full Sierra/scAPAtrap preprocessing including internal priming removal and pA merging within 24 nt

## Evidence

- [readme] vizAPAMarkers identifies genes with differential APA usages (called APA markers) and generate rich plots: [evidence span withheld — anchor status: external_doc]
- [abstract] vizAPA enables identification of genes with differential APA usage across biological samples and/or cell types: "vizAPA also enables identification of genes with differential APA usage across biological samples and/or cell types."
- [methods] Markers between ES and SC have lower RUD (shorter 3'UTR) in ES: "The higher nagative markers between ES and SC means these markers have lower RUD scores (avg_log2FC<0, shorter 3'UTR) in ES than in SC."
- [methods] 3'UTR shortening transition during sperm cell differentiation: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -\> RS -\> ES)."
