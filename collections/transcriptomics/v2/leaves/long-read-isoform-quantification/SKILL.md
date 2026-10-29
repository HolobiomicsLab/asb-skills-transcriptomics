---
name: long-read-isoform-quantification
description: Use when Nanopore single-cell/spatial FASTQ reads (e.g., Jurkat GSP-targeted,
  MOB Visium, CRCLM) must be turned into cell-by-isoform count matrices, or when benchmarking
  isoform quantification methods against Illumina or PacBio IsoSeq gold standards.
when_to_use_negative:
- Input reads are already a processed cell-by-isoform feature table — use downstream
  splicing analysis (Splice object, phi estimation) instead of re-quantifying
- Data are short-read Illumina sequences with no cell barcode/UMI error problem —
  Longcell's denoising rationale (85–99% Nanopore per-base error) does not apply
- Goal is de-novo isoform discovery without a cell/spatial barcode structure
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Longcell
  role: 'Core pipeline: cell-barcode and UMI recovery, consensus error correction,
    isoform quantification, and Beta-Binomial splicing modeling'
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Preprocessing companion package producing gene_bed.rds gene strand annotations
  repo: github.com/yuntianf/Longcell
- name: IsoQuant
  role: 'Isoform quantification run on Longcell-denoised reads (best benchmark: Spearman
    0.70 on MOB Visium)'
- name: Sicelore2
  role: Benchmark comparator using fixed-edit-distance UMI collapsing (next-best on
    Jurkat)
- name: FLAMES
  role: Benchmark comparator for isoform quantification
- name: Nanopore
  role: Long-read sequencing platform producing the input FASTQ (85–99% per-base accuracy
    motivating error correction)
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:161e5c38be94acfcd4fe2b7ca5f46b741266dd2326febe0747e7dae187f08ebc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - FLAMES
  - IsoQuant
  - Longcell
  - LongcellPre
  - Nanopore
  - Seurat
  - Sicelore2
  - dplyr
  - future
  - future.apply
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# long-read-isoform-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pipeline skill for converting single-cell or spatially barcoded Nanopore long-read FASTQ into a cell-by-isoform count matrix, built around Longcell's barcode/UMI recovery, consensus error correction, and UMI-clustered isoform quantification, with benchmarking against Illumina- or PacBio-supervised ground truth via per-cell Spearman correlation.

## When to use

Use when Nanopore single-cell/spatial FASTQ reads (e.g., Jurkat GSP-targeted, MOB Visium, CRCLM) must be turned into cell-by-isoform count matrices, or when benchmarking isoform quantification methods against Illumina or PacBio IsoSeq gold standards.

## When NOT to use

- Input reads are already a processed cell-by-isoform feature table — use downstream splicing analysis (Splice object, phi estimation) instead of re-quantifying
- Data are short-read Illumina sequences with no cell barcode/UMI error problem — Longcell's denoising rationale (85–99% Nanopore per-base error) does not apply
- Goal is de-novo isoform discovery without a cell/spatial barcode structure

## Inputs

- Nanopore FASTQ reads from single-cell or spatially barcoded libraries (Promethion R10.4.1/FLO-PRO114M, Visium MOB, CRCLM)
- Cell-barcode whitelist (10X-style)
- GRCh38 reference genome
- Gencode v39 GTF and its paftools.js-derived splice-junction bed file
- Ground-truth cell-by-isoform matrix (PacBio IsoSeq v3.4.0 with --min-aln-coverage 0.5 and --min-aln-identity 0.9, or Illumina-supervised quantification)

## Outputs

- Cell-by-isoform count matrix per method (e.g., sc_iso_count.rds)
- Per-cell Spearman correlation table vs ground truth (per_cell_isoform_correlation.csv, spearman_correlations.csv)
- LongcellPre gene strand annotation (gene_bed.rds) for downstream splicing-heterogeneity analysis

## How to apply

Preprocess Nanopore FASTQ with Longcell: search adapters/polyA, match cell barcodes by k-mer matching with edit-distance cutoff 3 and deviant start-position filtering, and cluster UMIs via Needleman-Wunsch alignment — this matters because Nanopore per-base accuracy of 85–99% makes UMIs (short, random sequences) more vulnerable to error than whitelist-checked cell barcodes. Align denoised reads to GRCh38 with minimap2 v2.24 in spliced mode using a splice-junction bed derived from the Gencode v39 GTF (minimap2 -ax splice --junc-bed --secondary=no --sam-hit-only). Quantify isoforms with Longcell+IsoQuant (which recovered consensus alignments per UMI cluster, where consensus among PCR-duplicate reads corrects truncation and mapping errors), or compare against FLAMES, Sicelore2, wf-single-cell, and IsoQuant alone using identical mapping software, genome, and annotation. Fully join each method's cell-by-isoform matrix with the ground-truth matrix by isoform, then compute per-cell Spearman correlations; reported benchmarks are 0.43 (Jurkat Nanopore vs PacBio IsoSeq, an 8.72% gain over Sicelore2) and 0.70 for Longcell-IsoQuant on the 918-cell MOB Visium dataset.

## Related tools

- **Longcell** (Core pipeline: cell-barcode and UMI recovery, consensus error correction, isoform quantification, and Beta-Binomial splicing modeling) — github.com/yuntianf/Longcell
- **LongcellPre** (Preprocessing companion package producing gene_bed.rds gene strand annotations) — github.com/yuntianf/Longcell
- **IsoQuant** (Isoform quantification run on Longcell-denoised reads (best benchmark: Spearman 0.70 on MOB Visium))
- **Sicelore2** (Benchmark comparator using fixed-edit-distance UMI collapsing (next-best on Jurkat))
- **FLAMES** (Benchmark comparator for isoform quantification)
- **Nanopore** (Long-read sequencing platform producing the input FASTQ (85–99% per-base accuracy motivating error correction))

## Examples

```
minimap2 -ax splice --junc-bed --secondary=no --sam-hit-only GRCh38.fa gencode_v39.bed reads.fastq > aln.sam
```

## Evaluation signals

- Reproduced mean per-cell Spearman correlation matches reported values within tolerance: Longcell+IsoQuant 0.70 and wf-single-cell 0.66 on the 918-cell MOB Visium benchmark; 0.43 for Longcell on Jurkat Nanopore vs PacBio IsoSeq (8.72% over Sicelore2)
- per_cell_isoform_correlation.csv contains correlation values for all 5881 Jurkat cells for every method
- Each method's cell-by-isoform matrix can be fully joined with the ground-truth matrix by isoform ID
- Quantification is UMI-count based: reads within a cell are grouped by recovered UMI and collapsed into one consensus alignment per UMI cluster, avoiding the inflation caused by fixed-edit-distance UMI collapsing under high sequencing error
- Longcell+IsoQuant (0.70) outperforms IsoQuant alone (0.58) on MOB Visium, confirming the denoising step adds accuracy

## Limitations

- Nanopore per-base sequencing accuracy of 85–99% compromises cell-barcode and UMI recovery; UMIs, being shorter and randomly generated, are especially vulnerable to error
- Existing fixed-edit-distance UMI collapsing methods overestimate UMI counts under high sequencing error, causing inflated quantification
- Accumulated sequencing errors can misalign small internal exons, producing false-positive novel isoforms; read truncation and mapping errors require consensus correction among PCR-duplicate reads
- The influence of sequencing errors, read truncation, and mapping errors on single-cell long-read isoform quantification had not been characterized prior to this study

## Evidence

- [discussion] discussion_consensus: [evidence span withheld — anchor status: not_found]
- [readme] readme_install: [evidence span withheld — anchor status: external_doc]
