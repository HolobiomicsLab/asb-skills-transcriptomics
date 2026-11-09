---
name: scrna-seq-read-pseudoalignment
description: Use when raw scRNA-seq reads (e.g., 10x Genomics v3 FASTQ, such as sample
  GSM4626009 from GSE152766) need fast alignment and quantification into spliced and
  unspliced count matrices for downstream quality filtering with COPILOT.
when_to_use_negative:
- You already have raw spliced and unspliced counts matrices from scKB — the next
  step is COPILOT quality filtering, not re-alignment
- You need a spliced/unspliced-aware reference for an organism whose genome cannot
  be converted into the required kallisto/BSgenome object — converting a customized
  genome is required first
- You require alignment-based results rather than pseudoalignment (e.g., splice-aware
  alignment for novel transcript discovery)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKB
  role: Wrapper bash script that orchestrates pseudoalignment and transcript count
    matrix generation; invoked directly on the toy data folder with the kallisto index
    and barcode whitelist
- name: kallisto
  role: Performs pseudoalignment of the 10x v3 reads against the cDNA_introns_10xv3.idx
    index, called internally by scKB
- name: bustools
  role: Performs barcode correction and transcript counting on pseudoaligned reads,
    called internally by scKB
- name: Cell Ranger
  role: Reference comparison baseline; scKB pseudoalignment is about ~30 times faster
    than CellRanger 3.1 in processing time
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:0783ae9c4405302c6c8b12b379b6649f19289bb82369d681186a0085350b2066
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - Cell Ranger
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# scrna-seq-read-pseudoalignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pseudoalign raw single-cell RNA-seq FASTQ reads and generate spliced/unspliced gene-by-cell count matrices using scKB, a wrapper script around kallisto and bustools. It runs about 30 times faster than CellRanger 3.1, which matters for large scRNA-seq datasets such as the Arabidopsis thaliana root atlas.

## When to use

Use when raw scRNA-seq reads (e.g., 10x Genomics v3 FASTQ, such as sample GSM4626009 from GSE152766) need fast alignment and quantification into spliced and unspliced count matrices for downstream quality filtering with COPILOT.

## When NOT to use

- You already have raw spliced and unspliced counts matrices from scKB — the next step is COPILOT quality filtering, not re-alignment
- You need a spliced/unspliced-aware reference for an organism whose genome cannot be converted into the required kallisto/BSgenome object — converting a customized genome is required first
- You require alignment-based results rather than pseudoalignment (e.g., splice-aware alignment for novel transcript discovery)

## Inputs

- Raw scRNA-seq FASTQ reads from a 10x Genomics v3 library (e.g., the toy_data subset of sample GSM4626009, GSE152766), placed in a folder named identically to the scKB sample.name argument
- Prebuilt kallisto transcriptome index containing cDNA plus intronic sequences (cDNA_introns_10xv3.idx)
- 10x Genomics v3 barcode whitelist (10xv3_whitelist.txt)

## Outputs

- Spliced gene-by-cell count matrix (.mtx) with row and column names
- Unspliced gene-by-cell count matrix (.mtx) with row and column names
- Spliced and unspliced barcodes.txt files and genes.txt files
- Alignment/counting statistics files (inspect. and run_info.) comparable to CellRanger 3.1 utility/summary files, needed for downstream COPILOT quality filtering

## How to apply

Place the raw FASTQ reads (e.g., the 80 MB toy_data subset of ~1 million reads from the wild-type Arabidopsis root scRNA-seq sample GSM4626009) in a working-directory folder named identically to the scKB sample.name argument. Retrieve the prebuilt kallisto transcriptome index cDNA_introns_10xv3.idx (cDNA plus intronic sequences so that both spliced and unspliced transcripts can be quantified) and the 10x Genomics v3 barcode whitelist. Execute the scKB bash wrapper, which calls kallisto for pseudoalignment of reads against cDNA_introns_10xv3.idx and bustools for barcode correction and transcript counting; on the toy data this takes about 30 s. Collect the gene-by-cell spliced and unspliced .mtx matrices (each with column and row names) plus stats files from the sample folder; these raw matrices are the recommended input for COPILOT quality filtering.

## Related tools

- **scKB** (Wrapper bash script that orchestrates pseudoalignment and transcript count matrix generation; invoked directly on the toy data folder with the kallisto index and barcode whitelist)
- **kallisto** (Performs pseudoalignment of the 10x v3 reads against the cDNA_introns_10xv3.idx index, called internally by scKB)
- **bustools** (Performs barcode correction and transcript counting on pseudoaligned reads, called internally by scKB)
- **Cell Ranger** (Reference comparison baseline; scKB pseudoalignment is about ~30 times faster than CellRanger 3.1 in processing time)

## Examples

```
./scKB -f ./toy_data -i ./cDNA_introns_10xv3.idx -s 10xv3 -w ./10xv3_whitelist.txt
```

## Evaluation signals

- Both spliced and unspliced .mtx count matrices exist in the sample output folder
- Each count matrix carries row names (genes) and column names (cell barcodes)
- The 8 expected output files are produced: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and .mtx gene-by-cell matrices
- Stats files are present in the sample folder, as required for downstream COPILOT quality filtering
- On the 1-million-read toy_data subset the run completes in roughly 30 s

## Limitations

- scKB is reported to fail to run properly in some cases (troubleshooting Problem 3 of the protocol); troubleshooting coverage for scKB execution problems is limited
- The cDNA+introns index must be built from the appropriate genome/annotation (e.g., TAIR10 for Arabidopsis); the required genome may not be available as a BSgenome object and users may need to convert a customized genome
- The downstream COPILOT annotation only supports the root of Arabidopsis thaliana for cell type and developmental stage annotation

## Evidence

- [results] Command and outputs of the toy-data run: [evidence span withheld — anchor status: not_found]
