---
name: single-cell-apd-dynamics-interpretation
description: Use when a PACdataset of single-cell poly(A) sites (e.g. from Sierra
  or scAPAtrap) exists and the goal is to interpret APA dynamics — 3'UTR shortening,
  differential pA usage, or APA markers — across cell types or biological stages.
when_to_use_negative:
- Input is bulk 3-seq/PAT-seq data without single-cell resolution and cell-level annotations
  — vizAPA's single-cell modules (vizUMAP, per-cell tracks) require barcoded scRNA-seq
  pA counts.
- Raw scRNA-seq FASTQ/BAM only — poly(A) sites must first be identified and quantified
  by an upstream tool (Sierra or scAPAtrap) before vizAPA interpretation.
- The research goal is peak calling or pA identification itself, not visualization
  and interpretation of already-called poly(A) sites.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: vizAPA
  role: Core R package providing PACdataset data structure, annoHub annotation, and
    visualization modules (vizTracks, vizStats, vizUMAP, vizAPAmarkers) for interpreting
    single-cell APA dynamics
  repo: https://github.com/BMILAB/vizAPA
- name: Sierra
  role: Upstream tool that identifies and quantifies poly(A) sites (peaks) from polyA-captured
    scRNA-seq BAM files; its MEX output is converted to a PACdataset for interpretation
  repo: https://github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream tool identifying and quantifying APA sites per cell from
    3' tag-based scRNA-seq; its rda output is read into vizAPA via readPACds
  repo: https://github.com/BMILAB/scAPAtrap
- name: movAPA
  role: Companion package loaded with vizAPA providing get3UTRAPApd and related helpers
    for choosing proximal/distal pAs and computing 3'UTR APA indices
  repo: https://github.com/BMILAB/movAPA
- name: Seurat
  role: Used optionally to compute cell embeddings (e.g. UMAP) from the pA count matrix
    in the PACdataset
- name: ggbio
  role: Track-layout engine used by vizTracks (and getTrackGeneModel/getTrackBams
    assembly) to render the genome-browser-like combined plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:b5e2f45b7ffa40bb1622ebcd4046e9246bd0a6f83b437ae9d73fd61d91c443df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Seurat
  - Sierra
  - ggbio
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# single-cell-apd-dynamics-interpretation

## Summary

Interpret single-cell alternative polyadenylation (APA) dynamics by visualizing poly(A) site usage, 3'UTR shortening (RUD scores), and APA markers across cell groups with vizAPA, e.g. tracking poly(A) site usage shifts of a gene like Asrgl1 across spermatogenesis stages (SC, RS, ES).

## When to use

Use when a PACdataset of single-cell poly(A) sites (e.g. from Sierra or scAPAtrap) exists and the goal is to interpret APA dynamics — 3'UTR shortening, differential pA usage, or APA markers — across cell types or biological stages.

## When NOT to use

- Input is bulk 3-seq/PAT-seq data without single-cell resolution and cell-level annotations — vizAPA's single-cell modules (vizUMAP, per-cell tracks) require barcoded scRNA-seq pA counts.
- Raw scRNA-seq FASTQ/BAM only — poly(A) sites must first be identified and quantified by an upstream tool (Sierra or scAPAtrap) before vizAPA interpretation.
- The research goal is peak calling or pA identification itself, not visualization and interpretation of already-called poly(A) sites.

## Inputs

- PACdataset of poly(A) sites from Sierra (e.g. pbmc2_sierra_APA.rda via Sierra::ReadPeakCounts MEX output) or scAPAtrap (scAPAtrapData.rda peaks.count/peaks.meta)
- Cell type annotation colData file (barcodes with celltype labels)
- BAM files with .bai index (e.g. dedup_GSM2803334.ES/RS/SC.mini.sorted.bam from mouse.sperm.bam; pbmc2.bam)
- Genome annotation: GTF (e.g. Homo_sapiens.GRCh38.109.gtf.gz or TxDb.Mmusculus.UCSC.mm10.knownGene) loaded into an annoHub
- Reference genome BSgenome object (e.g. BSgenome.Hsapiens.NCBI.GRCh38) for internal priming and sequence extraction

## Outputs

- Genome-browser-like vizTracks PNG (e.g. asrgl1_vizTracks.png) showing gene model, annotated 3'UTR poly(A) sites, per-cell expression grouped by celltype, and SC/RS/ES BAM coverage tracks
- vizStats violin/box/bubble/dot plots of proximal-distal pA usage across cell groups
- RUD scores / APA index (iPACds) revealing 3'UTR shortening or lengthening dynamics
- APA marker gene lists with violin, heat map, bubble, and UMAP plots from vizAPAmarkers

## How to apply

