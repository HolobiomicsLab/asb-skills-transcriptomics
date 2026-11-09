---
name: single-cell-read-pseudoalignment
description: Use when raw 10x scRNA-seq FASTQ files (e.g., SRR12046119/SRR12046120
  Arabidopsis col0 with cDNA_introns_10xv3.idx, or human pbmc_1k_v2) must be pseudoaligned
  quickly and converted into raw spliced and unspliced gene-by-cell count matrices.
when_to_use_negative:
- The goal is an official 10x Genomics analysis chain with the standard Cell Ranger
  web_summary and feature-barcode outputs rather than speed-optimized pseudoalignment.
- Input is already a gene-by-cell count matrix — scKB operates on raw FASTQ reads,
  and COPILOT should be applied downstream instead.
- A BSgenome object is unavailable for the organism and no customized genome has been
  converted, preventing index construction with BUSpaRse/BSgenome.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKB
  role: Wrapper script that drives kallisto pseudoalignment and bustools transcript
    count matrix generation on the FASTQ inputs
- name: kallisto
  role: Pseudoalignment engine used by scKB to align cDNA reads to the intron-inclusive
    transcriptome index
- name: bustools
  role: Corrects cell barcodes and generates the spliced and unspliced transcript
    count matrices from BUS files
- name: COPILOT
  role: Downstream consumer of the raw spliced and unpliced counts matrices for non-arbitrary
    quality filtering
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Cell Ranger
  role: Baseline alignment tool used as the runtime comparison target in the benchmark
    (Cell Ranger 6.1.2, 'cellranger count' with 'nosecondary')
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:a6cb17b688fc12e4fb94e63273fd7f20727b880cc29cdbf63522ef3e229767c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - BSgenome
  - COPILOT
  - Cell Ranger
  - DoubletFinder
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# single-cell-read-pseudoalignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fast pseudoalignment of single-cell RNA-seq raw reads using scKB, a wrapper script of kallisto and bustools, to accelerate alignment and gene-by-cell transcript count matrix generation relative to Cell Ranger (~30x faster in processing time).

## When to use

Use when raw 10x scRNA-seq FASTQ files (e.g., SRR12046119/SRR12046120 Arabidopsis col0 with cDNA_introns_10xv3.idx, or human pbmc_1k_v2) must be pseudoaligned quickly and converted into raw spliced and unspliced gene-by-cell count matrices.

## When NOT to use

- The goal is an official 10x Genomics analysis chain with the standard Cell Ranger web_summary and feature-barcode outputs rather than speed-optimized pseudoalignment.
- Input is already a gene-by-cell count matrix — scKB operates on raw FASTQ reads, and COPILOT should be applied downstream instead.
- A BSgenome object is unavailable for the organism and no customized genome has been converted, preventing index construction with BUSpaRse/BSgenome.

## Inputs

- 10x Genomics scRNA-seq FASTQ files (e.g., SRR12046119/SRR12046120, pbmc_1k_v2)
- Intron-inclusive kallisto index (e.g., cDNA_introns_10xv3.idx built from TAIR10 via BUSpaRse; or a 10xv2-compatible GRCh38 index)
- Reference genome annotation (genes.gtf, e.g., TAIR10 or refdata-gex-GRCh38-2020-A)

## Outputs

- Raw spliced gene-by-cell count matrix with row (gene) and column (cell barcode) names
- Raw unspliced gene-by-cell count matrix for downstream COPILOT input and RNA velocity

## How to apply

Dump SRA runs to FASTQ, then convert the reads into the scKB-compatible layout that separates barcode/UMI reads from cDNA reads, placing them in a directory whose name matches the copilot 'sample.name' convention. Run the scKB wrapper, which drives kallisto pseudoalignment of the cDNA reads against an intron-inclusive kallisto index (e.g., cDNA_introns_10xv3.idx built from TAIR10 or GRCh38 via BUSpaRse and BSgenome) followed by bustools barcode counting. Export the resulting gene-by-cell spliced and unspliced count matrices with proper row (gene) and column (cell barcode) names, since it is recommended to use these raw spliced and unpliced counts matrices as the input of COPILOT. Validate runtime against the benchmark: on the full col0 dataset (386 million reads), scKB takes about 22 minutes versus 640 minutes for Cell Ranger 6.1.2, consistent with the ~30-fold processing-time advantage.

## Related tools

- **scKB** (Wrapper script that drives kallisto pseudoalignment and bustools transcript count matrix generation on the FASTQ inputs)
- **kallisto** (Pseudoalignment engine used by scKB to align cDNA reads to the intron-inclusive transcriptome index)
- **bustools** (Corrects cell barcodes and generates the spliced and unspliced transcript count matrices from BUS files)
- **COPILOT** (Downstream consumer of the raw spliced and unpliced counts matrices for non-arbitrary quality filtering) — https://github.com/Hsu-Che-Wei/COPILOT
- **Cell Ranger** (Baseline alignment tool used as the runtime comparison target in the benchmark (Cell Ranger 6.1.2, 'cellranger count' with 'nosecondary'))

## Evaluation signals

- Both spliced and unspliced gene-by-cell matrices are produced with proper column (cell barcode) and row (gene) names.
- Run time on the full col0 dataset (386 million reads, 10x Genomics v3) is roughly 22 minutes, versus 640 minutes for Cell Ranger 6.1.2 on the same data.
- The speedup ratio between Cell Ranger and the scKB step is approximately 30-fold, consistent with the reported processing-time advantage.
- Matrix directory names match the 'sample.name' convention expected by the downstream copilot() call so the raw matrices can be passed directly to COPILOT.

## Limitations

- scKB fails to run properly in some environments, flagged as Problem 3 in the troubleshooting section with no solution provided.
- Building the kallisto index requires the genome as a BSgenome object; the required genome may not be available and users need to convert a customized genome into a BSgenome object.
- Advantages over Cell Ranger are stated relative to CellRanger 3.1; the runtime benchmark uses CellRanger 6.1.2 with the 'nosecondary' flag.

## Evidence
