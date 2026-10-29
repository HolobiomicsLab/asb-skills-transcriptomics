---
name: paired-scrnaseq-tcrseq-data-processing
description: Use when you have paired single-cell transcriptome and full-length TCR
  sequencing data (e.g. Smart-seq2 CD8 TILs) and need per-cell clonotype assignments
  to test clonal relatedness between transcriptional states.
when_to_use_negative:
- Your data is droplet-based (10X 5'/3') without full-length TCR capture — TraCeR
  is designed for full-length TCR reconstruction from single-cell RNA-seq, and the
  paper contrasts plate-based smart-seq2 with droplet technologies
- You only need transcriptomic state annotation without clonal information — TILPRED/ProjecTILs
  alone suffices and TraCeR reconstruction is unnecessary
- You need cross-sample/cross-patient clonotype sharing — this skill groups clonotypes
  separately per mouse/within-host, which prevents cross-sample clonotype comparison
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: tracer
  role: Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and
    groups cells with identical receptor sequences into clonotypes
  repo: https://github.com/Teichlab/tracer
- name: TILPRED
  role: Logistic regression classifier assigning each CD8 T cell in a SingleCellExperiment
    object to a reference TIL state (exhausted, memory-like, EffectorMemory, naive)
  repo: https://github.com/carmonalab/TILPRED
- name: ProjecTILs
  role: Maintained successor to TILPRED; projects scRNA-seq data into a reference
    CD8 TIL atlas for state annotation
  repo: https://github.com/carmonalab/ProjecTILs
- name: SingleCellExperiment
  role: Bioconductor container for the expression matrix that TILPRED reads
- name: AUCell
  role: Bioconductor package used by TILPRED for gene-signature enrichment scoring
    during state classification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:e3e40255c5afb4056776adb68150495bbd75f0d854f4e16c3fd8f3867c313686
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  repo_url: https://github.com/Teichlab/tracer
  tools:
  - AUCell
  - ProjecTILs
  - R
  - SingleCellExperiment
  - TILPRED
  - tracer
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paired-scrnaseq-tcrseq-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct paired TCR alpha/beta sequences from full-length TCR single-cell RNA-seq reads with TraCeR, group cells sharing identical receptor sequences into per-mouse clonotypes, and link each clonotype to its transcriptomically assigned CD8 TIL state (exhausted, memory-like, EM-like, naive) via TILPRED.

## When to use

Use when you have paired single-cell transcriptome and full-length TCR sequencing data (e.g. Smart-seq2 CD8 TILs) and need per-cell clonotype assignments to test clonal relatedness between transcriptional states.

## When NOT to use

- Your data is droplet-based (10X 5'/3') without full-length TCR capture — TraCeR is designed for full-length TCR reconstruction from single-cell RNA-seq, and the paper contrasts plate-based smart-seq2 with droplet technologies
- You only need transcriptomic state annotation without clonal information — TILPRED/ProjecTILs alone suffices and TraCeR reconstruction is unnecessary
- You need cross-sample/cross-patient clonotype sharing — this skill groups clonotypes separately per mouse/within-host, which prevents cross-sample clonotype comparison

## Inputs

- Paired single-cell RNA-seq FASTQ/BAM with full-length TCR genes (>3500 CD8 TILs, Smart-seq2 plate-based data, GSE116390)
- SingleCellExperiment expression matrix (e.g. B16CD8TIL_SCE)
- Per-mouse sample metadata to enforce within-host clonotype grouping
- TraCeR configuration file (tracer.conf) with tool locations for Bowtie2, Trinity, IgBLAST, Kallisto/Salmon

## Outputs

- Per-cell paired TCR alpha/beta receptor sequences reconstructed by TraCeR
- Per-mouse clonotype assignments (groups of cells sharing identical receptor sequences)
- TILPRED state labels per cell (exhausted, memory-like, naive, EM-like; plus Treg/CD4T/NKT for non-CD8 cells)
- Cell-level table joining clonotype ID to transcriptomic TIL state for overlap analysis

## How to apply

Start from the paired scRNA-seq/TCR-seq data (e.g. >3500 CD8 TILs from seven tumor-bearing mice at GEO accession GSE116390) and run the TraCeR pipeline to reconstruct paired TCR alpha/beta sequences from the full-length TCR reads; TraCeR assembles TCR contigs with Trinity, aligns with Bowtie2, and annotates contigs with IgBLAST ($IGDATA must point to the IgBLAST install). Run TraCeR separately for each mouse so cells sharing identical receptor sequences are grouped into clonotypes within-host, not across hosts. In parallel, classify each cell's transcriptomic state with TILPRED, a logistic regression classifier operating on a SingleCellExperiment object (installed via remotes::install_github("carmonalab/TILPRED"), requires doParallel, doRNG, AUCell; use filterCD8T=TRUE and scoreThreshold/cellCycleThreshold parameters in predictTilState). Merge TraCeR clonotype calls with TILPRED state labels per cell to produce a cell-by-clonotype-by-state table that downstream clonal-expansion and Morisita-Horn overlap analyses consume. Note that TILPRED is no longer maintained and its successor is ProjecTILs, which projects query cells into the same CD8 TIL reference atlas.

## Related tools

- **tracer** (Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and groups cells with identical receptor sequences into clonotypes) — https://github.com/Teichlab/tracer
- **TILPRED** (Logistic regression classifier assigning each CD8 T cell in a SingleCellExperiment object to a reference TIL state (exhausted, memory-like, EffectorMemory, naive)) — https://github.com/carmonalab/TILPRED
- **ProjecTILs** (Maintained successor to TILPRED; projects scRNA-seq data into a reference CD8 TIL atlas for state annotation) — https://github.com/carmonalab/ProjecTILs
- **SingleCellExperiment** (Bioconductor container for the expression matrix that TILPRED reads)
- **AUCell** (Bioconductor package used by TILPRED for gene-signature enrichment scoring during state classification)

## Examples

```
remotes::install_github("carmonalab/TILPRED"); predictTilState(data, nCores=4, filterCD8T=TRUE, scoreThreshold, cellCycleThreshold)
```

## Evaluation signals

- Each cell with adequate TCR reads receives a paired TCR alpha/beta reconstruction from TraCeR; cells sharing identical receptor sequences collapse to one clonotype ID
- Clonotypes are grouped strictly within-mouse — no clonotype should span multiple mice
- Every cell passing CD8 T-cell filtering (filterCD8T) receives a TILPRED state label among exhausted, memory-like, naive, EM-like; non-CD8 cells classify as Treg, CD4T, or NKT
- Non-naive states show large clonal expansion (fraction of cells in clones observed in >1 cell is high) and pairwise Morisita-Horn indices satisfy 0 < MH < 1, with exhausted–memory-like highest (~0.30) and EM-like–memory-like near zero
- TraCeR tool prerequisites resolve: IGDATA points to a working IgBLAST with internal_data and optional_file directories, and Trinity jellyfish memory is set in tracer.conf

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs instead for CD8 TIL state classification
- TraCeR requires installation of external tools (Bowtie2, Trinity, IgBLAST, Kallisto/Salmon) and correct tracer.conf paths; IgBLAST additionally needs internal_data and optional_file directories and the $IGDATA environment variable
- Reference CD8 TIL map was built from B16 melanoma in a single mouse strain (C57BL/6 plus PMEL transgenic); generalization to other strains and cancer types is untested in this study

## Evidence

- [readme] TraCeR reconstructs TCR sequences: [evidence span withheld — anchor status: external_doc]
- [readme] TraCeR prerequisites: [evidence span withheld — anchor status: external_doc]
