---
name: morisita-horn-repertoire-overlap-analysis
description: Use when paired single-cell RNA-seq with full-length TCR sequencing is
  available (e.g. plate-based smart-seq2, >3500 CD8 TILs) and you must test whether
  annotated CD8 TIL states share overlapping TCR clonotypes within a sample.
when_to_use_negative:
- Single-cell data lacking TCR sequences — clonotype reconstruction and Morisita-Horn
  overlap cannot be computed
- Bulk TCR repertoire data without per-cell transcriptomic state annotations — states
  to compare cannot be defined
- Comparing repertoires across different mice/samples — clonotypes must be grouped
  separately per mouse before computing overlap
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tracer
  role: Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and
    groups cells with identical receptor sequences into clonotypes
  repo: https://github.com/Teichlab/tracer
- name: TILPRED
  role: Logistic regression classifier assigning each CD8 T cell to a reference TIL
    transcriptomic state (exhausted, memory-like, EM-like) from the SingleCellExperiment
    object before repertoire overlap is computed
  repo: https://github.com/carmonalab/TILPRED
- name: R
  role: Environment for computing clonal expansion fractions and pairwise Morisita-Horn
    indices between state repertoires
- name: SingleCellExperiment
  role: Bioconductor container for the CD8 TIL expression matrix that TILPRED reads
    for state prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:14fd67c4b6d18cc47d53ca795a8764296bedecaa7aa98311199262a40f2067cc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# morisita-horn-repertoire-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify clonal relatedness between annotated single-cell T cell transcriptomic states by reconstructing TCR clonotypes from paired scRNA/TCR-seq and computing pairwise Morisita-Horn overlap indices between state repertoires (exhausted, memory-like, EM-like) within each biological replicate.

## When to use

Use when paired single-cell RNA-seq with full-length TCR sequencing is available (e.g. plate-based smart-seq2, >3500 CD8 TILs) and you must test whether annotated CD8 TIL states share overlapping TCR clonotypes within a sample.

## When NOT to use

- Single-cell data lacking TCR sequences — clonotype reconstruction and Morisita-Horn overlap cannot be computed
- Bulk TCR repertoire data without per-cell transcriptomic state annotations — states to compare cannot be defined
- Comparing repertoires across different mice/samples — clonotypes must be grouped separately per mouse before computing overlap

## Inputs

- Paired single-cell RNA-seq FASTQ/counts including full-length TCR alpha and beta genes (e.g. GEO GSE116390)
- SingleCellExperiment expression matrix (e.g. data(B16CD8TIL_SCE))
- Per-cell transcriptomic state annotations from TILPRED (exhausted, memory-like, naive, EM-like)
- Per-mouse clonotype tables grouping cells by identical paired TCR alpha/beta sequences

## Outputs

- Per-mouse clonotype assignments from TraCeR
- Clonal expansion fractions per state (fraction of cells in clones observed in >1 cell)
- Pairwise Morisita-Horn overlap matrix between exhausted, memory-like and EM-like repertoires per mouse
- Validation summary of clonal relatedness across states

## How to apply

Retrieve paired single-cell transcriptome and full-length TCR data (e.g. GEO GSE116390, >3500 CD8 TILs from seven B16 tumor-bearing mice) and reconstruct paired TCR alpha/beta sequences per cell using the TraCeR pipeline, grouping cells with identical receptor sequences into clonotypes separately for each mouse. Annotate each cell with its CD8 TIL state (exhausted, memory-like, naive, EM-like) by running the TILPRED logistic-regression classifier on the SingleCellExperiment expression matrix (predictTilState with filterCD8T to keep only CD8 T cells and a scoreThreshold for confident assignments). For each mouse and each non-naive state, compute the fraction of cells belonging to expanded clonotypes (clones observed in more than one cell). Then compute pairwise Morisita-Horn indices between the clonotype repertoires of the exhausted, memory-like and EM-like states within each mouse. The rationale is that 0 < MH < 1 with highest overlap between exhausted and memory-like (reported index ~0.30) and near-zero overlap between EM-like and memory-like demonstrates partial clonal relatedness between pre-exhausted/progenitor and terminally exhausted states.

## Related tools

- **tracer** (Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and groups cells with identical receptor sequences into clonotypes) — https://github.com/Teichlab/tracer
- **TILPRED** (Logistic regression classifier assigning each CD8 T cell to a reference TIL transcriptomic state (exhausted, memory-like, EM-like) from the SingleCellExperiment object before repertoire overlap is computed) — https://github.com/carmonalab/TILPRED
- **R** (Environment for computing clonal expansion fractions and pairwise Morisita-Horn indices between state repertoires)
- **SingleCellExperiment** (Bioconductor container for the CD8 TIL expression matrix that TILPRED reads for state prediction)

## Examples

```
remotes::install_github("carmonalab/TILPRED"); preds <- TILPRED::predictTilState(data = B16CD8TIL_SCE, filterCD8T = TRUE)
```

## Evaluation signals

- All within-mouse state-pair Morisita-Horn indices fall in the partial-relatedness range 0 < MH < 1
- Highest MH index is between exhausted and memory-like repertoires, consistent with the reported value of 0.30
- EM-like vs memory-like MH index is near zero, matching the reported pattern of partial clonal relatedness
- Clonotype grouping is strictly per mouse — no clonotypes are shared across mice by construction
- Clonal expansion fraction counts only clones observed in more than one cell

## Limitations

- TILPRED is no longer maintained; its authors recommend ProjecTILs as the replacement for CD8 TIL state classification
- TraCeR requires a multi-tool setup (Bowtie2, Trinity v1/v2, IgBLAST with internal_data and optional_file directories and $IGDATA set, Kallisto or Salmon, Graphviz) and a tracer.conf with tool_locations
- Requires full-length TCR genes captured by the scRNA-seq protocol (plate-based smart-seq2); droplet 10X 5'/3' protocols are used in the paper only for other analyses
- The reference map is limited to the B16 melanoma model in wild-type C57BL/6 and PMEL transgenic mice; other strains and cancer types may differ

## Evidence

- [readme] TraCeR purpose: [evidence span withheld — anchor status: external_doc]
