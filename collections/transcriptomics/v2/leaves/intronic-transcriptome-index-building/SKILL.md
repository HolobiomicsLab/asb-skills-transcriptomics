---
name: intronic-transcriptome-index-building
description: Use when FASTQ files from 10x Genomics scRNA-seq (e.g. v2 chemistry)
  must be processed with scKB/kallisto-bustools and spliced AND unspliced counts (for
  RNA velocity or COPILOT input) are required.
when_to_use_negative:
- The analysis only needs filtered counts from standard Cell Ranger and spliced/unspliced
  matrices are not required (no RNA velocity or COPILOT-based QC planned)
- The read data are full-length plate-based scRNA-seq rather than 10x droplet chemistry,
  since the index is built to be 10xv2-compatible
- The required genome is not available as (and cannot be converted to) a BSgenome
  object, blocking BUSpaRse index construction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: kallisto
  role: Pseudoaligner whose index is built intron-inclusive; scKB incorporates kallisto
    for accelerated alignment
- name: bustools
  role: Wraps barcode/UMI correction and count matrix generation from BUS files after
    kallisto pseudoalignment
- name: scKB
  role: Wrapper script of kallisto and bustools used to run alignment and generate
    spliced/unspliced transcript count matrices
- name: BUSpaRse
  role: Extracts spliced+unspliced transcriptome-to-genome mappings from genes.gtf
    to build the index
- name: BSgenome
  role: Supplies the genome sequence object (e.g. BSgenome.Hsapiens) required for
    index building; custom genomes must be converted into BSgenome objects
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:cc08379299f506a590b0fb75c0ec8842889afe7d9806dbbc5a702f587345ca78
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - BSgenome
  - BUSpaRse
  - COPILOT
  - DoubletFinder
  - bustools
  - kallisto
  - scKB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# intronic-transcriptome-index-building

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a 10x-chemistry-compatible kallisto index that includes intronic sequence so that scKB can produce both spliced and unspliced gene-by-cell UMI count matrices, the recommended input for COPILOT quality filtering.

## When to use

Use when FASTQ files from 10x Genomics scRNA-seq (e.g. v2 chemistry) must be processed with scKB/kallisto-bustools and spliced AND unspliced counts (for RNA velocity or COPILOT input) are required.

## When NOT to use

- The analysis only needs filtered counts from standard Cell Ranger and spliced/unspliced matrices are not required (no RNA velocity or COPILOT-based QC planned)
- The read data are full-length plate-based scRNA-seq rather than 10x droplet chemistry, since the index is built to be 10xv2-compatible
- The required genome is not available as (and cannot be converted to) a BSgenome object, blocking BUSpaRse index construction

## Inputs

- 10x Genomics scRNA-seq FASTQ files (e.g. pbmc_1k_v2, v2 chemistry)
- Genome annotation GTF (e.g. refdata-gex-GRCh38-2020-A genes.gtf or TAIR10)
- Reference genome sequence as a BSgenome object (e.g. BSgenome.Hsapiens)

## Outputs

- 10xv2-compatible intron-inclusive kallisto index
- Transcriptome-to-genome (t2g) mappings from genes.gtf (spliced + unspliced)
- Raw spliced and unspliced gene-by-cell UMI count matrices (via scKB/kallisto/bustools) in a sample-named directory

## How to apply

Take the cell-sample FASTQs (e.g. pbmc_1k_v2, 10x v2 chemistry) and a genome annotation (e.g. refdata-gex-GRCh38-2020-A genes.gtf, or TAIR10 for Arabidopsis). Using BUSpaRse, extract spliced+unspliced transcriptome-to-genome mappings from genes.gtf, and supply the genome as a BSgenome object (e.g. BSgenome.Hsapiens; if the genome is not already available as BSgenome it must be converted). Use these mappings to build a 10xv2-compatible intron-inclusive kallisto index, which scKB (a wrapper of kallisto and bustools) then uses for pseudoalignment and busTools count capture, emitting raw spliced and unspliced matrices in a directory named after the sample. The rationale is that kallisto pseudoalignment plus intronic inclusion is what lets scKB emit the spliced/unspliced matrix pair that COPILOT expects and that RNA-velocity analysis requires, and it runs ~30x faster than Cell Ranger 3.1.

## Related tools

- **kallisto** (Pseudoaligner whose index is built intron-inclusive; scKB incorporates kallisto for accelerated alignment)
- **bustools** (Wraps barcode/UMI correction and count matrix generation from BUS files after kallisto pseudoalignment)
- **scKB** (Wrapper script of kallisto and bustools used to run alignment and generate spliced/unspliced transcript count matrices)
- **BUSpaRse** (Extracts spliced+unspliced transcriptome-to-genome mappings from genes.gtf to build the index)
- **BSgenome** (Supplies the genome sequence object (e.g. BSgenome.Hsapiens) required for index building; custom genomes must be converted into BSgenome objects)

## Evaluation signals

- The index is usable with 10xv2 chemistry barcodes/UMIs, i.e. scKB completes pseudoalignment and bustools count capture without barcode-chemistry errors
- Running scKB on the FASTQs produces BOTH raw spliced and unspliced gene-by-cell matrices with matching row (gene) and column (barcode) names, in a directory named after the sample
- The generated matrices are accepted directly as COPILOT input (spliced.mtx / unspliced.mtx / total.mtx arguments) without re-derivation
- Processing time is far faster than a Cell Ranger 3.1 run (protocol reports ~30x speedup), providing a sanity check on the kallisto/bustools path

## Limitations

- The required genome may not be available as a BSgenome object; users need to convert a customized genome into a BSgenome object before index construction
- COPILOT's own annotation downstream is limited: it only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- Troubleshooting coverage is incomplete — the protocol's Problem 3 addresses scKB failing to run during 'align raw reads to genome via scKB' with no solution given for several related problems

## Evidence
