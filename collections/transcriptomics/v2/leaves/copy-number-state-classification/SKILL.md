---
name: copy-number-state-classification
description: Use when scRNA-seq count data from a tumor sample has been segmented
  (e.g. via getBreaksVegaMC/computeCNAmtx) and each segment must be assigned an integer
  CNV state label for tumor/normal or clonal analysis.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
  — the aneuploidy-based assumption underlying the five-state model may not be suited
  in this case
- Bulk DNA copy number data — CNVkit-style integer cutoffs (-1, -0.25, 0.2, 0.7) apply
  to bulk segmented log-ratios, not the expression-based five-state mixture used here
- Input that has not been segmented and baseline-corrected — the mixture model operates
  on per-cell segment means relative to the confident-normal-cell median, not raw
  counts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCEVAN
  role: Implements the five-state truncated-normal mixture (EM) classification of
    segment means and the majority-vote consensus, exposed via pipelineCNA
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Identifies confident normal cells whose median expression defines the copy
    number baseline removed before classification
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:c7780c4fc125d112712f287e943dfb7ac657a452c2fbd61966bcdbd0007d0d0a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# copy-number-state-classification

## Summary

Classify each genomic segment of a scRNA-seq sample into one of five copy number states (deletion, loss, neutral, gain, amplification) by fitting a five-component truncated-normal mixture to per-cell segment means via EM, then taking a majority vote across cells to form a consensus per-segment CNV call.

## When to use

Use when scRNA-seq count data from a tumor sample has been segmented (e.g. via getBreaksVegaMC/computeCNAmtx) and each segment must be assigned an integer CNV state label for tumor/normal or clonal analysis.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) — the aneuploidy-based assumption underlying the five-state model may not be suited in this case
- Bulk DNA copy number data — CNVkit-style integer cutoffs (-1, -0.25, 0.2, 0.7) apply to bulk segmented log-ratios, not the expression-based five-state mixture used here
- Input that has not been segmented and baseline-corrected — the mixture model operates on per-cell segment means relative to the confident-normal-cell median, not raw counts

## Inputs

- Raw scRNA-seq count matrix (genes on rows, cells on columns; gene symbols or Ensembl IDs), e.g. MGH106 from GSE131928 loaded from MGH106_data.RData
- Genomic segment definition (segmentation breakpoints from getBreaksVegaMC / computeCNAmtx, e.g. MGH106_CNAmtx.RData)
- Gene-to-genomic-location annotation (e.g. MGH106_count_mtx_annot.RData)

## Outputs

- Per-segment consensus copy number state label, each segment carrying exactly one of: deletion, loss, neutral, gain, amplification
- Per-cell segment CNV mean matrix (CNA_mtx_relat) usable for plotting single-cell CN profiles
- pipelineCNA results data frame with per-cell tumor/normal classification and subclone assignment

## How to apply

Starting from the count matrix (genes x cells), compute per-cell segment mean expression over genomic segments defined by the joint segmentation breaks (getBreaksVegaMC / computeCNAmtx), after the copy number baseline (median of confident normal cells identified via yaGST with P < 1e-10 and NES > 1.0) has been subtracted. Model the per-cell segment means as a mixture of five truncated normal distributions and estimate mixture parameters by EM, initialized from empirically chosen fixed parameters (Supplementary Table 1). Assign each segment to exactly one of the five states — deletion, loss, neutral, gain, amplification — per cell. Aggregate the per-cell calls by majority vote across cells to obtain the consensus segment-level CNV call. The segmentation smoothing parameter beta_vega (default 0.5, higher = coarser segmentation) determines the granularity of the segments being classified.

## Related tools

- **SCEVAN** (Implements the five-state truncated-normal mixture (EM) classification of segment means and the majority-vote consensus, exposed via pipelineCNA) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Identifies confident normal cells whose median expression defines the copy number baseline removed before classification) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE)
```

## Evaluation signals

- The output table exists and every segment carries exactly one of the five state labels (deletion, loss, neutral, gain, amplification)
- The consensus segment calls are consistent with the per-cell tumor/normal classification returned by the pipelineCNA results data frame
- Per-cell segment means were computed relative to the baseline (median of confident normal cells with P < 1e-10 and NES > 1.0), or a synthetic gaussian baseline was removed if the sample was deemed pure
- EM convergence from the fixed initial parameters of Supplementary Table 1 yields a valid five-component mixture assignment
- Called deletions/gains visually match inferred CN ratios plotted from the .seg / CNA_mtx_relat regions (e.g. FeaturePlot of chr3 mean)

## Limitations

- Initial mixture parameters are deferred to Supplementary Table 1 and are not stated in the main text, complicating exact reimplementation
- The beta constant (the only segmentation parameter, default 0.5 in the README) trades off segmentation granularity; higher beta gives more coarse-grained segments
- Ground truth from WES/WGS bulk has different resolution than single-cell data, requiring resampling before comparison of CNV calls
- Aneuploidy-based approach may not suit tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas

## Evidence

- [methods] pipelineCNA single-call classification: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] baseline removal before classification: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [discussion] EM initialization from fixed parameters: "starting from empirically chosen initial fixed parameters (Supplementary Table 1)."
- [discussion] segmentation parameter beta: "where β is a positive constant, representing the only parameter of the segmentation algorithm."
- [readme] README on pipelineCNA scope: [evidence span withheld — anchor status: external_doc]
