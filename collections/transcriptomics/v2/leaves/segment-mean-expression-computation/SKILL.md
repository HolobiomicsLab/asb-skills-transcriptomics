---
name: segment-mean-expression-computation
description: Use when a raw scRNA-seq count matrix (e.g. MGH106 from GSE131928) must
  be aggregated from gene-level expression to per-cell segment mean values over the
  breakpoints shared across cells, as input to downstream CNV state classification.
when_to_use_negative:
- If the goal is clone phylogeny or subclone comparison across samples (use multiSampleComparisonClonalCN
  instead of raw segment means alone)
- If the tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers,
  ependymomas), since aneuploidy-based inference from segment means is unsuitable
- If matched tumor-normal DNA data already provides a segmented CNV profile, bypassing
  expression-based segment means
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCEVAN
  role: Provides getBreaksVegaMC for joint segmentation and computeCNAmtx for per-cell
    segment mean computation within pipelineCNA
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:4c3980db14165a8e2f518efb2e8ad411de909986fde3b152d80ce56957be1e61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# segment-mean-expression-computation

## Summary

Compute per-cell mean expression over genomic segments defined by SCEVAN's joint segmentation breaks, producing the CNA matrix whose smoothed segment means are modeled as a mixture of five truncated normal distributions for copy number state classification.

## When to use

Use when a raw scRNA-seq count matrix (e.g. MGH106 from GSE131928) must be aggregated from gene-level expression to per-cell segment mean values over the breakpoints shared across cells, as input to downstream CNV state classification.

## When NOT to use

- If the goal is clone phylogeny or subclone comparison across samples (use multiSampleComparisonClonalCN instead of raw segment means alone)
- If the tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, ependymomas), since aneuploidy-based inference from segment means is unsuitable
- If matched tumor-normal DNA data already provides a segmented CNV profile, bypassing expression-based segment means

## Inputs

- Raw scRNA-seq count matrix with genes on rows (Gene Symbol or Ensembl ID) and cells on columns
- Genomic segment breakpoints from SCEVAN's joint multichannel segmentation (getBreaksVegaMC)
- Gene-to-genomic-location annotation (e.g. MGH106_count_mtx_annot.RData)
- Baseline-adjusted relative expression matrix (median of confident normal cells subtracted)

## Outputs

- Per-cell segment mean expression matrix (CNA matrix, e.g. MGH106_CNAmtx.RData)
- Segment means used as input to the five-state mixture model (deletion, loss, neutral, gain, amplification)

## How to apply

First preprocess the count matrix: remove cells with fewer than 200 detected genes, remove genes expressed in less than 1% of cells, filter REACTOME cell cycle genes, and sort remaining genes by genomic coordinates using an Ensembl-based annotation. Run SCEVAN's joint segmentation (getBreaksVegaMC, with beta_vega, default 0.5, controlling segmentation granularity) to define segment boundaries shared across all cells. Then compute, per cell, the mean of the smoothed relative expression over all genes falling within each segment (computeCNAmtx), yielding the CNA matrix (e.g. MGH106_CNAmtx.RData). These per-cell segment means are subsequently modeled as a mixture of five truncated normal distributions fitted by EM, so the segment-mean step must be computed consistently with the joint breakpoints, not per-cell segmentation.

## Related tools

- **SCEVAN** (Provides getBreaksVegaMC for joint segmentation and computeCNAmtx for per-cell segment mean computation within pipelineCNA) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Every segment in the CNA matrix carries exactly one of the five state labels (deletion, loss, neutral, gain, amplification) after classification
- Segment boundaries match the joint segmentation breaks (getBreaksVegaMC) shared across all cells, not per-cell segmentations
- Segment means are computed from the relative expression matrix with the median-based normal cell baseline already subtracted
- Per-cell tumor/normal classification returned by the pipelineCNA results data frame is consistent with the segment-level calls
- Loadable output artifacts exist (e.g. MGH106_CNAmtx.RData, MGH106_count_mtx_annot.RData in the output directory)

## Limitations

- Initial parameters of the downstream mixture model are empirically chosen fixed values deferred to Supplementary Table 1 and not included in the main text
- Sparse droplet-based scRNA-seq coverage makes segment mean estimates noisy; methods like inferCNV perform sub-optimally in this regime, and the baseline subtraction depends on identifying confident normal cells (P < 1e-10, NES > 1.0)
- If no confident normal cells are found, the sample is assumed pure and a synthetic gaussian baseline is removed instead

## Evidence

- [methods] Compute per-cell segment mean expression over the genomic segments defined by the joint segmentation breaks (getBreaksVegaMC / computeCNAmtx): [evidence span withheld — anchor status: not_found]
- [methods] Mixture of five truncated normal distributions: [evidence span withheld — anchor status: not_found]
- [discussion] Pre-filtering of cells and genes: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] Baseline subtraction producing relative matrix: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [readme] README CNA matrix artifact: [evidence span withheld — anchor status: external_doc]
- [readme] README pipeline entry point: [evidence span withheld — anchor status: external_doc]
