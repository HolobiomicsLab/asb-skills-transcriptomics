---
name: scatac-seq-fragment-quality-control
description: Use when starting an scATAC-seq analysis from a fragments file (e.g.
  BMMC dataset) and low-quality cells, doublets, and high-dimensional tile/peak matrices
  must be filtered and reduced before clustering.
when_to_use_negative:
- Input is already a filtered, dimensionality-reduced scATAC-seq embedding or peak-count
  matrix with cell QC previously applied
- Data are scRNA-seq rather than scATAC-seq — SCALA's RNA module uses PCA (e.g. 25
  PCs for the SF dataset), not fragment QC and LSI on 500-bp tiles
- Non-human, non-mouse datasets — SCALA currently supports only human and mouse for
  both modalities
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SCALA
  role: Wraps the whole QC and LSI workflow behind create_arrow_file.R and its Shiny
    interface
- name: ArchR
  role: Arrow-file construction, TSS/nFrags QC metrics, doublet identification, peak
    calling, iterative LSI, SNN/Louvain clustering and UMAP
  repo: https://github.com/GreenleafLab/ArchR
- name: MACS2
  role: Accessible peak calling used to build the peak-count matrix after cell filtering
- name: UMAP
  role: 2D embedding of cells in the LSI/Louvain cluster space
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:f20e9c4ecf334edefe9b058b0312263186da9e776103ea77db01b92557152e55
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
  - UMAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# scatac-seq-fragment-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs cell-level quality control on scATAC-seq fragment data (TSS enrichment and unique nuclear fragment counts) within SCALA's ArchR-based pipeline, and reduces dimensionality of filtered chromatin accessibility data using latent semantic indexing (LSI).

## When to use

Use when starting an scATAC-seq analysis from a fragments file (e.g. BMMC dataset) and low-quality cells, doublets, and high-dimensional tile/peak matrices must be filtered and reduced before clustering.

## When NOT to use

- Input is already a filtered, dimensionality-reduced scATAC-seq embedding or peak-count matrix with cell QC previously applied
- Data are scRNA-seq rather than scATAC-seq — SCALA's RNA module uses PCA (e.g. 25 PCs for the SF dataset), not fragment QC and LSI on 500-bp tiles
- Non-human, non-mouse datasets — SCALA currently supports only human and mouse for both modalities

## Inputs

- scATAC-seq fragments file (e.g. BMMC fragments)
- Reference genome build selection (UCSC hg19, hg38, mm9, or mm10)
- Genome annotations required by ArchR arrow file construction

## Outputs

- ArchR arrow file of filtered cells
- Per-cell QC plots (TSS enrichment vs log10(nFrags), Fig. 3A style)
- Peak-count matrix from MACS2 reproducible peak calling
- LSI dimensionality-reduced cell embeddings (30 dimensions)
- Louvain cluster assignments and UMAP projection colored by cluster (Fig. 3C style, PNG)

## How to apply

Build an ArchR arrow file from the scATAC-seq fragments file with SCALA's create_arrow_file.R, applying default cell filters minTSS=4 and minFrags=1000 and the appropriate UCSC genome build (hg19/hg38/mm9/mm10). Compute per-cell QC metrics — TSS enrichment (ratio of TSS signal to per-base-pair 2 kb flanking region enrichment) and log10(nFrags) — inspect QC plots, and remove low-quality cells and doublets (ArchR doublet-enrichment with a chosen filterRatio). Call accessible peaks with MACS2 via ArchR reproducible peak-calling to construct the peak-count matrix, then run iterative LSI on genome-wide 500-bp tile counts normalized to a constant of 10,000 with inverse document frequency normalization and log transformation, re-iterating on the most variable tiles. Apply LSI with resolution 0.6, 30 dimensions, and 4 iterations (otherwise default settings), build a shared-nearest-neighbor graph, cluster with Louvain, and embed cells in 2D with UMAP.

## Related tools

- **SCALA** (Wraps the whole QC and LSI workflow behind create_arrow_file.R and its Shiny interface)
- **ArchR** (Arrow-file construction, TSS/nFrags QC metrics, doublet identification, peak calling, iterative LSI, SNN/Louvain clustering and UMAP) — https://github.com/GreenleafLab/ArchR
- **MACS2** (Accessible peak calling used to build the peak-count matrix after cell filtering)
- **UMAP** (2D embedding of cells in the LSI/Louvain cluster space)

## Examples

```
Rscript create_arrow_file.R --fragments bmmc.fragments.tsv.gz --genome hg38 --minTSS 4 --minFrags 1000
```

## Evaluation signals

- QC filtering thresholds were applied as specified: minTSS=4 and minFrags=1000 on the arrow file
- LSI ran with resolution 0.6, 30 dimensions, 4 iterations on 500-bp tiles normalized to 10,000, with IDF normalization and log transformation
- Doublets were removed via ArchR doublet-enrichment with a recorded filterRatio value
- QC plots in Fig. 3A style and UMAP colored by cluster identity in Fig. 3C style were produced as PNG files
- Cluster abundance bar plot (Fig. 3B style) reproduces the expected cluster composition of the BMMC dataset

## Limitations

- SCALA supports analysis of only human and mouse datasets for both modalities
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session, and cannot complete some steps for datasets larger than 50,000 cells
- Software version numbers for Seurat and ArchR employed in SCALA are not stated in the article

## Evidence

- [other] Following scATAC-seq QC filtering, SCALA's scATAC-seq pipeline employs latent semantic indexing (LSI) with a resolution of 0.6, 30 dimensions, and 4 iterations: [evidence span withheld — anchor status: not_found]
