---
name: single-cell-polya-expression-track-rendering
description: Use when you need a per-gene genome-browser view (e.g., Fig. 1C) showing
  where poly(A) sites sit in a gene's 3'UTR and how single-cell pA counts and bulk
  BAM coverage change across sample groups such as SC/RS/ES.
when_to_use_negative:
- The goal is group-level statistics of pA usage (violin/box/bubble/dot plots) rather
  than a per-gene genomic view — use vizStats instead
- The input is raw Sierra MEX output or a scAPAtrap RDA that has not yet been converted
  into a PACdataset via readPACds/createPACdataset
- You need APA-marker detection or UMAP embedding of APA profiles — vizAPAmarkers
  or vizUMAP are the appropriate modules
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: vizAPA
  role: Provides vizTracks plus the getTrack* helpers, PACdataset structure, and annoHub
    used to build the browser-like plot
  repo: https://github.com/BMILAB/vizAPA
- name: ggbio
  role: Combines individual tracks returned by getTrackGeneModel/getTrackPACds/getTrackCells/getTrackBams
    into one figure
- name: Sierra
  role: Upstream generator of poly(A) site data that can be converted to a PACdataset
    via readPACds for plotting
  repo: https://github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream source of pA lists convertible into PACdataset for track
    rendering
  repo: https://github.com/BMILAB/scAPAtrap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:69a1cf3f613eba7a6de6c6323ecd4863833622fe8959ae92befbf85f4f71c252
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-polya-expression-track-rendering

## Summary

Render a genome-browser-like multi-track plot of a single gene's poly(A) site usage with vizAPA::vizTracks, layering the gene model, PACdataset pA positions, per-cell expression grouped by cell type, and BAM read-coverage tracks for one or more biological conditions.

## When to use

Use when you need a per-gene genome-browser view (e.g., Fig. 1C) showing where poly(A) sites sit in a gene's 3'UTR and how single-cell pA counts and bulk BAM coverage change across sample groups such as SC/RS/ES.

## When NOT to use

- The goal is group-level statistics of pA usage (violin/box/bubble/dot plots) rather than a per-gene genomic view — use vizStats instead
- The input is raw Sierra MEX output or a scAPAtrap RDA that has not yet been converted into a PACdataset via readPACds/createPACdataset
- You need APA-marker detection or UMAP embedding of APA profiles — vizAPAmarkers or vizUMAP are the appropriate modules

## Inputs

- PACdataset of poly(A) sites with counts and cell-level colData (e.g., scPACds demo with 974 pAs, 413 genes, celltype annotations)
- Sorted, indexed BAM files per condition (e.g., dedup_GSM2803334.ES/RS/SC.mini.sorted.bam from the mouse.sperm.bam deposit) plus readBAMFileNames configuration
- Genome annotation source for annoHub (TxDb.Mmusculus.UCSC.mm10.knownGene or a GTF via useGff)
- Gene identifier or gene name to plot (e.g., gene=66514 for Asrgl1)

## Outputs

- Genome-browser-like multi-track ggplot/ggbio figure (gene model + pA positions + single-cell expression grouped by celltype + BAM coverage), saved e.g. as asrgl1_vizTracks.png
- Optionally, the four individual track objects from getTrackGeneModel/getTrackPACds/getTrackCells/getTrackBams for custom assembly

## How to apply

Load the demo PACdataset (data(scPACds, package='vizAPA'), 974 pAs from 413 mouse sperm genes) or convert Sierra/scAPAtrap output with readPACds, then prepare BAM tracks with readBAMFileNames, setting bam.order=c('SC','RS','ES') so BAM colors stay consistent with the PACdataset. Build an annoHub with new('annoHub') and addAnno (e.g., TxDb.Mmusculus.UCSC.mm10.knownGene) so the gene model can be drawn. Call vizTracks(gene=66514, bams=bams, PACds.list=list(PA=scPACds), PA.show=c('pos'), cells=TRUE, cells.group='celltype', cells.width=200, cells.method='sum', cells.sort='group', PA.columns='coord', PA.width=10, space5=0, space3=100) to assemble gene model, pA, single-cell, and coverage tracks. Alternatively, build the figure manually from getTrackGeneModel, getTrackPACds, getTrackCells, and getTrackBams (region extended by 1000 bp at both ends) combined with ggbio::tracks, zooming with xlim (e.g., c(9111000, 9113500) for the Asrgl1 3'UTR). Interpretation: dominant proximal vs distal pA read coverage across stages (e.g., Asrgl1's proximal site dominates in SC/RS/ES and coverage of both sites increases from ES to SC) signals potential APA dynamics during spermatogenesis.

## Related tools

- **vizAPA** (Provides vizTracks plus the getTrack* helpers, PACdataset structure, and annoHub used to build the browser-like plot) — https://github.com/BMILAB/vizAPA
- **ggbio** (Combines individual tracks returned by getTrackGeneModel/getTrackPACds/getTrackCells/getTrackBams into one figure)
- **Sierra** (Upstream generator of poly(A) site data that can be converted to a PACdataset via readPACds for plotting) — https://github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream source of pA lists convertible into PACdataset for track rendering) — https://github.com/BMILAB/scAPAtrap

## Examples

```
vizTracks(gene=66514, bams=bams, PACds.list=list(PA=scPACds), PA.show=c('pos'), cells=TRUE, cells.group='celltype', PA.columns='coord', space3=100)
```

## Evaluation signals

- Output PNG displays the correct gene model with its annotated 3'UTR poly(A) sites (e.g., two pAs for Asrgl1) at the expected coordinates
- BAM track colors match the PACdataset stage order because readBAMFileNames was called with bam.order=c('SC','RS','ES')
- Cell-level track shows per-cell pA counts grouped/sorted by 'celltype' (cells.group='celltype', cells.sort='group') with cells.width=200 aggregation window
- If assembled manually, the combined ggbio::tracks figure matches the direct vizTracks output for the same xlim (e.g., c(9111000, 9113500))
- Interpretation check: relative proximal-vs-distal coverage pattern and stage trend (Asrgl1 coverage increasing ES→SC) are readable from the rendered tracks

## Limitations

- BAM ordering must be set explicitly (bam.order) or track colors will be inconsistent with the PACdataset grouping
- Manual track assembly requires widening the region by 1000 bp at both ends; the demo gene's 3'UTR needs zooming (xlim) to be legible
- Only genes present in the PACdataset (e.g., 413 demo genes) and with available BAM/annotation data can be rendered; demo BAMs are mini subsets of GSM2803334

## Evidence

- [abstract] vizTracks gene-browser-like plot: [evidence span withheld — anchor status: external_doc]
- [methods] workflow call: [evidence span withheld — anchor status: not_found]
- [methods] bam ordering: [evidence span withheld — anchor status: not_found]
- [methods] manual assembly: [evidence span withheld — anchor status: not_found]
- [methods] demo dataset: "there is a demo `PACdataset` object of mouse sperm cells, containing 974 pAs [poly(A) sites] from 413 genes"
- [methods] mini BAMs: "extracted from the original BAM file (accession number: GSM2803334)"
