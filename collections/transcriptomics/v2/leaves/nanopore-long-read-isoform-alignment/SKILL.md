---
name: nanopore-long-read-isoform-alignment
description: Use when quantifying isoforms or PSI from single-cell/spatially barcoded
  Nanopore long reads (e.g., CRCLM sample) and raw minimap2 alignments show misaligned
  small internal exons inflating false-positive isoforms.
when_to_use_negative:
- Input is short-read sequencing data — Longcell and its minimap2 spliced-alignment
  workflow are designed for Nanopore long reads (85–99% per-base accuracy)
- The goal is gene-level expression only; UMI-cluster consensus isoform correction
  is unnecessary when isoform/PSI resolution is not required
- Reads are already consensus-corrected isoform assignments rather than raw alignments
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: Preprocessing pipeline that recovers cell barcodes/UMIs, corrects sequencing
    errors, and applies UMI-cluster consensus misalignment correction to splice alignments
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Long-read sequencing platform whose full-length reads carry the isoform-defining
    splice junctions
- name: minimap2
  role: Aligner run at v2.24 in spliced alignment mode against GRCh38 with the Gencode
    v39 splice-junction bed
- name: dplyr
  role: R data manipulation for tallying spliced-in vs. gene counts per internal exon
    and computing PSI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:3df615686224934827036013125f2f9365226b0e6aae14c228ba6a6aa625f5a6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - Nanopore
  - dplyr
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-long-read-isoform-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align Nanopore long reads to GRCh38 in spliced mode with minimap2 v2.24 using a Gencode v39 GTF-derived splice-junction bed, then apply Longcell's UMI-cluster consensus correction so isoform quantification is not biased by sequencing errors, read truncation, or misaligned small internal exons.

## When to use

Use when quantifying isoforms or PSI from single-cell/spatially barcoded Nanopore long reads (e.g., CRCLM sample) and raw minimap2 alignments show misaligned small internal exons inflating false-positive isoforms.

## When NOT to use

- Input is short-read sequencing data — Longcell and its minimap2 spliced-alignment workflow are designed for Nanopore long reads (85–99% per-base accuracy)
- The goal is gene-level expression only; UMI-cluster consensus isoform correction is unnecessary when isoform/PSI resolution is not required
- Reads are already consensus-corrected isoform assignments rather than raw alignments

## Inputs

- Nanopore FASTQ long reads from single-cell or spatially barcoded libraries (e.g., 10X CRCLM sample)
- GRCh38 reference genome
- Gencode v39 GTF (converted to splice-junction bed via paftools.js)
- gene_bed.rds from LongcellPre (gene strand information for meta splicing sites)

## Outputs

- Read-level splice-site table with corrected cell barcodes and UMIs
- Splice object (via creatSplice_from_df) supporting PSI, sashimi, and phi metrics
- Pre- vs. post-correction PSI per internal exon (e.g., psi_correction_comparison.csv showing bulk PSI shift 0.67 → 0.92)
- Per-isoform misalignment flags (Aii / sum_{i≠j} Aij < 2)

## How to apply

Process reads through Longcell's adapter search, polyA detection, and cell barcode/UMI assignment, then align to GRCh38 with minimap2 v2.24 in spliced alignment mode against a splice-junction bed generated from the Gencode v39 GTF using paftools.js. Build a Splice object from the read-level splice-site table with creatSplice_from_df, and compute pre-correction PSI by tallying spliced-in vs. gene counts per internal exon (retaining constitutive internal exons with read count > 30). Apply Longcell's UMI-clustering misalignment correction, flagging isoform i as wrongly mapped when Aii / sum_{i≠j} Aij < 2 (default threshold), then recompute PSI. Rationale: within each UMI cluster reads are PCR duplicates, so a consensus alignment corrects read truncation and mapping errors that raw alignments cannot, e.g. the ~20% of VPS28-201 reads wrongly assigned a small middle exon, which biased constitutive-exon PSI from 0.92 down to 0.67 pre-correction.

## Related tools

- **Longcell** (Preprocessing pipeline that recovers cell barcodes/UMIs, corrects sequencing errors, and applies UMI-cluster consensus misalignment correction to splice alignments) — github.com/yuntianf/Longcell
- **Nanopore** (Long-read sequencing platform whose full-length reads carry the isoform-defining splice junctions)
- **minimap2** (Aligner run at v2.24 in spliced alignment mode against GRCh38 with the Gencode v39 splice-junction bed)
- **dplyr** (R data manipulation for tallying spliced-in vs. gene counts per internal exon and computing PSI)

## Examples

```
minimap2 -ax splice -u f --junc-bed gencode_v39.bed GRCh38.fa reads.fastq > aligned.sam; R: crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Average bulk PSI across retained constitutive internal exons (read count > 30) rises from ~0.67 pre-correction to ~0.92 post-correction
- Wrongly-mapped fraction of VPS28-201 reads at the small middle exon is ~20% pre-correction and drops after consensus correction
- Every read has an assigned cell barcode and UMI after Longcell preprocessing before alignment
- Splice junction bed reproduces Gencode v39 splice sites (bed generated with paftools.js from the GTF)
- Misalignment flag criterion holds: isoform i flagged when Aii / sum_{i≠j} Aij < 2

## Limitations

- Nanopore per-base accuracy of 85–99% causes sequencing errors, read truncation, and misalignment of small internal exons, leading to false-positive novel isoforms before correction
- Fixed-edit-distance UMI collapsing (e.g., Sicelore2, FLAMES) overestimates UMI counts under high sequencing error, causing inflated quantification; Longcell's statistical UMI clustering is the recommended alternative
- Read truncation means not all reads span the full transcript, so correction depends on consensus among PCR duplicates within a UMI cluster

## Evidence

- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI, which correspond to PCR duplicates: [evidence span withheld — anchor status: not_found]
