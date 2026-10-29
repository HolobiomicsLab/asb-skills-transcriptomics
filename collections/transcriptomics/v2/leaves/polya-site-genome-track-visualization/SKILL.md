---
name: polya-site-genome-track-visualization
description: Use when you have a PACdataset of poly(A) sites with per-cell or per-sample
  counts plus BAM alignments and genome annotation, and need to visually inspect proximal
  vs distal pA usage for a specific gene.
when_to_use_negative:
- You need quantitative differential APA statistics (e.g., RUD scores or APA markers)
  rather than a visual coverage/usage inspection — use vizStats or vizAPAmarkers instead
- Your poly(A) sites have not yet been identified or converted into a PACdataset (e.g.,
  raw Sierra or scAPAtrap output still needs readPACds conversion)
- No genome annotation is available, since the gene model track requires an annoHub
  built from TxDb or GTF
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: vizAPA
  role: Provides the PACdataset structure, annoHub, readBAMFileNames, vizTracks, and
    getTrack* functions that build the genome-browser-like plot
  repo: https://github.com/BMILAB/vizAPA
- name: ggbio
  role: Underlying R graphics package used to combine the individual gene model, pA,
    cells, and BAM tracks via ggbio::tracks
- name: Sierra
  role: Upstream tool whose poly(A) site calls can be read with readPACds into the
    PACdataset visualized by vizTracks
  repo: github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream tool whose identified pAs can be converted to a PACdataset
    with readPACds for visualization
  repo: github.com/BMILAB/scAPAtrap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:4c5dc0c40f53afa4c50274de1d8d6751d7164296e55f34a889822ea42ebeb6a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Sierra
  - ggbio
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# polya-site-genome-track-visualization

## Summary

Generates a genome-browser-like multi-track plot (gene model, poly(A) sites, single-cell counts, BAM read coverage) for a gene of interest using the vizAPA vizTracks module on R/ggbio, to visually inspect poly(A) site usage across cell types or samples.

## When to use

Use when you have a PACdataset of poly(A) sites with per-cell or per-sample counts plus BAM alignments and genome annotation, and need to visually inspect proximal vs distal pA usage for a specific gene.

## When NOT to use

- You need quantitative differential APA statistics (e.g., RUD scores or APA markers) rather than a visual coverage/usage inspection — use vizStats or vizAPAmarkers instead
- Your poly(A) sites have not yet been identified or converted into a PACdataset (e.g., raw Sierra or scAPAtrap output still needs readPACds conversion)
- No genome annotation is available, since the gene model track requires an annoHub built from TxDb or GTF

## Inputs

- PACdataset of poly(A) sites with per-cell counts and colData annotations (e.g., scPACds demo dataset with 974 pAs from 413 genes)
- BAM alignment files with .bai indexes (e.g., dedup_GSM2803334.ES/RS/SC.mini.sorted.bam from mouse.sperm.bam deposit)
- Genome annotation (e.g., TxDb.Mmusculus.UCSC.mm10.knownGene or GTF such as Homo_sapiens.GRCh38.109.gtf.gz)

## Outputs

- Genome-browser-like multi-track plot (PNG, e.g., asrgl1_vizTracks.png) showing gene model, annotated poly(A) site positions, per-cell expression grouped by celltype, and per-sample BAM read coverage

## How to apply

Load the poly(A) site data into a PACdataset (e.g., data(scPACds, package='vizAPA'), the mouse sperm demo with 974 pAs from 413 genes), then prepare BAM tracks with readBAMFileNames, setting bam.order (e.g., c('SC','RS','ES')) so BAM colors stay consistent with the PACdataset samples. Build an annoHub by instantiating new('annoHub') and loading an annotation source such as TxDb.Mmusculus.UCSC.mm10.knownGene via addAnno. Call vizTracks(gene=..., bams=..., PACds.list=list(PA=scPACds), PA.show='pos', cells=TRUE, cells.group='celltype', cells.width=200, cells.method='sum', PA.columns='coord', PA.width=10, space3=100) to render the gene model, pA, single-cell, and BAM coverage tracks. Alternatively, assemble the figure manually from getTrackGeneModel, getTrackPACds, getTrackCells, and getTrackBams (region extended 1000 bp on both ends) combined with ggbio::tracks, and zoom with xlim (e.g., c(9111000, 9113500) for the Asrgl1 3'UTR) to focus on the pA sites.

## Related tools

- **vizAPA** (Provides the PACdataset structure, annoHub, readBAMFileNames, vizTracks, and getTrack* functions that build the genome-browser-like plot) — https://github.com/BMILAB/vizAPA
- **ggbio** (Underlying R graphics package used to combine the individual gene model, pA, cells, and BAM tracks via ggbio::tracks)
- **Sierra** (Upstream tool whose poly(A) site calls can be read with readPACds into the PACdataset visualized by vizTracks) — github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream tool whose identified pAs can be converted to a PACdataset with readPACds for visualization) — github.com/BMILAB/scAPAtrap

## Examples

```
vizTracks(gene=66514, bams=bams, PACds.list=list(PA=scPACds), PA.show=c('pos'), cells=TRUE, cells.group='celltype', cells.width=200, cells.method='sum', cells.sort='group', PA.columns='coord', PA.width=10, space5=0, space3=100)
```

## Evaluation signals

- Output file (e.g., asrgl1_vizTracks.png) exists and renders the full multi-track layout: gene model with two annotated 3'UTR pA sites, per-cell expression grouped by celltype, and SC/RS/ES BAM coverage tracks
- BAM track colors match the PACdataset sample order because bam.order was set consistently (e.g., c('SC','RS','ES'))
- Zoomed xlim shows the intended 3'UTR window (e.g., 9111000–9113500 for Asrgl1) with both proximal and distal pAs visible
- The plot reproduces the tutorial's expected pattern: proximal Asrgl1 pA dominant over the distal one, with coverage increasing from ES to SC

## Limitations

- vizTracks visualizes only one gene (or region) at a time; it does not itself compute differential APA usage — RUD/APA-marker analysis requires separate get3UTRAPAds/getAPAindexPACds steps
- BAM files and their .bai index files must be available and ordered consistently (bam.order) or track colors will mismatch the PACdataset sample order
- Manual track assembly requires extending the region by 1000 bp at both ends via the getTrack* functions, otherwise flanking context may be clipped

## Evidence

- [readme] Visualization module vizTracks that generates a genome-browser-like plot, which utilizes tracks to display different types information related to APA, including gene models, positions and expression levels or usages of poly(A) sites, read alignments, and single-cell read counts: [evidence span withheld — anchor status: external_doc]
- [methods] readBAMFileNames with bam.order keeps BAM colors consistent with the PACdataset: [evidence span withheld — anchor status: not_found]
- [methods] vizTracks call with key parameters for the Asrgl1 example: [evidence span withheld — anchor status: not_found]
- [methods] Manual track assembly workflow with ggbio::tracks and region extension: [evidence span withheld — anchor status: not_found]
- [methods] Demo dataset size: 974 pAs from 413 genes in the mouse sperm scPACds: "there is a demo `PACdataset` object of mouse sperm cells, containing 974 pAs [poly(A) sites] from 413 genes"
- [methods] BAM files and indexes downloadable from the vizAPA GitHub site: [evidence span withheld — anchor status: not_found]