Import Sierra or scAPAtrap poly(A) site results into a vizAPA PACdataset via readPACds (e.g. scPACds <- readPACds(pacFile=pafile, colDataFile=coldata, noIntergenic=FALSE)), attach cell type annotations to colData, and filter out internal priming with removePACdsIP (e.g. up=-140, dn=10, conA=6 on BSgenome.Mmusculus.UCSC.mm10). For 3'UTR-APA interpretation, extend the 3'UTR by 1000bp with ext3UTRPACds, subset to anno$ftr=='3UTR', select genes with 3'UTR APA using get3UTRAPApd (minDist=50, maxDist=5000, minRatio=0.05), and build a proximal-distal index with getAPAindexPACds(choose2PA='PD') for RUD calculation. Interpret dynamics through vizAPA's visualization modules: vizTracks for a genome-browser-like plot of gene model, poly(A) sites, single-cell counts, and BAM coverage tracks (e.g. vizTracks(gene=66514, bams=bams, PACds.list=list(PA=scPACds), PA.show=c('pos'), cells.group='celltype')), vizStats for violin/box/bubble plots of pA usage across cell groups, and vizAPAmarkers for genes with differential APA usage (avg_log2FC<0 markers have lower RUD, i.e. shorter 3'UTR). Validate interpretation by checking that, e.g., the Asrgl1 proximal poly(A) site dominates across stages, coverage of both sites increases from ES to SC, and the RUD score decreases from SC to ES, consistent with 3'UTR shortening during sperm cell differentiation (SC -> RS -> ES).

## Related tools

- **vizAPA** (Core R package providing PACdataset data structure, annoHub annotation, and visualization modules (vizTracks, vizStats, vizUMAP, vizAPAmarkers) for interpreting single-cell APA dynamics) — https://github.com/BMILAB/vizAPA
- **Sierra** (Upstream tool that identifies and quantifies poly(A) sites (peaks) from polyA-captured scRNA-seq BAM files; its MEX output is converted to a PACdataset for interpretation) — https://github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream tool identifying and quantifying APA sites per cell from 3' tag-based scRNA-seq; its rda output is read into vizAPA via readPACds) — https://github.com/BMILAB/scAPAtrap
- **movAPA** (Companion package loaded with vizAPA providing get3UTRAPApd and related helpers for choosing proximal/distal pAs and computing 3'UTR APA indices) — https://github.com/BMILAB/movAPA
- **Seurat** (Used optionally to compute cell embeddings (e.g. UMAP) from the pA count matrix in the PACdataset)
- **ggbio** (Track-layout engine used by vizTracks (and getTrackGeneModel/getTrackBams assembly) to render the genome-browser-like combined plot)

## Examples

```
vizTracks(gene=66514, bams=bams, PACds.list=list(PA=scPACds), PA.show=c('pos'), cells=TRUE, cells.group='celltype', cells.width=200, cells.method='sum', cells.sort='group', PA.columns='coord', PA.width=10, space5=0, space3=100)
```

## Evaluation signals

- The rendered vizTracks figure displays the gene model with the expected annotated poly(A) sites — for Asrgl1 (gene=66514), two 3'UTR pAs at zoom xlim=c(9111000, 9113500) with the 3'UTR region visible.
- Per-cell expression track is grouped by celltype (cells.group='celltype', cells.width=200, cells.method='sum') and BAM coverage track colors match bam.order=c('SC','RS','ES') consistently with the PACdataset.
- Interpretive check: for Asrgl1, the proximal pA is more dominant than the distal pA across all stages and coverage of both sites increases from ES to SC, while the RUD score decreases from SC to ES — consistent with 3'UTR shortening during sperm cell differentiation.
- PACdataset invariants hold after filtering: demo mouse sperm scPACds contains 974 pAs from 413 genes; 3'UTR subsetting leaves only anno$ftr=='3UTR' rows; internal-priming-filtered pAs survive removePACdsIP(up=-140, dn=10, conA=6).
- vizAPAmarkers output: negative ES-vs-SC markers (avg_log2FC<0) correspond to lower RUD scores (shorter 3'UTR) in ES than in SC, matching the expected differentiation direction.

## Limitations

- vizAPA consumes APA results produced by Sierra or scAPAtrap — it does not identify poly(A) sites itself; data must be converted to PACdataset format first.
- RUD calculation only applies to genes with 3'UTR APA (get3UTRAPAds), and get3UTRAPApd requires minDist=50, maxDist=5000, minRatio=0.05 between proximal and distal pAs, so genes outside these constraints are excluded.
- Internal priming can confound pA interpretation; removePACdsIP filtering (e.g. six continuous As within up=-140/dn=10) should be applied, and scAPAtrap's recommended strategy retains only 3'UTR pAs supported by poly(A) tails or with sufficient expression (e.g. >55% of tail-supported pAs' total expression).
- vizTracks visual evidence (e.g. proximal site dominance in Asrgl1) suggests potential APA dynamics but is qualitative; differential APA claims should be backed by RUD/APA-marker statistics.

## Evidence

- [readme] vizTracks for a genome-browser-like plot displaying gene models, pA expression/usage, read alignments, and single-cell counts: [evidence span withheld — anchor status: external_doc]
- [methods] RUD decreases during sperm differentiation (SC -> RS -> ES): "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -\> RS -\> ES)"
- [methods] Negative ES-vs-SC APA markers mean shorter 3'UTR in ES: "The higher nagative markers between ES and SC means these markers have lower RUD scores (avg_log2FC<0, shorter 3'UTR) in ES than in SC"
- [methods] Sierra/scAPAtrap outputs feed vizAPA via PACdataset: [evidence span withheld — anchor status: not_found]
- [abstract] vizAPA enables differential APA usage identification: "vizAPA also enables identification of genes with differential APA usage across biological samples and/or cell types"
