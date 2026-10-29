---
name: bash-based-scrna-pipeline-execution
description: Use when raw 10xv3 scRNA-seq FASTQ reads need alignment and quantification
  into spliced/unspliced count matrices, especially when CellRanger is too slow or
  RNA velocity / splicing-dynamics downstream analyses are planned.
when_to_use_negative:
- Input is already a gene-by-cell UMI count matrix — scKB operates on raw FASTQ reads,
  and the matrices should instead be fed directly into COPILOT quality filtering
- The reads are not from a 10x Genomics v3 chemistry library, since the workflow is
  parameterized with the 10xv3 index and the 10xv3 whitelist
- A splice-aware whole-genome alignment is required (e.g. genome browser track generation),
  as scKB performs pseudoalignment for transcript counting only
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3526
tools:
- name: scKB
  role: Bash wrapper script that orchestrates the kallisto/bustools run and organizes
    outputs into the sample folder
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: kallisto
  role: Performs pseudoalignment of the FASTQ reads against the cDNA_introns_10xv3.idx
    index
- name: bustools
  role: Performs barcode correction and transcript counting after pseudoalignment
- name: Cell Ranger
  role: Reference comparison tool — scKB is ~30x faster than CellRanger 3.1 in processing
    time
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:32552d82dbf6d093044832bab04d61d225b3ef3eb4822584c2897b6bdafe5020
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
  - Cell Ranger
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3526
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bash-based-scrna-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes the scKB bash wrapper script, which orchestrates kallisto pseudoalignment against a cDNA+introns index and bustools barcode correction to generate spliced and unspliced gene-by-cell count matrices from 10x Genomics scRNA-seq FASTQs, roughly 30x faster than CellRanger 3.1.

## When to use

Use when raw 10xv3 scRNA-seq FASTQ reads need alignment and quantification into spliced/unspliced count matrices, especially when CellRanger is too slow or RNA velocity / splicing-dynamics downstream analyses are planned.

## When NOT to use

- Input is already a gene-by-cell UMI count matrix — scKB operates on raw FASTQ reads, and the matrices should instead be fed directly into COPILOT quality filtering
- The reads are not from a 10x Genomics v3 chemistry library, since the workflow is parameterized with the 10xv3 index and the 10xv3 whitelist
- A splice-aware whole-genome alignment is required (e.g. genome browser track generation), as scKB performs pseudoalignment for transcript counting only

## Inputs

- Raw 10x Genomics v3 scRNA-seq FASTQ reads (e.g. toy_data subset of GSM4626009 from GSE152766) in a folder named to match sample.name
- Prebuilt kallisto transcriptome index containing cDNA plus intronic sequences (cDNA_introns_10xv3.idx)
- 10x Genomics v3 barcode whitelist (10xv3_whitelist.txt)

## Outputs

- Spliced gene-by-cell count matrix (.mtx) with row and column names
- Unspliced gene-by-cell count matrix (.mtx) with row and column names
- genes.txt and spliced/unspliced barcodes.txt files
- Alignment and counting statistics files (inspect., run_info.) required as COPILOT input

## How to apply

Place the raw FASTQ subset (e.g. the 80 MB toy_data from GSM4626009, GSE152766) in a working-directory folder named identically to the scKB sample.name argument, since scKB requires this naming convention for COPILOT compatibility downstream. Retrieve the prebuilt kallisto transcriptome index (e.g. cDNA_introns_10xv3.idx, containing cDNA plus intronic sequences to permit spliced and unspliced quantification) and the 10x Genomics v3 barcode whitelist, then execute the scKB wrapper with -f (FASTQ dir), -i (index), -s (chemistry, 10xv3), and -w (whitelist) flags. scKB internally calls kallisto for pseudoalignment of the reads and bustools for barcode correction and transcript counting; on a 1-million-read toy dataset the run completes in about 30 s versus far longer under CellRanger 3.1. Collect the eight output files from the sample folder — inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and .mtx gene-by-cell matrices — and validate that both count matrices carry row and column names and that the stats files exist for downstream COPILOT quality filtering.

## Related tools

- **scKB** (Bash wrapper script that orchestrates the kallisto/bustools run and organizes outputs into the sample folder) — https://github.com/Hsu-Che-Wei/COPILOT
- **kallisto** (Performs pseudoalignment of the FASTQ reads against the cDNA_introns_10xv3.idx index)
- **bustools** (Performs barcode correction and transcript counting after pseudoalignment)
- **Cell Ranger** (Reference comparison tool — scKB is ~30x faster than CellRanger 3.1 in processing time)

## Examples

```
./scKB -f ./toy_data -i ./cDNA_introns_10xv3.idx -s 10xv3 -w ./10xv3_whitelist.txt
```

## Evaluation signals

- All 8 expected output files appear in the sample folder: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and the .mtx matrices
- Both spliced and unspliced .mtx matrices exist and carry row (gene) and column (barcode) names
- The working directory containing the FASTQs is named identically to the sample.name argument, enabling direct consumption by COPILOT
- The stats files (inspect., run_info.) are present, as they are required for COPILOT quality filtering
- Runtime is consistent with the pseudoalignment speed expectation (~30x faster than CellRanger 3.1; ~30 s for the 1-million-read toy dataset)

## Limitations

- The output matrices are intended as input to COPILOT, whose current version supports cell type and developmental stage annotation only for the root of Arabidopsis thaliana
- COPILOT downstream of scKB may fail in conda-installed R environments lacking an X11 device, which is required to generate feature plots for the summary file
- Per the paper's troubleshooting notes, no solutions are provided for some scKB execution failures (Problem 3, related to 'align raw reads to genome via scKB')

## Evidence
