---
name: archr-arrow-file-generation
description: Use when raw scATAC-seq fragment files (e.g., BMMC fragments.tsv.gz)
  must be converted into an ArchR arrow file with QC-filtered cells and reduced to
  an LSI latent space for clustering and UMAP visualization.
when_to_use_negative:
- Input is scRNA-seq data rather than scATAC-seq — the arrow-file/LSI route is specific
  to chromatin accessibility
- Dataset is from an organism other than human or mouse — SCALA supports analysis
  of only human and mouse datasets
- Dataset is larger than 50,000 cells and only the online SCALA version is available
  — some steps cannot be completed in the web version
tools:
- name: SCALA
  role: Wraps the scATAC-seq pipeline (create_arrow_file.R script) that builds ArchR
    arrow files and runs LSI
- name: ArchR
  role: Provides arrow-file creation, doublet-enrichment measurement, reproducible
    MACS2 peak calling, and iterative LSI on 500-bp genome-wide tiles
  repo: https://github.com/GreenleafLab/ArchR
- name: MACS2
  role: Called through ArchR to identify accessible peaks and construct the peak-count
    matrix
- name: UMAP
  role: Embeds the Louvain-clustered cells from LSI space in 2D
- name: R
  role: Implementation language for SCALA's scATAC-seq module and ArchR scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:c434bf0d79c8433ababb51b8c67d00a7a5d9ac98d239ce5b89f4379e169a82a1
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# ArchR arrow file generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate ArchR arrow files from scATAC-seq fragments and perform cell-level QC, doublet removal, MACS2 peak calling, and iterative LSI dimensionality reduction as implemented in SCALA's scATAC-seq pipeline.

## When to use

Use when raw scATAC-seq fragment files (e.g., BMMC fragments.tsv.gz) must be converted into an ArchR arrow file with QC-filtered cells and reduced to an LSI latent space for clustering and UMAP visualization.

## When NOT to use

- Input is scRNA-seq data rather than scATAC-seq — the arrow-file/LSI route is specific to chromatin accessibility
- Dataset is from an organism other than human or mouse — SCALA supports analysis of only human and mouse datasets
- Dataset is larger than 50,000 cells and only the online SCALA version is available — some steps cannot be completed in the web version

## Inputs

- scATAC-seq fragments file (e.g., BMMC fragments.tsv.gz)
- UCSC genome build annotation (hg19, hg38, mm9, or mm10)
- filterRatio parameter for doublet removal

## Outputs

- ArchR arrow file with QC-filtered cells
- Per-cell QC metric plots (TSS enrichment, log10(nFrags)) as PNG files
- MACS2 peak-count matrix
- LSI dimensionality-reduced embedding (30 dimensions)
- Louvain cluster assignments and 2D UMAP projection colored by cluster identity (Fig. 3C style PNG)

## How to apply

Build the ArchR arrow file from the scATAC-seq fragments file using SCALA's create_arrow_file.R script, applying the default cell filters (minTSS=4, minFrags=1000) and the appropriate UCSC genome build (hg19/hg38/mm9/mm10). Compute per-cell QC metrics — TSS enrichment (ratio of TSS signal to per-base-pair 2 kb flanking region enrichment) and log10(nFrags) — inspect the QC plots, and exclude low-quality cells. Identify putative doublets via ArchR's doublet-enrichment measurement and remove them with a chosen filterRatio. Call accessible peaks with MACS2 through ArchR's reproducible peak-calling step to construct the peak-count matrix. Run iterative LSI on genome-wide 500-bp tile counts normalized to a constant of 10,000 with inverse document frequency normalization and log transformation, re-iterating on the most variable tiles across low-resolution clusters (resolution 0.6, 30 dimensions, 4 iterations). Finally, build a Shared-Nearest-Neighbor graph in LSI space, partition it with the Louvain algorithm, and embed cells in 2D with UMAP.

## Related tools

- **SCALA** (Wraps the scATAC-seq pipeline (create_arrow_file.R script) that builds ArchR arrow files and runs LSI)
- **ArchR** (Provides arrow-file creation, doublet-enrichment measurement, reproducible MACS2 peak calling, and iterative LSI on 500-bp genome-wide tiles) — https://github.com/GreenleafLab/ArchR
- **MACS2** (Called through ArchR to identify accessible peaks and construct the peak-count matrix)
- **UMAP** (Embeds the Louvain-clustered cells from LSI space in 2D)
- **R** (Implementation language for SCALA's scATAC-seq module and ArchR scripts)

## Examples

```
Rscript create_arrow_file.R --fragments bmmc_fragments.tsv.gz --genome hg19 --minTSS 4 --minFrags 1000
```

## Evaluation signals

- Cell filters minTSS=4 and minFrags=1000 were applied when building the arrow file
- QC plots show TSS enrichment computed as ratio of TSS signal to per-base-pair 2 kb flanking region enrichment and log10(nFrags) per cell
- LSI was run with resolution 0.6, 30 dimensions, and 4 iterations on 500-bp tiles normalized to 10,000 with TF-IDF and log transformation
- A peak-count matrix exists derived from MACS2 reproducible peak calling
- UMAP projection colored by Louvain cluster identity and a cluster abundance bar plot are exported as PNG files

## Limitations

- SCALA supports analysis of only human and mouse datasets in both modalities
- The online SCALA version restricts dataset size to < 2 GB with no more than 2 CPU threads per session; for datasets > 50,000 cells the desktop version is required as some steps cannot complete online
- SCALA requires modifications to the Seurat package for phate/phateR compatibility, which may lead to inconsistencies with other packages in the R installation

## Evidence

- [abstract] we present SCALA, a bioinformatics tool for analysis and visualization of single-cell RNA sequencing (scRNA-seq) and Assay for Transposase-Accessible Chromatin using sequencing (scATAC-seq) datasets: "we present SCALA, a bioinformatics tool for analysis and visualization of single-cell RNA sequencing (scRNA-seq) and Assay for Transposase-Accessible Chromatin using sequencing (scATAC-seq) datasets"
- [methods] It is worth noting that in both modalities, the analysis of only human and mouse datasets is currently supported.: "It is worth noting that in both modalities, the analysis of only human and mouse datasets is currently supported."
- [readme] A web tool that handles the analysis of scRNAseq data, from quality control and normalization, to dimensionality reduction, differential expression analysis, clustering and visualization.: "A web tool that handles the analysis of scRNAseq data, from quality control and normalization, to dimensionality reduction, differential expression analysis, clustering and visualization."
