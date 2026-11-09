---
name: macs2-accessible-peak-calling
description: Use when processing scATAC-seq fragments (e.g., BMMC dataset) after QC/doublet
  filtering and a peak-by-cell count matrix is needed before iterative LSI and Louvain
  clustering in the SCALA/ArchR pipeline.
when_to_use_negative:
- Input is scRNA-seq data — this step is specific to chromatin accessibility; scRNA-seq
  in SCALA uses variable-feature detection and PCA instead of MACS2 peak calling and
  LSI.
- Dataset is not human or mouse — SCALA currently supports analysis of only human
  and mouse datasets for both modalities.
- Dataset exceeds 2 GB on the online SCALA web server — use the desktop version, as
  some steps cannot be completed online (and > 50,000 cells is not supported online).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: MACS2
  role: Peak caller invoked underneath ArchR's reproducible peak-calling to define
    accessible chromatin peaks; a Conda system dependency of SCALA
- name: ArchR
  role: Orchestrates arrow-file creation, cell filtering (minTSS, minFrags), doublet
    removal, MACS2 reproducible peak calling, iterative LSI, SNN/Louvain clustering
    and UMAP
  repo: https://github.com/GreenleafLab/ArchR
- name: SCALA
  role: Web/desktop pipeline (R/Shiny) exposing create_arrow_file.R and the scATAC
    QC-through-UMAP workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:17d0b5903f5067c225eebda9ecc6d988a7c971b58609488c2a773d9ab1a22527
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# macs2-accessible-peak-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls accessible chromatin peaks from scATAC-seq fragment data with MACS2 via ArchR's reproducible peak-calling step inside the SCALA pipeline, producing the peak-count matrix used for LSI dimensionality reduction.

## When to use

Use when processing scATAC-seq fragments (e.g., BMMC dataset) after QC/doublet filtering and a peak-by-cell count matrix is needed before iterative LSI and Louvain clustering in the SCALA/ArchR pipeline.

## When NOT to use

- Input is scRNA-seq data — this step is specific to chromatin accessibility; scRNA-seq in SCALA uses variable-feature detection and PCA instead of MACS2 peak calling and LSI.
- Dataset is not human or mouse — SCALA currently supports analysis of only human and mouse datasets for both modalities.
- Dataset exceeds 2 GB on the online SCALA web server — use the desktop version, as some steps cannot be completed online (and > 50,000 cells is not supported online).

## Inputs

- scATAC-seq fragments file (e.g., BMMC scATAC-seq fragments, TSV/GZ)
- UCSC genome build reference (hg19, hg38, mm9, or mm10 BSgenome)
- ArchR Arrow file with filtered cells (minTSS=4, minFrags=1000, doublets removed)

## Outputs

- Reproducible accessible peak set (MACS2-called via ArchR)
- Peak-by-cell count matrix
- LSI embedding (30 dimensions, resolution 0.6, 4 iterations)
- UMAP projection colored by cluster identity (PNG), QC plots, and cluster abundance bar plot

## How to apply

First build an ArchR arrow file from the scATAC fragments file with SCALA's create_arrow_file.R, applying default cell filters (minTSS=4, minFrags=1000) and the appropriate UCSC genome build (hg19/hg38/mm9/mm10); verify per-cell QC metrics (TSS enrichment as the ratio of TSS signal to per-base-pair 2 kb flanking region enrichment, and log10(nFrags)) and remove putative doublets via ArchR's doublet-enrichment measurement with a chosen filterRatio. Then invoke ArchR's reproducible peak-calling step, which runs MACS2 underneath, to call accessible peaks across cell groups and construct the peak-count matrix. Downstream, run iterative LSI on genome-wide 500-bp tile counts (depth-normalized to 10,000, IDF and log transformed, resolution 0.6, 30 dimensions, 4 iterations), build a Shared-Nearest-Neighbor graph, partition with Louvain, and embed with UMAP. MACS2 is a Conda-installed system dependency of SCALA, so the environment must include it (plus Python 3.7+, libarrow-dev, hdf5) before any scATAC run.

## Related tools

- **MACS2** (Peak caller invoked underneath ArchR's reproducible peak-calling to define accessible chromatin peaks; a Conda system dependency of SCALA)
- **ArchR** (Orchestrates arrow-file creation, cell filtering (minTSS, minFrags), doublet removal, MACS2 reproducible peak calling, iterative LSI, SNN/Louvain clustering and UMAP) — https://github.com/GreenleafLab/ArchR
- **SCALA** (Web/desktop pipeline (R/Shiny) exposing create_arrow_file.R and the scATAC QC-through-UMAP workflow)

## Examples

```
Rscript create_arrow_file.R --fragments bmmc_fragments.tsv.gz --genome hg38 --minTSS 4 --minFrags 1000
```

## Evaluation signals

- The arrow file was built with default cell filters (minTSS=4, minFrags=1000) for the correct UCSC genome build (hg19/hg38/mm9/mm10).
- Per-cell QC plots of TSS enrichment (vs 2 kb flanking-region ratio) and log10(nFrags) were inspected and low-quality cells excluded before peak calling.
- A peak-count matrix is produced and iterative LSI completes at the specified settings (resolution 0.6, 30 dimensions, 4 iterations).
- SNN graph + Louvain clustering yields clusters and a UMAP projection comparable to the article's Fig. 3C style output; cluster abundance bar plot (Fig. 3B style) renders.
- MACS2 and other system dependencies (Python 3.7+, Arrow C++, hdf5) are confirmed installed in the Conda environment prior to the run.

## Limitations

- SCALA supports analysis of only human and mouse datasets for both scRNA-seq and scATAC-seq modalities.
- Online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session; datasets > 50,000 cells may fail to complete some steps online.
- Software version numbers for ArchR employed in SCALA are not stated, so reproducing exact peak calls may require matching the publication's environment.
- Seurat must be patched for PHATE support, and installation order matters; these modifications may lead to inconsistencies with other packages in the R installation.

## Evidence

- [other] workflow step 1: [evidence span withheld — anchor status: not_found]
- [other] workflow step 4: [evidence span withheld — anchor status: not_found]
- [other] LSI finding: [evidence span withheld — anchor status: not_found]
