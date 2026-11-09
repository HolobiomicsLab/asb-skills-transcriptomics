---
name: tcr-clonotype-assignment-per-mouse
description: Use when paired single-cell transcriptome plus full-length TCR sequencing
  of CD8 TILs (e.g. >3500 cells from seven tumor-bearing mice, GSE116390) must be
  grouped into clonotypes per mouse before comparing clonal overlap across transcriptomic
  states.
when_to_use_negative:
- Input is droplet-based (10X 5'/3') data without full-length TCR genes — TraCeR requires
  full-length TCR sequences and the study's clonotype reconstruction was done on plate-based
  smart-seq2 data
- Cells come from a pooled experiment without per-mouse/sample identity — clonotypes
  must be grouped separately for each mouse, so this skill cannot be applied
- Goal is bulk TCR repertoire analysis rather than linking single-cell transcriptomic
  states to clonotypes
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tracer
  role: Reconstructs TCR alpha/beta sequences from single-cell RNA-seq and identifies
    cells with identical receptor sequences as clonotypes
  repo: https://github.com/Teichlab/tracer
- name: TILPRED
  role: Logistic regression classifier assigning each CD8 T cell in the SingleCellExperiment
    object to reference CD8 TIL states (Exhausted, MemoryLike, EffectorMemory, naive)
    for clonotype-state pairing
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Bioconductor container holding the CD8 TIL expression matrix on which predictTilState
    is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:70b49de58d5e5e30c50ff844b7f3d90e04f942059f4fbd5a5da25ec69db62a68
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
  - R
  - SingleCellExperiment
  - TILPRED
  - tracer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tcr-clonotype-assignment-per-mouse

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns single CD8 TILs to TCR clonotypes by reconstructing paired full-length TCR alpha/beta sequences from single-cell RNA-seq with TraCeR, grouping cells sharing identical receptor sequences separately within each mouse, and pairing the resulting clonotype labels with transcriptomic states (exhausted, memory-like, EM-like, naive) predicted by TILPRED to quantify clonal relatedness (Morisita-Horn) between CD8 TIL states.

## When to use

Use when paired single-cell transcriptome plus full-length TCR sequencing of CD8 TILs (e.g. >3500 cells from seven tumor-bearing mice, GSE116390) must be grouped into clonotypes per mouse before comparing clonal overlap across transcriptomic states.

## When NOT to use

- Input is droplet-based (10X 5'/3') data without full-length TCR genes — TraCeR requires full-length TCR sequences and the study's clonotype reconstruction was done on plate-based smart-seq2 data
- Cells come from a pooled experiment without per-mouse/sample identity — clonotypes must be grouped separately for each mouse, so this skill cannot be applied
- Goal is bulk TCR repertoire analysis rather than linking single-cell transcriptomic states to clonotypes

## Inputs

- Paired single-cell transcriptome + full-length TCR sequencing data (GEO GSE116390, >3500 CD8 TILs from seven tumor-bearing mice, wild-type C57BL/6 and PMEL transgenic)
- Per-cell FASTQ files with TCR loci coverage (smart-seq2 plate-based scRNA-seq)
- SingleCellExperiment expression matrix (e.g. B16CD8TIL_SCE) for state annotation

## Outputs

- Per-cell TCR clonotype assignments (paired TCR alpha/beta sequences), grouped separately per mouse
- Per-mouse fraction of cells belonging to expanded clonotypes (>1 cell) per non-naive CD8 TIL state
- Pairwise Morisita-Horn overlap indices between clonotype repertoires of exhausted, memory-like and EM-like states (e.g. ~0.30 for exhausted vs memory-like)
- TILPRED state probability scores per cell (Exhausted, MemoryLike, EffectorMemory, naive)

## How to apply

Retrieve paired scRNA-seq/full-length TCR data (GSE116390, smart-seq2) and run TraCeR's assemble and summarise modes on each cell to reconstruct paired TCR alpha/beta contigs, setting $IGDATA to the IgBLAST installation and configuring tool paths in tracer.conf. Group cells sharing identical reconstructed receptor sequences into clonotypes strictly within each mouse — never pool clonotypes across mice, since repertoires are private per animal. Annotate each cell with its CD8 TIL transcriptomic state using TILPRED on the SingleCellExperiment object (states: Exhausted, MemoryLike, EffectorMemory, naive), excluding non-CD8 cells via filterCD8T. Compute, per mouse, the fraction of cells in expanded clonotypes (clones seen in more than one cell) and pairwise Morisita-Horn indices between the clonotype repertoires of exhausted, memory-like and EM-like states.

## Related tools

- **tracer** (Reconstructs TCR alpha/beta sequences from single-cell RNA-seq and identifies cells with identical receptor sequences as clonotypes) — https://github.com/Teichlab/tracer
- **TILPRED** (Logistic regression classifier assigning each CD8 T cell in the SingleCellExperiment object to reference CD8 TIL states (Exhausted, MemoryLike, EffectorMemory, naive) for clonotype-state pairing) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Bioconductor container holding the CD8 TIL expression matrix on which predictTilState is run)

## Examples

```
export IGDATA=/<path_to_igblast>/igblast/1.4.0/bin; python tracer assemble -b Mmus -f fastq_dir -o tracer_out; python tracer summarise -o tracer_out
```

## Evaluation signals

- Every reconstructed cell carries a paired TCR alpha/beta sequence or an explicit reconstruction-failed flag; clonotype IDs are identical for and only for cells sharing identical receptor sequences
- Clonotype grouping is verified to be per-mouse — no clonotype ID is shared across mice
- TILPRED output assigns each non-excluded cell one of the reference states (Exhausted, MemoryLike, EffectorMemory, naive) with a probability score
- Computed Morisita-Horn indices satisfy 0 < MH < 1 with highest overlap between exhausted and memory-like states and near-zero overlap between EM-like and memory-like states
- Scale check: >3500 CD8 TILs from seven tumor-bearing mice are processed, with non-naive states showing large clonal expansion (clones observed in more than one cell)

## Limitations

- TraCeR requires external tools (Bowtie2, Trinity, IgBLAST, Kallisto/Salmon) with explicit paths in tracer.conf; IgBLAST additionally needs the internal_data and optional_file directories downloaded and the $IGDATA environment variable set
- TraCeR's max_jellyfish_memory for Trinity must be set appropriately in the config; statsmodels/seaborn version conflicts can break installation
- TILPRED is no longer maintained; the authors recommend ProjecTILs instead
- Reference clonotype/state map is limited to the B16 melanoma model and a single mouse strain (C57BL/6 with PMEL transgenic); repertoires are private to each mouse so clonotypes cannot be compared across mice

## Evidence

- [readme] TraCeR function: [evidence span withheld — anchor status: external_doc]
- [readme] IgBLAST setup requirement: [evidence span withheld — anchor status: external_doc]
