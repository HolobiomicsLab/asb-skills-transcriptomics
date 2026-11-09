---
name: scrna-seq-run-statistics-collection
description: Use when a scKB pseudoalignment run on 10xv3 FASTQ data (e.g. the 80
  MB toy_data from GSM4626009, GSE152766) has finished and you must verify and gather
  its 8 output files for downstream COPILOT input.
when_to_use_negative:
- Input is already a quality-filtered count matrix — COPILOT expects the raw spliced/unspliced
  matrices from scKB, so re-running alignment and re-collecting stats would be redundant
- The target tool is Cell Ranger; scKB statistics collection applies only to scKB
  (kallisto + bustools) runs, not Cell Ranger output folders
- You need cell type annotation for a species/tissue other than Arabidopsis thaliana
  root — COPILOT annotation downstream of these stats only supports that case
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKB
  role: Wrapper script executed to generate the statistics files and spliced/unspliced
    count matrices that this skill collects
- name: kallisto
  role: Called by scKB to pseudoalign the reads against cDNA_introns_10xv3.idx, producing
    the alignment statistics gathered here
- name: bustools
  role: Called by scKB for barcode correction and transcript counting, producing the
    barcodes.txt, genes.txt and .mtx outputs
- name: COPILOT
  role: Downstream consumer of the collected raw spliced/unspliced matrices and stats
    for non-arbitrary quality control
  repo: github.com/Hsu-Che-Wei/COPILOT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:26ad8832e8d84fec3f4a8bc457a3f28ad9252bf47b780033690fe8568079ef6b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - COPILOT
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-run-statistics-collection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collects the alignment/counting statistics and matrix files (inspect., run_info., spliced/unspliced barcodes.txt, genes.txt, and .mtx gene-by-cell count matrices) emitted by a scKB run so they can be validated and passed downstream to COPILOT quality filtering.

## When to use

Use when a scKB pseudoalignment run on 10xv3 FASTQ data (e.g. the 80 MB toy_data from GSM4626009, GSE152766) has finished and you must verify and gather its 8 output files for downstream COPILOT input.

## When NOT to use

- Input is already a quality-filtered count matrix — COPILOT expects the raw spliced/unspliced matrices from scKB, so re-running alignment and re-collecting stats would be redundant
- The target tool is Cell Ranger; scKB statistics collection applies only to scKB (kallisto + bustools) runs, not Cell Ranger output folders
- You need cell type annotation for a species/tissue other than Arabidopsis thaliana root — COPILOT annotation downstream of these stats only supports that case

## Inputs

- FASTQ toy_data subset (80 MB, ~1 million reads) of sample GSM4626009 from GSE152766, placed in a folder named identically to the scKB sample.name argument
- Prebuilt kallisto transcriptome index including cDNA plus intronic sequences (cDNA_introns_10xv3.idx)
- 10x Genomics v3 barcode whitelist (10xv3_whitelist.txt)

## Outputs

- 8 output files per sample folder: inspect. and run_info. statistics files, spliced and unspliced barcodes.txt, genes.txt
- Spliced gene-by-cell count matrix (.mtx) with row and column names
- Unspliced gene-by-cell count matrix (.mtx) with row and column names

## How to apply

Run the scKB wrapper script (which calls kallisto for pseudoalignment and bustools for barcode correction and counting) against the kallisto cDNA+introns index (cDNA_introns_10xv3.idx) and the 10x Genomics v3 barcode whitelist, with the FASTQ folder named identically to the scKB sample.name argument. After the run (about 30 s on the 1-million-read toy data), inspect the sample output folder for 8 files: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and the .mtx gene-by-cell count matrices. Confirm that both the spliced and unspliced count matrices exist and each carries row and column names, since COPILOT's spliced.mtx/unspliced.mtx arguments require named matrices. Keep the stats files (run_info./inspect.) because COPILOT's summary/utility output is directly comparable to CellRanger 3.1 and downstream steps expect these statistics. The raw spliced and unspliced matrices collected here are the recommended input to copilot() for quality filtering.

## Related tools

- **scKB** (Wrapper script executed to generate the statistics files and spliced/unspliced count matrices that this skill collects)
- **kallisto** (Called by scKB to pseudoalign the reads against cDNA_introns_10xv3.idx, producing the alignment statistics gathered here)
- **bustools** (Called by scKB for barcode correction and transcript counting, producing the barcodes.txt, genes.txt and .mtx outputs)
- **COPILOT** (Downstream consumer of the collected raw spliced/unspliced matrices and stats for non-arbitrary quality control) — github.com/Hsu-Che-Wei/COPILOT

## Examples

```
./scKB -f ./toy_data -i ./cDNA_introns_10xv3.idx -s 10xv3 -w ./10xv3_whitelist.txt
```

## Evaluation signals

- Exactly 8 output files appear in the sample folder: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and the .mtx matrices
- Both spliced and unspliced .mtx matrices exist and each carries row (gene) and column (barcode) names, as required by COPILOT's matrix arguments
- The run completes in roughly 30 s on the 1-million-read toy_data subset, consistent with the reported ~30x speedup over CellRanger 3.1
- Stats files (run_info./inspect.) are present alongside the matrices so downstream COPILOT quality filtering has the required inputs
- The toy_data folder name matches the scKB sample.name argument, so outputs land in the expected sample directory

## Limitations

- scKB may fail to run properly during the 'align raw reads to genome via scKB' section, and no troubleshooting solution is provided for that problem
- A mismatch between the FASTQ folder name and the scKB sample.name argument will break output collection, since COPILOT expects sample.name to be the directory containing spliced and unspliced matrices
- The run_info./inspect. statistics are comparable to CellRanger 3.1 but not to later Cell Ranger versions

## Evidence

- [other] Running scKB on the toy data takes about 30 s and produces 8 output files: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and .mtx gene-by-cell matrices.: [evidence span withheld — anchor status: not_found]
- [other] Validation requires both spliced and unspliced matrices with names plus stats files for COPILOT.: [evidence span withheld — anchor status: not_found]
