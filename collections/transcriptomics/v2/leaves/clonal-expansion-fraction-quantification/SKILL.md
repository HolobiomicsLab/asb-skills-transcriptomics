---
name: clonal-expansion-fraction-quantification
description: Use when paired single-cell transcriptome + full-length TCR data are
  available (e.g. >3500 CD8 TILs from seven B16 melanoma mice, GEO GSE116390) and
  the goal is to test whether distinct CD8 TIL states share overlapping TCR clonotypes.
when_to_use_negative:
- Input is bulk TCR repertoire sequencing without single-cell pairing — per-cell state-to-clonotype
  assignment and TILPRED annotation are impossible
- Droplet-based 10X 3' data without full-length TCR capture — TraCeR requires full-length
  TCR transcript reconstruction, which the study performed with plate-based Smart-seq2
- The goal is state classification of human CD8 T cells rather than clonal quantification
  — TILPRED is no longer maintained and the authors recommend ProjecTILs instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tracer
  role: Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and
    groups cells with identical receptor sequences into clonotypes per mouse
  repo: https://github.com/Teichlab/tracer
- name: TILPRED
  role: Logistic-regression classifier that assigns each CD8 T cell in a SingleCellExperiment
    object to a reference CD8 TIL state so clonotype fractions can be computed per
    state
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Bioconductor container holding the CD8 TIL expression matrix that TILPRED
    reads for state prediction
- name: R
  role: Language used to compute clonotype expansion fractions and Morisita-Horn overlap
    indices (TILPRED dependency stack includes doParallel, doRNG, AUCell)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:5ba1cbb18990f791563feee65d2cebd6dd70fe05ffee7a6b9a98ca09e565576a
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clonal-expansion-fraction-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies clonal expansion of CD8 TIL transcriptomic states by computing, per mouse, the fraction of cells belonging to expanded clonotypes (clones seen in >1 cell) and pairwise Morisita-Horn overlap between the TCR repertoires of exhausted, memory-like and EM-like states, using TraCeR-reconstructed paired TCR sequences and TILPRED state annotations.

## When to use

Use when paired single-cell transcriptome + full-length TCR data are available (e.g. >3500 CD8 TILs from seven B16 melanoma mice, GEO GSE116390) and the goal is to test whether distinct CD8 TIL states share overlapping TCR clonotypes.

## When NOT to use

- Input is bulk TCR repertoire sequencing without single-cell pairing — per-cell state-to-clonotype assignment and TILPRED annotation are impossible
- Droplet-based 10X 3' data without full-length TCR capture — TraCeR requires full-length TCR transcript reconstruction, which the study performed with plate-based Smart-seq2
- The goal is state classification of human CD8 T cells rather than clonal quantification — TILPRED is no longer maintained and the authors recommend ProjecTILs instead

## Inputs

- Paired single-cell RNA-seq with full-length TCR genes from >3500 CD8 TILs (Smart-seq2, seven tumor-bearing mice: wild-type C57BL/6 and PMEL transgenic; GEO GSE116390)
- Per-cell transcriptomic state labels from TILPRED on a SingleCellExperiment object (exhausted, memory-like, naive, EM-like)
- TraCeR per-cell TCR clonotype assignments computed separately per mouse

## Outputs

- Per mouse and per non-naive state: fraction of cells belonging to expanded clonotypes (clones observed in >1 cell)
- Pairwise Morisita-Horn overlap indices between exhausted, memory-like and EM-like clonotype repertoires per mouse
- Evidence of partial clonal relatedness across states (reported MH ~ 0.30 between exhausted and memory-like)

## How to apply

First reconstruct paired TCR alpha/beta sequences per cell with TraCeR, which groups cells sharing identical receptor sequences into clonotypes, and do this separately for each mouse so clones are never pooled across animals. Annotate each cell with its CD8 TIL transcriptomic state (exhausted, memory-like, naive, EM-like) using the TILPRED logistic-regression classifier run on the SingleCellExperiment expression matrix (e.g. the B16CD8TIL_SCE object). For each mouse and each non-naive state, compute the fraction of cells belonging to expanded clonotypes, defined as clones observed in more than one cell. Then compute pairwise Morisita-Horn indices between the clonotype repertoires of the exhausted, memory-like and EM-like states within each mouse. The rationale is that MH values in (0,1) indicate partial clonal relatedness between states, which distinguishes true state transitions from unrelated state populations.

## Related tools

- **tracer** (Reconstructs paired TCR alpha/beta sequences from single-cell RNA-seq and groups cells with identical receptor sequences into clonotypes per mouse) — https://github.com/Teichlab/tracer
- **TILPRED** (Logistic-regression classifier that assigns each CD8 T cell in a SingleCellExperiment object to a reference CD8 TIL state so clonotype fractions can be computed per state) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Bioconductor container holding the CD8 TIL expression matrix that TILPRED reads for state prediction)
- **R** (Language used to compute clonotype expansion fractions and Morisita-Horn overlap indices (TILPRED dependency stack includes doParallel, doRNG, AUCell))

## Examples

```
python tracer assemble -f /path/to/cells.fastq -o /path/to/tcr_out; python tracer summarise /path/to/tcr_out
```

## Evaluation signals

- Clonotypes are grouped within each mouse only — no clonotype counts are pooled across the seven mice
- An expanded clonotype is defined as a clone observed in more than one cell; expansion fractions are reported separately for exhausted, memory-like and EM-like states, excluding naive
- Computed Morisita-Horn indices lie in (0, 1), with the highest overlap between exhausted and memory-like and near-zero overlap between EM-like and memory-like, consistent with the reported partial clonal relatedness (MH ~ 0.30)
- TraCeR output contains paired productive TCR alpha/beta sequences per cell; IgBLAST was installed with internal_data and optional_file directories and $IGDATA set, otherwise reconstruction silently fails
- Cell counts reconcile with the >3500 CD8 TILs from seven tumor-bearing mice after filterCD8T filtering

## Limitations

- TraCeR requires external tools (Bowtie2, Trinity with a Bowtie v1 install and a max_jellyfish_memory setting, IgBLAST with $IGDATA set, kallisto or salmon) and is now maintained by the original author only as a personal project
- The TCR reconstruction approach depends on plate-based Smart-seq2 full-length TCR sequencing; it is not directly applicable to droplet 3'/5' counting data
- The CD8 TIL reference map underlying state labels was built from the B16 melanoma model and a single mouse strain; the authors note future studies of additional mouse strains and cancer types are needed to refine it
- IgBLAST installation is error-prone: downloading executables alone is insufficient — internal_data and optional_file directories must be copied next to the executable

## Evidence

- [readme] This tool reconstructs the sequences of rearranged and expressed T cell receptor genes from single-cell RNA-seq data.: [evidence span withheld — anchor status: external_doc]
- [readme] It then uses the TCR sequences to identify cells that have the same receptor sequences and so derive from the same original clonally-expanded cell.: [evidence span withheld — anchor status: external_doc]
