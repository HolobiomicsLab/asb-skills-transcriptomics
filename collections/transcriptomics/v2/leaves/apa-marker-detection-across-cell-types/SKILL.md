---
name: apa-marker-detection-across-cell-types
description: Use when you have a PACdataset of poly(A) sites with cell group annotations
  (e.g., sperm differentiation stages SC/RS/ES) and need to identify and visualize
  per-cell-type APA usage differences such as 3'UTR shortening.
when_to_use_negative:
- The APA data has not yet been converted into a PACdataset with cell-type colData
  — first import Sierra or scAPAtrap results with readPACds and annotate with annotatePAC.
- You only want to visualize usage of a single known poly(A) site across groups without
  testing for markers — vizStats alone suffices.
- Genes lack 3'UTR APA (no two pAs with usable proximal-distal separation), so no
  RUD index can be computed for them.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: vizAPA
  role: Provides getAPAindexPACds, getAPAmarkers, vizAPAMarkers, and vizStats used
    to compute APA markers and render violin/heatmap/bubble/dot/UMAP plots
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Supplies get3UTRAPApd for choosing proximal/distal pA pairs and shared PACdataset
    utilities (loaded alongside vizAPA)
  repo: https://github.com/BMILAB/movAPA
- name: Sierra
  role: Upstream identification and quantification of poly(A) sites from scRNA-seq
    whose output (PACdataset) feeds marker detection
  repo: github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream tool for identifying and quantifying poly(A) sites from
    3' tag-based scRNA-seq, convertible into PACdataset input
  repo: github.com/BMILAB/scAPAtrap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:1baa240f5f4fd697428ce2f04b1717482d435c860de8c4762fbac6bb88cdf9de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Sierra
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# apa-marker-detection-across-cell-types

## Summary

Detects genes with differential alternative polyadenylation (APA) usage — 'APA markers' — between cell types or stages using vizAPA's getAPAmarkers on an APA usage index (e.g., RUD scores), then visualizes them with vizAPAMarkers as violin, heatmap, bubble, dot, or UMAP plots.

## When to use

Use when you have a PACdataset of poly(A) sites with cell group annotations (e.g., sperm differentiation stages SC/RS/ES) and need to identify and visualize per-cell-type APA usage differences such as 3'UTR shortening.

## When NOT to use

- The APA data has not yet been converted into a PACdataset with cell-type colData — first import Sierra or scAPAtrap results with readPACds and annotate with annotatePAC.
- You only want to visualize usage of a single known poly(A) site across groups without testing for markers — vizStats alone suffices.
- Genes lack 3'UTR APA (no two pAs with usable proximal-distal separation), so no RUD index can be computed for them.

## Inputs

- PACdataset of single-cell poly(A) site counts with cell type annotations in colData (e.g., scPACds mouse sperm demo dataset with 974 pAs from 413 genes)
- 3'UTR-subset PACdataset with proximal-distal pA pairs (from get3UTRAPApd, minDist=50, maxDist=5000, minRatio=0.05)
- APA usage index matrix (RUD scores per cell per gene) from getAPAindexPACds(choose2PD='PD')
- Cell group factor, reordered to biological progression (SC < RS < ES)

## Outputs

- APA marker list per cell-type pair with avg_log2FC and detection statistics (wilcoxon rank sum test)
- Violin, heatmap, bubble (optionally axis-swapped via statTheme=list(xgroup=FALSE)), dot, and UMAP figures of selected markers
- Exported figures file (e.g., figures.pptx via eoffice::topptx)

## How to apply

First compute a per-cell APA usage index: subset the PACdataset to 3'UTR poly(A) sites (scPACds@anno$ftr=='3UTR'), obtain proximal-distal pA pairs with movAPA::get3UTRAPApd (minDist=50, maxDist=5000, minRatio=0.05), and calculate RUD (relative usage of distal pA) with getAPAindexPACds(choose2PA='PD'). Then run getAPAmarkers(RUD, group='celltype', everyPair=TRUE, min.pct=0.1, logFC=0.12) using the wilcoxon rank sum test, which tests every pair of cell types and returns markers with avg_log2FC; positive log2FC means higher RUD (longer 3'UTR usage) in the first group. Take top rowid entries as the marker list and render with vizAPAMarkers(figType='violin'|'heatmap'|'bubble'|'umap'), optionally restricting selGroups to the two stages being compared, and produce per-gene dot plots with vizStats(figType='dot', gene=marker). Export figures with eoffice::topptx to figures.pptx.

## Related tools

- **vizAPA** (Provides getAPAindexPACds, getAPAmarkers, vizAPAMarkers, and vizStats used to compute APA markers and render violin/heatmap/bubble/dot/UMAP plots) — https://github.com/BMILAB/vizAPA
- **movAPA** (Supplies get3UTRAPApd for choosing proximal/distal pA pairs and shared PACdataset utilities (loaded alongside vizAPA)) — https://github.com/BMILAB/movAPA
- **Sierra** (Upstream identification and quantification of poly(A) sites from scRNA-seq whose output (PACdataset) feeds marker detection) — github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream tool for identifying and quantifying poly(A) sites from 3' tag-based scRNA-seq, convertible into PACdataset input) — github.com/BMILAB/scAPAtrap

## Examples

```
RUD = getAPAindexPACds(scPACds[scPACds@anno$ftr=='3UTR'], choose2PA='PD'); markers = getAPAmarkers(RUD, group='celltype', everyPair=TRUE, min.pct=0.1, logFC=0.12); vizAPAMarkers(markers, figType='violin')
```

## Evaluation signals

- Markers between RS/SC show clearly higher RUD scores in SC than in RS/ES (positive avg_log2FC), consistent with the reported 3'UTR-shortening trend during sperm differentiation.
- Most ES-vs-SC markers have negative avg_log2FC (<0), i.e., lower RUD (shorter 3'UTR) in ES than in SC.
- Violin plots show statistically distinct RUD score distributions between stages (e.g., SC differs significantly from other sperm differentiation stages).
- Per-gene UMAP plots overlay RUD usage on UMAP_1/UMAP_2 embeddings and show stage-specific separation consistent with cell type labels.
- All figures export cleanly via eoffice::topptx to figures.pptx.

## Limitations

- Only genes with 3'UTR APA can be used for RUD calculation; single-pA genes are dropped by get3UTRAPAds/choose2PA='PD'.
- Marker calls depend on detection thresholds (min.pct=0.1, logFC=0.12) and the wilcoxon rank sum test; weakly expressed pAs may be filtered out before testing.
- Interpretation is directional: negative markers between ES and SC indicate shorter 3'UTR in ES, so sign conventions must be checked against the factor ordering of celltype (SC < RS < ES).

## Evidence

- [abstract] vizAPA also enables identification of genes with differential APA usage across biological samples and/or cell types.: "vizAPA also enables identification of genes with differential APA usage across biological samples and/or cell types."
- [methods] getAPAindexPACds choose2PA='PD': "iPACds=getAPAindexPACds(scPACds, choose2PA = "PD")"
- [methods] The higher negative markers between ES and SC mean lower RUD scores in ES.: "The higher nagative markers between ES and SC means these markers have lower RUD scores (avg_log2FC<0, shorter 3'UTR) in ES than in SC."
- [methods] Only genes with 3'UTR APA can be used for RUD calculation.: "# get 3UTR PACds, only genes with 3'UTR APA can be used for RUD calculation."
- [readme] vizAPAmarkers identifies genes with differential APA usages and generates rich plots.: [evidence span withheld — anchor status: external_doc]
