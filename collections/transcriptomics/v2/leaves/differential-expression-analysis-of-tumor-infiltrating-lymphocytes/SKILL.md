---
name: differential-expression-analysis-of-tumor-infiltrating-lymphocytes
description: Use when comparing bulk or single-cell RNA-seq of CD8 TILs between immunotherapy
  (anti-PD-1) and control conditions, or when scoring an input scRNA-seq dataset for
  CD8 TIL transcriptomic states to test EM-like signature enrichment.
when_to_use_negative:
- Input is not CD8 T cell transcriptomic data — TILPRED assumes a SingleCellExperiment
  of CD8 T cell profiles and needs filterCD8T to handle non-CD8 contamination
- The goal is general cell-type reference mapping of scRNA-seq beyond TIL state prediction
  — TILPRED is no longer maintained and the authors recommend ProjecTILs for reference-based
  projection
- Tumor antigen specificity of the samples is unknown and the question depends on
  clone-level specificity — antigen specificity 'is often unknown' for polyclonal
  CD8 TIL samples, so transcriptome-level DE alone cannot resolve clonal behavior
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Fit the differential expression model (anti-PD-1 vs control) on tximport-aggregated
    gene-level counts
- name: tximport
  role: Aggregate Salmon transcript-level estimates to gene-level counts for DESeq2
- name: Salmon
  role: Quantify transcript abundances per sample via quasi-mapping against the mouse
    transcriptome
- name: clusterProfiler
  role: Run GSEA of the EM-like CD8 TIL signature on log2-fold-change-ranked genes
- name: TILPRED
  role: Logistic regression classifier assigning SingleCellExperiment CD8 T cells
    to Exhausted, MemoryLike or EffectorMemory states via predictTilState
  repo: https://github.com/carmonalab/TILPRED
- name: AUCell
  role: Score CD8 TIL state gene signature enrichment on single-cell data
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:801715f19da37da350db3a64b602c658eb3b9e449f06aaf94f2c1d8a39562caa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  repo_url: https://github.com/carmonalab/TILPRED
  tools:
  - AUCell
  - BiocManager
  - DESeq2
  - R
  - Salmon
  - TILPRED
  - Trimmomatic
  - clusterProfiler
  - tximport
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Differential expression analysis of tumor-infiltrating lymphocytes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies and contrasts transcriptomic profiles of CD8 TILs across anti-PD-1 vs control conditions, using DESeq2/tximport for bulk RNA-seq and TILPRED/AUCell to classify CD8 TIL states (exhausted, memory-like, naive, EM-like), to test whether checkpoint blockade selectively enriches the EM-like CD8 TIL signature.

## When to use

Use when comparing bulk or single-cell RNA-seq of CD8 TILs between immunotherapy (anti-PD-1) and control conditions, or when scoring an input scRNA-seq dataset for CD8 TIL transcriptomic states to test EM-like signature enrichment.

## When NOT to use

- Input is not CD8 T cell transcriptomic data — TILPRED assumes a SingleCellExperiment of CD8 T cell profiles and needs filterCD8T to handle non-CD8 contamination
- The goal is general cell-type reference mapping of scRNA-seq beyond TIL state prediction — TILPRED is no longer maintained and the authors recommend ProjecTILs for reference-based projection
- Tumor antigen specificity of the samples is unknown and the question depends on clone-level specificity — antigen specificity 'is often unknown' for polyclonal CD8 TIL samples, so transcriptome-level DE alone cannot resolve clonal behavior

## Inputs

- Bulk RNA-seq FASTQ files of CD8 TIL samples (e.g. GSE62771 anti-PD-1 vs control) with sample metadata
- SingleCellExperiment object containing CD8 T cell profiles for TILPRED classification
- CD8 TIL state gene signatures (EM-like, exhausted, memory-like) as GSEA query gene sets

## Outputs

- Gene-level count matrix and DESeq2 differential expression table (anti-PD-1 vs control)
- GSEA statistics for the EM-like CD8 TIL signature (e.g. written to gsea_emlike_enrichment.csv)
- Per-cell state probability scores and class labels (Exhausted, MemoryLike, EffectorMemory, Treg, CD4T, NKT)

## How to apply

For bulk RNA-seq (e.g. the Gubin et al. murine sarcoma dataset GSE62771, anti-PD-1 vs control CD8 TILs), download FASTQ and metadata from GEO/SRA, trim with Trimmomatic, quantify transcripts with Salmon quasi-mapping against the mouse transcriptome, aggregate to gene level with tximport, and fit a DESeq2 model (anti-PD-1 vs control). Rank genes by log2 fold change and run GSEA in clusterProfiler with the EM-like CD8 TIL state signature as the query gene set, expecting nominal enrichment (reported p = 0.035). For single-cell input, classify each cell with TILPRED, a logistic regression-based classifier over a SingleCellExperiment object that assigns probability scores for Exhausted, MemoryLike (Tpex), and EffectorMemory states, using predictTilState parameters including scoreThreshold, cellCycleThreshold and filterCD8T; non-CD8 T cells are classified via curated gene signatures into Treg (Foxp3), CD4T and NKT classes. Enrichment of state signatures can be computed with AUCell on the SingleCellExperiment object.

## Related tools

- **DESeq2** (Fit the differential expression model (anti-PD-1 vs control) on tximport-aggregated gene-level counts)
- **tximport** (Aggregate Salmon transcript-level estimates to gene-level counts for DESeq2)
- **Salmon** (Quantify transcript abundances per sample via quasi-mapping against the mouse transcriptome)
- **clusterProfiler** (Run GSEA of the EM-like CD8 TIL signature on log2-fold-change-ranked genes)
- **TILPRED** (Logistic regression classifier assigning SingleCellExperiment CD8 T cells to Exhausted, MemoryLike or EffectorMemory states via predictTilState) — https://github.com/carmonalab/TILPRED
- **AUCell** (Score CD8 TIL state gene signature enrichment on single-cell data) — https://bioconductor.org/packages/release/bioc/html/AUCell.html

## Examples

```
remotes::install_github("carmonalab/TILPRED"); predictTilState(data, nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T)
```

## Evaluation signals

- GSEA of the EM-like signature on the Gubin et al. anti-PD-1 vs control CD8 TIL bulk data reproduces the reported nominal p = 0.035 with selective enrichment in anti-PD-1-treated samples
- DESeq2 results table contains log2 fold change and adjusted p-value columns and can be joined to the GSEA ranking by gene ID
- TILPRED output assigns each cell a probability score for Exhausted, MemoryLike and EffectorMemory states; non-CD8 T cells fall into Treg/CD4T/NKT classes per curated signatures
- Salmon quantification aggregated through tximport yields integer-like gene counts compatible with the DESeq2 model design (anti-PD-1 vs control)
- EM-like labeling is consistent with the validated marker Ly6C (Ly6c2) plus Tcf7, Lef1, Il7r expression and low-to-intermediate Pdcd1

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs instead
- In prior human studies of EM-like TILs, tumor specificity was unknown, limiting interpretation of EM-like enrichment in polyclonal bulk samples
- Human CD8 TIL samples show large biological and technical variability with often unknown antigen specificity, complicating cross-condition comparison
- The CD8 TIL reference map is limited to the B16 melanoma model and a single mouse strain (C57BL/6); additional mouse strains and cancer types are needed for generalization
- TraCeR (used for paired TCR reconstruction) was developed and maintained as a personal project, with dependencies on Bowtie2, Trinity, IgBLAST and Kallisto/Salmon that must be configured via ~/.tracerrc

## Evidence
