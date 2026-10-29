---
name: single-cell-gene-barcode-count-matrix-generation
description: Use when raw FASTQ files from a droplet scRNA-seq platform (e.g. SRA
  SRP073767) must be converted into a per-cell UMI count matrix with barcode/UMI correction
  and PCR-duplicate removal.
when_to_use_negative:
- Input is already a gene-cell feature/count table (e.g. for downstream Seurat analysis
  of the 68k PBMCs) — alignment, barcode correction, and duplicate marking have already
  been done
- Data are full-length or plate-based RNA-seq reads without cell barcodes and UMIs,
  so the (cell barcode, UMI, gene ID) duplicate-marking tuple cannot be constructed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting to produce the gene-barcode matrix
  repo: github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to hg19; only confidently mapped (MAPQ=255) alignments
    proceed to counting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:41c1890bca97bb3e555cf2632e62577a0dcbb8ce6b14ff98047265889f327239
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell Ranger Single-Cell Software Suite
  - SRA SRP073767
  - STAR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-gene-barcode-count-matrix-generation

## Summary

Generates a gene-by-cell-barcode UMI count matrix (gene_barcode_matrix.csv) from droplet single-cell 3' RNA-seq FASTQ data by filtering and correcting cell barcodes and UMIs, marking PCR duplicates, and retaining only confidently mapped, non-duplicate reads with valid barcodes and UMIs.

## When to use

Use when raw FASTQ files from a droplet scRNA-seq platform (e.g. SRA SRP073767) must be converted into a per-cell UMI count matrix with barcode/UMI correction and PCR-duplicate removal.

## When NOT to use

- Input is already a gene-cell feature/count table (e.g. for downstream Seurat analysis of the 68k PBMCs) — alignment, barcode correction, and duplicate marking have already been done
- Data are full-length or plate-based RNA-seq reads without cell barcodes and UMIs, so the (cell barcode, UMI, gene ID) duplicate-marking tuple cannot be constructed

## Inputs

- Raw FASTQ files for a deposited sample (e.g. from SRA accession SRP073767)
- hg19 human reference genome
- Whitelist of candidate cell barcodes used for posterior-probability correction

## Outputs

- gene-barcode UMI count matrix (gene_barcode_matrix.csv) with one row per gene and one column per valid cell barcode

## How to apply

First align Read1 (the cDNA insert) to the hg19 reference genome with STAR, retaining only confidently mapped alignments (MAPQ=255). For each read, extract the cell barcode and correct it only if the posterior probability of a candidate barcode is at least 0.975, choosing the highest-posterior candidate; otherwise discard the read. Validate UMIs by keeping only those with sequencing quality score >10 that are not homopolymers, then collapse UMIs that are 1-Hamming-distance away from a more abundant UMI for the same cell barcode and gene. Mark PCR duplicates as reads sharing the same (cell barcode, UMI, gene ID) tuple and retain one read per tuple. Finally, tabulate surviving non-duplicate reads with valid barcodes and UMIs into a gene-by-cell-barcode UMI count matrix written to gene_barcode_matrix.csv.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting to produce the gene-barcode matrix) — github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to hg19; only confidently mapped (MAPQ=255) alignments proceed to counting)

## Evaluation signals

- Matrix counts derive only from reads with MAPQ=255 alignments, valid barcodes, valid UMIs, and unique (cell barcode, UMI, gene ID) tuples
- Every cell barcode in the matrix either matches the whitelist exactly or was corrected only when a candidate's posterior probability was >=0.975
- No retained UMI has quality score <=10 and no UMI is a homopolymer
- UMI collapsing resolves all 1-Hamming-distance pairs per (cell barcode, gene) so each collapsed UMI has at least as many reads as its corrected partner
- Output is a gene-by-cell-barcode matrix written to gene_barcode_matrix.csv

## Limitations

- Barcode correction is conservative: reads whose best candidate barcode posterior probability falls below 0.975 are discarded entirely
- Some earlier libraries used a 5 nt UMI and 5 bp Read2 instead of the standard lengths, introducing protocol heterogeneity across libraries
- Cell Ranger source is provided for informational purposes only; 10x does not support modifying, building, or running the code

## Evidence

- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] Barcode correction threshold: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] UMI validity and 1-Hamming-distance correction: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [readme] Cell Ranger role in the pipeline: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
- [discussion] STAR alignment of Read1: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
