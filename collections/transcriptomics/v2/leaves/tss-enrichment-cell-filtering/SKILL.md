---
name: tss-enrichment-cell-filtering
description: Use when starting a scATAC-seq analysis from a fragments file (e.g. BMMC
  data) and you must exclude low-quality cells — low transcription start site (TSS)
  enrichment or too few unique nuclear fragments — before building the peak-count
  matrix and running LSI.
when_to_use_negative:
- Input is scRNA-seq data — SCALA's RNA modality uses Seurat-based QC rather than
  TSS enrichment metrics.
- Cells have already been filtered and an ArchR Arrow project exists — reapplying
  minTSS/minFrags would duplicate QC.
- Analysis targets organisms other than human or mouse — SCALA supports only hg19/hg38/mm9/mm10
  genome builds.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: SCALA
  role: Pipeline that exposes the TSS/nFrags QC filtering step via its create_arrow_file.R
    script
- name: ArchR
  role: Builds Arrow files, computes TSS enrichment and nFrags QC metrics, and measures
    doublet enrichment for filtering
  repo: https://github.com/GreenleafLab/ArchR
- name: MACS2
  role: Called through ArchR's reproducible peak-calling step after QC filtering to
    construct the peak-count matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:a21830a71ad098c9187d8f2aa73e47618a1634b33a6bdb8f25d5ab34ad2c6a9c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/GreenleafLab/ArchR
  tools:
  - ArchR
  - MACS2
  - R
  - SCALA
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# TSS enrichment cell filtering (scATAC-seq QC)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control filter for scATAC-seq cells based on TSS enrichment and fragment-count thresholds, as implemented in SCALA/ArchR. It removes low-quality cells before dimensionality reduction so that downstream LSI, clustering, and UMAP reflect accessible chromatin rather than technical noise.

## When to use

Use when starting a scATAC-seq analysis from a fragments file (e.g. BMMC data) and you must exclude low-quality cells — low transcription start site (TSS) enrichment or too few unique nuclear fragments — before building the peak-count matrix and running LSI.

## When NOT to use

- Input is scRNA-seq data — SCALA's RNA modality uses Seurat-based QC rather than TSS enrichment metrics.
- Cells have already been filtered and an ArchR Arrow project exists — reapplying minTSS/minFrags would duplicate QC.
- Analysis targets organisms other than human or mouse — SCALA supports only hg19/hg38/mm9/mm10 genome builds.

## Inputs

- scATAC-seq fragments file (e.g. BMMC scATAC-seq fragments.tsv.gz)
- UCSC genome build annotation (hg19, hg38, mm9, or mm10)

## Outputs

- ArchR arrow file containing only cells passing minTSS=4 and minFrags=1000
- Per-cell QC plots of TSS enrichment vs. log10(nFrags) (Fig. 3A style PNG)
- Doublet-filtered Arrow project ready for MACS2 peak calling and LSI

## How to apply

Build an ArchR arrow file from the scATAC-seq fragments file with SCALA's create_arrow_file.R script, applying the default cell filters minTSS=4 and minFrags=1000 and the appropriate UCSC genome build (hg19/hg38/mm9/mm10). Compute per-cell QC metrics: TSS enrichment, calculated as the ratio of TSS signal to per-base-pair 2 kb flanking region enrichment, and the number of unique nuclear fragments (log10(nFrags)). Inspect the resulting QC plots (Fig. 3A style) and exclude cells failing the thresholds. Remove putative doublets, detected via ArchR's doublet-enrichment measurement, using a chosen filterRatio before proceeding to MACS2 peak calling and LSI.

## Related tools

- **SCALA** (Pipeline that exposes the TSS/nFrags QC filtering step via its create_arrow_file.R script)
- **ArchR** (Builds Arrow files, computes TSS enrichment and nFrags QC metrics, and measures doublet enrichment for filtering) — https://github.com/GreenleafLab/ArchR
- **MACS2** (Called through ArchR's reproducible peak-calling step after QC filtering to construct the peak-count matrix)

## Examples

```
Rscript create_arrow_file.R --fragments bmmc.fragments.tsv.gz --genome hg38 --minTSS 4 --minFrags 1000
```

## Evaluation signals

- All retained cells have TSS enrichment ≥ 4 (minTSS=4) and ≥ 1000 unique nuclear fragments (minFrags=1000).
- QC plot of TSS enrichment vs. log10(nFrags) shows the excluded cells falling below the threshold lines.
- Putative doublets identified by ArchR's doublet-enrichment measurement are removed at the chosen filterRatio.
- Downstream iterative LSI (resolution 0.6, 30 dimensions, 4 iterations) runs without QC-flagged low-quality cells influencing clustering.

## Limitations

- SCALA supports analysis of only human and mouse datasets (hg19/hg38/mm9/mm10 genome builds).
- In the online SCALA version, dataset size is restricted to < 2 GB with no more than 2 CPU threads per session, and some steps cannot be completed for datasets larger than 50,000 cells.
- The default minTSS=4/minFrags=1000 cut-offs are task-defined defaults; an inappropriate filterRatio for doublet removal can distort cluster abundance.

## Evidence

- [methods] Default TSS/nFrags thresholds and genome-build options: [evidence span withheld — anchor status: not_found]
- [methods] Definition of the TSS enrichment QC metric: [evidence span withheld — anchor status: not_found]
