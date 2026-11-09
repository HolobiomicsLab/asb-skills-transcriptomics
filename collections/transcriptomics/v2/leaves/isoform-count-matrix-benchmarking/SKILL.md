---
name: isoform-count-matrix-benchmarking
description: Use when evaluating long-read (Nanopore) isoform quantification methods
  on single-cell or spatially barcoded data where an independent short-read ground-truth
  cell barcode/UMI matrix is available for comparison.
when_to_use_negative:
- Input data has no independent Illumina short-read ground-truth matrix to compare
  against, making Spearman correlation against supervision impossible
- The goal is downstream alternative splicing analysis (highly variable isoform identification,
  differential alternative splicing) rather than quantification method benchmarking
  — use Longcell's analysis functions instead
- Data are short-read-only without Nanopore long reads, since the denoising and consensus-correction
  steps address Nanopore-specific errors (85–99% per-base accuracy)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Preprocesses Nanopore FASTQ (adapter/polyA search, k-mer cell barcode matching
    with edit-distance cutoff 3, Needleman-Wunsch UMI clustering) to denoise reads
    before isoform quantification; benchmarked in combination with IsoQuant
  repo: github.com/yuntianf/Longcell
- name: IsoQuant
  role: Performs isoform quantification on Longcell-denoised reads and is also benchmarked
    standalone (v3.3, default parameters)
- name: FLAMES
  role: Baseline UMI-collapsing isoform quantification method benchmarked (git commit
    774e16ae53a1430e03081970827e93f1fbaecead, default parameters)
- name: Sicelore2
  role: Baseline method (v1) that collapses UMIs within a fixed edit distance; benchmarked
    for comparison against Longcell's statistical UMI recovery
- name: Nanopore
  role: Long-read sequencing platform generating the FASTQ input data with low per-base
    accuracy (85–99%) that motivates the denoising benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:d9b1dbd815b3ff980de96a2f2ec9ceef6bcc6112a68366fef2181a908bf7eca1
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
  - Nanopore
  - Sicelore2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isoform-count-matrix-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark single-cell/spatial Nanopore isoform quantification pipelines by comparing each method's cell-by-isoform count matrix against an Illumina-supervised ground-truth matrix using per-cell Spearman correlations averaged across cells.

## When to use

Use when evaluating long-read (Nanopore) isoform quantification methods on single-cell or spatially barcoded data where an independent short-read ground-truth cell barcode/UMI matrix is available for comparison.

## When NOT to use

- Input data has no independent Illumina short-read ground-truth matrix to compare against, making Spearman correlation against supervision impossible
- The goal is downstream alternative splicing analysis (highly variable isoform identification, differential alternative splicing) rather than quantification method benchmarking — use Longcell's analysis functions instead
- Data are short-read-only without Nanopore long reads, since the denoising and consensus-correction steps address Nanopore-specific errors (85–99% per-base accuracy)

## Inputs

- Nanopore FASTQ reads from single-cell or Visium spatially barcoded libraries
- Illumina-supervised cell barcode and UMI matrix used as ground truth (from Lebrigand et al. for the MOB Visium benchmark)
- GRCh38 reference genome
- Gencode v39 GTF annotation and paftools.js-derived splice junction bed file
- Method-specific software: Longcell, FLAMES, Sicelore2 v1, wf-single-cell v0.2.9, IsoQuant v3.3

## Outputs

- Per-method cell-by-isoform count matrix
- Per-cell Spearman correlation values against the Illumina-supervised matrix
- Mean per-cell Spearman correlation per method, written as a comparison table (spearman_correlations.csv)

## How to apply

Preprocess Nanopore FASTQ reads to recover cell barcodes and UMIs — e.g. with Longcell using k-mer-based barcode matching with an edit-distance cutoff of 3 and Needleman-Wunsch UMI clustering — because UMIs, unlike whitelisted cell barcodes, are shorter and random and thus more vulnerable to sequencing errors that inflate counts under fixed-edit-distance collapsing. Align denoised reads to GRCh38 with minimap2 in spliced mode (-ax splice --junc-bed --secondary=no --sam-hit-only) using a splice junction bed derived from the Gencode v39 GTF with paftools.js. Run each method (Longcell+IsoQuant, FLAMES, Sicelore2, wf-single-cell, IsoQuant alone) on the same FASTQ files with identical mapping software, reference genome, and annotation. Fully join each method's cell-by-isoform matrix with the Illumina-supervised ground truth by isoform, compute per-cell Spearman correlations, and average across all cells (e.g. 918 cells in the MOB Visium Nanopore benchmark). Validate reproduction by checking mean correlations match reported values (Longcell+IsoQuant 0.70, wf-single-cell 0.66, IsoQuant 0.58) within tolerance.

## Related tools

- **Longcell** (Preprocesses Nanopore FASTQ (adapter/polyA search, k-mer cell barcode matching with edit-distance cutoff 3, Needleman-Wunsch UMI clustering) to denoise reads before isoform quantification; benchmarked in combination with IsoQuant) — github.com/yuntianf/Longcell
- **IsoQuant** (Performs isoform quantification on Longcell-denoised reads and is also benchmarked standalone (v3.3, default parameters))
- **FLAMES** (Baseline UMI-collapsing isoform quantification method benchmarked (git commit 774e16ae53a1430e03081970827e93f1fbaecead, default parameters))
- **Sicelore2** (Baseline method (v1) that collapses UMIs within a fixed edit distance; benchmarked for comparison against Longcell's statistical UMI recovery)
- **Nanopore** (Long-read sequencing platform generating the FASTQ input data with low per-base accuracy (85–99%) that motivates the denoising benchmark)

## Examples

```
minimap2 -ax splice --junc-bed gencode_v39_junc.bed --secondary=no --sam-hit-only GRCh38.fa longcell_denoised.fq > aligned.sam
```

## Evaluation signals

- Mean per-cell Spearman correlation against the Illumina-supervised matrix matches reported values (Longcell+IsoQuant 0.70, wf-single-cell 0.66, IsoQuant 0.58) within a reasonable tolerance
- All methods were run with identical mapping software (minimap2 spliced mode), reference genome (GRCh38), and annotation (Gencode v39) so the comparison is controlled
- Cell-by-isoform matrices from each method fully join with the ground-truth matrix by isoform (complete join, no orphan keys)
- Correlations are averaged across all 918 cells of the MOB Visium Nanopore dataset, not a subsample
- Comparison table is written to spearman_correlations.csv with one mean value per method

## Limitations

- Fixed-edit-distance UMI collapsing in existing methods (Sicelore2, FLAMES) overestimates UMI counts under high sequencing error, causing inflated quantification
- Sequencing error accumulation and read truncation can misalign small internal exons, producing false-positive novel isoforms
- Benchmark accuracy is defined relative to Illumina-supervised barcode/UMI recovery, so it inherits any biases of that ground truth
- Biological mechanisms of intra-cell isoform switching remain unexplained and require further research

## Evidence
