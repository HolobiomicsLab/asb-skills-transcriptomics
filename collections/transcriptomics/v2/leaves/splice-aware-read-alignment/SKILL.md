---
name: splice-aware-read-alignment
description: Use when preprocessing Nanopore single-cell or spatially barcoded long
  reads for isoform-level analysis and you need spliced alignments that respect known
  transcript junctions before UMI/barcode denoising and isoform counting.
when_to_use_negative:
- Input reads are short Illumina reads where splice-aware long-read alignment (minimap2
  -ax splice) is not appropriate.
- The analysis goal is genome-level structural variant calling rather than transcript/isoform-level
  quantification.
- Reads are already denoised, UMI-collapsed, and aligned; re-aligning would duplicate
  work or break existing UMI-to-read linkages.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Spliced alignment of Nanopore long reads to GRCh38 using Gencode v39-derived
    splice junction bed
- name: paftools.js
  role: Generate splice junction BED file from Gencode v39 GTF for minimap2 --junc-bed
    input
- name: Longcell
  role: Upstream cell barcode/UMI recovery and downstream isoform quantification pipeline
    consuming splice-aware alignments
  repo: github.com/yuntianf/Longcell
- name: IsoQuant
  role: Isoform quantification from Longcell-denoised, splice-aligned reads
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:a7d328128d19de1c19042fdadd5e6ee44095ae7b8454abf30f05e7ecfb66acf6
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
  - Sicelore2
  - minimap2
  - paftools.js
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# splice-aware-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align Nanopore long reads to GRCh38 with minimap2 in spliced alignment mode, using a Gencode v39 GTF-derived splice junction bed to guide junction detection and suppress ambiguous secondary alignments prior to isoform quantification.

## When to use

Use when preprocessing Nanopore single-cell or spatially barcoded long reads for isoform-level analysis and you need spliced alignments that respect known transcript junctions before UMI/barcode denoising and isoform counting.

## When NOT to use

- Input reads are short Illumina reads where splice-aware long-read alignment (minimap2 -ax splice) is not appropriate.
- The analysis goal is genome-level structural variant calling rather than transcript/isoform-level quantification.
- Reads are already denoised, UMI-collapsed, and aligned; re-aligning would duplicate work or break existing UMI-to-read linkages.

## Inputs

- Nanopore long-read FASTQ files from single-cell or Visium spatially barcoded libraries (e.g., MOB Visium Nanopore, 918 cells)
- Reference genome GRCh38 FASTA
- Splice junction BED file derived from Gencode v39 GTF using paftools.js

## Outputs

- Splice-aware BAM/SAM alignment of long reads to GRCh38 with secondary alignments suppressed
- Cell-by-isoform count matrix after downstream quantification (e.g., via Longcell+IsoQuant)

## How to apply

Run minimap2 v2.24 (or equivalent) with `-ax splice --junc-bed --secondary=no --sam-hit-only`, supplying a splice junction bed generated from the Gencode v39 GTF via paftools.js so the aligner prefers annotated donor/acceptor pairs. Align reads to GRCh38 so that junction-spanning long reads split correctly, which reduces false-positive novel isoforms caused by wrongly mapped small internal exons under high sequencing error. Use identical mapping software, reference genome, and annotation across all methods being benchmarked (e.g., Longcell+IsoQuant, FLAMES, Sicelore2, wf-single-cell) so downstream per-cell Spearman correlations against Illumina-supervised ground truth are comparable. The output BAM/sam-hit-only alignments then feed UMI-aware consensus collapsing (e.g., Longcell's Needleman-Wunsch UMI clustering) and isoform quantification tools such as IsoQuant.

## Related tools

- **minimap2** (Spliced alignment of Nanopore long reads to GRCh38 using Gencode v39-derived splice junction bed)
- **paftools.js** (Generate splice junction BED file from Gencode v39 GTF for minimap2 --junc-bed input)
- **Longcell** (Upstream cell barcode/UMI recovery and downstream isoform quantification pipeline consuming splice-aware alignments) — github.com/yuntianf/Longcell
- **IsoQuant** (Isoform quantification from Longcell-denoised, splice-aligned reads)

## Examples

```
minimap2 -ax splice --junc-bed gencode_v39_junc.bed --secondary=no --sam-hit-only GRCh38.fa reads.fastq > aligned.sam
```

## Evaluation signals

- Alignment command includes `--junc-bed` pointing to a Gencode v39-derived splice junction BED; verify the BED was generated via paftools.js from the same GTF.
- `--secondary=no` and `--sam-hit-only` flags are present so ambiguous multi-mapping reads do not inflate isoform counts.
- Benchmarked methods all use identical mapping software, reference genome (GRCh38), and annotation, so per-cell Spearman correlations are directly comparable.
- Mean per-cell Spearman correlation against Illumina-supervised quantification reaches ~0.70 (Longcell+IsoQuant) or ~0.66 (wf-single-cell) on the MOB Visium Nanopore benchmark, within reasonable tolerance of reported values.
- No false-positive novel isoforms attributable to misaligned small internal exons, checked by inspecting novel isoform calls in well-annotated genes.

## Limitations

- Nanopore per-base accuracy (85–99%) and read truncation can still cause misidentification of isoforms and false-positive novel isoforms from wrongly mapped small internal exons even with splice-aware alignment.
- Performance depends on annotation quality: junction-guided alignment assumes the Gencode v39 (or equivalent) annotation captures true splice junctions, potentially biasing against novel isoforms.
- Downstream quantification bias persists if upstream UMI/barcode recovery is inaccurate, so alignment alone does not guarantee accurate isoform counts.

## Evidence

- [other] Alignment command specification: [evidence span withheld — anchor status: not_found]
- [other] Benchmark comparison with identical parameters: [evidence span withheld — anchor status: not_found]
