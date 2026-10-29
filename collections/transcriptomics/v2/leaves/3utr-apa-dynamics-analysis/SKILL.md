---
name: 3utr-apa-dynamics-analysis
description: Use when analyzing polyA-captured scRNA-seq data (Sierra or scAPAtrap
  PACdataset) to detect and visualize 3'UTR shortening/lengthening dynamics across
  cell states or differentiation trajectories.
when_to_use_negative:
- The input is bulk APA data or already-summarized usage tables rather than per-cell
  poly(A) site counts from polyA-captured scRNA-seq — RUD per-cell gradients require
  single-cell PACdataset input.
- The pA sites of interest are not in 3'UTRs — only genes with 3'UTR APA can be used
  for RUD calculation, so intronic/intergenic events need other analyses.
- Cell embeddings (UMAP_1/UMAP_2) cannot be computed or cell-type annotations are
  absent, since vizUMAP overlay and trajectory ordering both depend on per-cell metadata.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: vizAPA
  role: 'Core R package: imports Sierra/scAPAtrap outputs into PACdataset, computes
    RUD index via getAPAindexPACds (choose2PA=''PD''), and renders vizUMAP gradient
    overlays of per-cell RUD scores'
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Provides PACdataset structures and helper functions such as get3UTRAPApd for
    selecting proximal-distal 3'UTR pA pairs used in RUD calculation
- name: Sierra
  role: Upstream identification and quantification of poly(A) site peaks from polyA-captured
    scRNA-seq BAM files, whose output is read into vizAPA
  repo: https://github.com/VCCRI/Sierra
- name: scAPAtrap
  role: Alternative upstream tool for identifying and quantifying poly(A) sites from
    3'-tag scRNA-seq; its scAPAtrapData.rda is converted to a PACdataset for RUD analysis
  repo: https://github.com/BMILAB/scAPAtrap
- name: Seurat
  role: 'Optional: computes 2D embeddings from the pA count matrix in the PACdataset
    when no precomputed UMAP coordinates exist'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:417a31523ac15f52323f3d01df790a957ff3d4d56a2354e4a59e17d9235d8496
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Seurat
  - Sierra
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# 3utr-apa-dynamics-analysis

## Summary

Quantify 3'UTR shortening/lengthening dynamics (APA) at single-cell resolution by computing per-gene, per-cell RUD (Relative Usage of Distal poly(A) site) indices on 3'UTR poly(A) sites with vizAPA/movAPA, then visualizing global APA dynamics as RUD-score gradients overlaid on 2D UMAP embeddings.

## When to use

Use when analyzing polyA-captured scRNA-seq data (Sierra or scAPAtrap PACdataset) to detect and visualize 3'UTR shortening/lengthening dynamics across cell states or differentiation trajectories.

## When NOT to use

- The input is bulk APA data or already-summarized usage tables rather than per-cell poly(A) site counts from polyA-captured scRNA-seq — RUD per-cell gradients require single-cell PACdataset input.
- The pA sites of interest are not in 3'UTRs — only genes with 3'UTR APA can be used for RUD calculation, so intronic/intergenic events need other analyses.
- Cell embeddings (UMAP_1/UMAP_2) cannot be computed or cell-type annotations are absent, since vizUMAP overlay and trajectory ordering both depend on per-cell metadata.

## Inputs

- Sierra MEX output (matrix.mtx.gz, barcodes.tsv.gz) or scAPAtrap scAPAtrapData.rda peak counts from polyA-captured scRNA-seq BAM files
- Genome annotation GTF (e.g., Homo_sapiens.GRCh38.109.gtf.gz)
- Cell type annotation / colData table with per-cell metadata (e.g., celltype labels)
- Optionally a BSgenome object (e.g., BSgenome.Hsapiens.NCBI.GRCh38) for internal-priming filtering

## Outputs

