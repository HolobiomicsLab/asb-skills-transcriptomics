---
name: lsi-dimensionality-reduction-scatac
description: Use when raw scATAC-seq fragment files (BAMMC-style fragments, hg19/hg38/mm9/mm10)
  must be converted to an ArchR arrow file, QC-filtered, peak-called with MACS2, and
  reduced to a low-dimensional LSI representation for clustering and UMAP embedding.
when_to_use_negative:
- The input is scRNA-seq transcriptomic data — SCALA's scRNA-seq path uses PCA on
  variable genes (e.g., 25 PCs, 1535 variable genes), not LSI on 500-bp tiles
- The dataset is from an organism other than human or mouse — SCALA supports analysis
  of only human and mouse datasets for both modalities
- You already hold a processed feature/cell matrix or a fitted embedding — this skill
  starts from raw fragments and arrow-file construction
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SCALA
  role: Interactive pipeline wrapping the scATAC-seq QC, MACS2 peak calling, and LSI/UMAP
    workflow (create_arrow_file.R and companion scripts)
  repo: http://scala.pavlopouloslab.info
- name: ArchR
  role: Arrow-file construction, TSS/nFrags QC, doublet identification, reproducible
    MACS2 peak calling, iterative LSI, SNN/Louvain clustering, and UMAP embedding
  repo: https://github.com/GreenleafLab/ArchR
- name: MACS2
  role: Called accessible peaks from filtered fragments to build the peak-count matrix
    feeding LSI
- name: R
  role: Language in which SCALA (R, Shiny, JavaScript) and the ArchR-based scATAC
    module are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:85451eebe29d24ecc338f76ac58e010d8a8dfa6d203969ce6b5635efe3291879
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - ArchR
  - MACS2
  - R
  - SCALA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lsi-dimensionality-reduction-scatac

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SCALA's scATAC-seq dimensionality-reduction skill: after QC filtering (minTSS=4, minFrags=1000) and MACS2 peak calling via ArchR, it runs iterative latent semantic indexing (LSI, resolution 0.6, 30 dimensions, 4 iterations) on genome-wide 500-bp tile counts, then clusters cells with SNN/Louvain and embeds them with UMAP.

## When to use

Use when raw scATAC-seq fragment files (BAMMC-style fragments, hg19/hg38/mm9/mm10) must be converted to an ArchR arrow file, QC-filtered, peak-called with MACS2, and reduced to a low-dimensional LSI representation for clustering and UMAP embedding.

## When NOT to use

- The input is scRNA-seq transcriptomic data — SCALA's scRNA-seq path uses PCA on variable genes (e.g., 25 PCs, 1535 variable genes), not LSI on 500-bp tiles
- The dataset is from an organism other than human or mouse — SCALA supports analysis of only human and mouse datasets for both modalities
- You already hold a processed feature/cell matrix or a fitted embedding — this skill starts from raw fragments and arrow-file construction

## Inputs

- scATAC-seq fragments file (e.g., BMMCs scATAC-seq dataset fragments)
- UCSC genome build annotation (hg19, hg38, mm9, or mm10)
- ArchR filter parameters (minTSS=4, minFrags=1000, filterRatio)
- MACS2-called accessible peak set used to build the peak-count matrix

## Outputs

- ArchR arrow file of QC-passed, doublet-filtered cells
- Per-cell QC metrics and QC plots (Fig. 3A style, PNG)
- Peak-count matrix over MACS2-called peaks
- 30-dimensional LSI embedding of cells
- Louvain cluster assignments and cluster abundance bar plot (Fig. 3B style)
- 2D UMAP projection colored by cluster identity (Fig. 3C style, PNG)

## How to apply

Build an ArchR arrow file from the fragments file with SCALA's create_arrow_file.R, applying the default cell filters (minTSS=4, minFrags=1000) and the correct UCSC genome build. Compute per-cell QC metrics — TSS enrichment (ratio of TSS signal to per-base-pair 2 kb flanking region enrichment) and log10(nFrags) — inspect the QC plots, exclude low-quality cells, and remove ArchR-predicted doublets using a chosen filterRatio. Call accessible peaks with MACS2 through ArchR's reproducible peak-calling step to build the peak-count matrix. Run iterative LSI on genome-wide 500-bp tile counts normalized to a constant of 10,000 with inverse document frequency normalization and log transformation, using resolution 0.6, 30 dimensions, and 4 iterations, re-iterating on the most variable tiles identified across low-resolution clusters. Build a shared-nearest-neighbor graph in LSI space, partition it with the Louvain algorithm, and embed cells in 2D with UMAP; save QC plots, cluster abundance bar plot, and UMAP colored by cluster identity as PNGs.

## Related tools

- **SCALA** (Interactive pipeline wrapping the scATAC-seq QC, MACS2 peak calling, and LSI/UMAP workflow (create_arrow_file.R and companion scripts)) — http://scala.pavlopouloslab.info
- **ArchR** (Arrow-file construction, TSS/nFrags QC, doublet identification, reproducible MACS2 peak calling, iterative LSI, SNN/Louvain clustering, and UMAP embedding) — https://github.com/GreenleafLab/ArchR
- **MACS2** (Called accessible peaks from filtered fragments to build the peak-count matrix feeding LSI)
- **R** (Language in which SCALA (R, Shiny, JavaScript) and the ArchR-based scATAC module are implemented)

## Examples

```
Rscript create_arrow_file.R
```

## Evaluation signals

- QC filters applied match defaults: minTSS=4 and minFrags=1000 on the arrow file, with doublets removed at a documented filterRatio
- LSI settings are resolution 0.6, 30 dimensions, and 4 iterations on 500-bp tiles normalized to a constant of 10,000 with IDF normalization and log transformation
- The resulting LSI space contains 30 dimensions per cell, followed by an SNN graph partitioned by Louvain and a 2D UMAP embedding
- QC plots, cluster abundance bar plot (Fig. 3B style), and UMAP colored by cluster identity (Fig. 3C style) are exported as PNG files
- Peak calling is reproducible via ArchR's MACS2-backed peak-calling step on the same fragments file and genome build

## Limitations

- SCALA supports analysis of only human and mouse datasets for both modalities
- In the online version, dataset size is restricted to < 2 GB with no more than 2 CPU threads per session; some steps cannot be completed online for datasets larger than 50,000 cells, so the desktop version is preferred
- Software version numbers for ArchR employed in SCALA are not stated; only 30 LSI dimensions and 4 iterations are documented, so other parameter defaults are implicit in ArchR

## Evidence

- [methods] LSI resolution 0.6, 30 dimensions, 4 iterations: [evidence span withheld — anchor status: not_found]