- Per-gene, per-cell RUD PACdataset (RUD index for proximal-distal 3'UTR pA pairs)
- vizUMAP gradient-color plot of mean per-cell RUD scores on 2D embeddings showing 3'UTR shortening transitions across cell types
- APA markers with differential APA usage between cell groups (e.g., ES vs SC, with avg_log2FC sign indicating lower/higher RUD)

## How to apply

Convert Sierra or scAPAtrap outputs into a PACdataset with vizAPA::readPACds, annotate poly(A) sites with annotatePAC against a GTF-derived annoHub, and subset to 3'UTR features (scPACds@anno$ftr=='3UTR') since only genes with 3'UTR APA can be used for RUD calculation. Select proximal-distal pA pairs (e.g., movAPA::get3UTRAPApd with minDist=50, maxDist=5000, minRatio=0.05) and compute the per-gene, per-cell RUD index with vizAPA::getAPAindexPACds using choose2PA='PD'. Compute 2D embeddings with reduceDim (e.g., nfeature=2000, dims=1:10) and overlay mean per-cell RUD scores as gradient color on the embeddings via vizUMAP(RUD, group='celltype', xcol='UMAP_1', ycol='UMAP_2'). Relevel cell-type factors to a biologically meaningful order (e.g., SC -> RS -> ES for sperm differentiation) so the gradient interpretation reflects the trajectory. Optionally identify APA markers between cell types with vizAPAmarkers and inspect gene-level RUD (e.g., Asrgl1) to confirm directionality of shortening.

## Related tools

- **vizAPA** (Core R package: imports Sierra/scAPAtrap outputs into PACdataset, computes RUD index via getAPAindexPACds (choose2PA='PD'), and renders vizUMAP gradient overlays of per-cell RUD scores) — https://github.com/BMILAB/vizAPA
- **movAPA** (Provides PACdataset structures and helper functions such as get3UTRAPApd for selecting proximal-distal 3'UTR pA pairs used in RUD calculation)
- **Sierra** (Upstream identification and quantification of poly(A) site peaks from polyA-captured scRNA-seq BAM files, whose output is read into vizAPA) — https://github.com/VCCRI/Sierra
- **scAPAtrap** (Alternative upstream tool for identifying and quantifying poly(A) sites from 3'-tag scRNA-seq; its scAPAtrapData.rda is converted to a PACdataset for RUD analysis) — https://github.com/BMILAB/scAPAtrap
- **Seurat** (Optional: computes 2D embeddings from the pA count matrix in the PACdataset when no precomputed UMAP coordinates exist)

## Examples

```
data(scPACds, package='vizAPA'); scPACds<-scPACds[scPACds@anno$ftr=='3UTR']; RUD=getAPAindexPACds(scPACds, choose2PA='PD'); vizUMAP(RUD, group='celltype', xcol='UMAP_1', ycol='UMAP_2')
```

## Evaluation signals

- The vizUMAP output figure exists and exhibits the reported gradual transition of decreased RUD scores (3'UTR shortening) along SC -> RS -> ES.
- The RUD PACdataset contains per-gene, per-cell RUD values only for genes with 3'UTR APA (subset to scPACds@anno$ftr=='3UTR' before index computation).
- APA markers between cell types show expected sign structure: most ES-vs-SC markers are negative (lower RUD in ES, avg_log2FC<0, shorter 3'UTR).
- Known trajectory genes such as Asrgl1 show decreased RUD score from SC to ES, consistent with APA dynamics during mouse spermatogenesis.
- Mean per-cell RUD scores computed from the RUD PACdataset align with celltype labels stored in colData used for the gradient overlay.

## Limitations

- Only genes with 3'UTR APA can be used for RUD calculation, so non-3'UTR APA events are excluded from this analysis.
- The mouse sperm demo dataset is small (974 pAs from 413 genes); RUD per-cell estimates on sparse single-cell data may be noisy for lowly expressed genes.
- vizUMAP requires precomputed UMAP_1/UMAP_2 coordinates in colData or embeddings computed via reduceDim/Seurat, adding a dependency on embedding quality.

## Evidence

- [methods] RUD on 3'UTR features: "scPACds<-scPACds[scPACds@anno$ftr=="3UTR"]"
- [methods] RUD index with proximal-distal pair selection: "iPACds=getAPAindexPACds(scPACds, choose2PA = "PD")"
- [methods] Proximal-distal pA selection thresholds: "pd=movAPA::get3UTRAPApd(pacds=scPACds, minDist=50, maxDist=5000, minRatio=0.05"
- [methods] Reported finding: 3'UTR shortening gradient during spermatogenesis: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES)."
- [readme] vizUMAP module purpose: [evidence span withheld — anchor status: not_found]
- [methods] Embedding computation parameters: "scPACds=reduceDim(scPACds, nfeature=2000, dims=1:10, dimLabel='umap_norm', norm=TRUE)"
