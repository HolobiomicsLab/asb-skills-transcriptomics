---
name: 10x-genomics-barcode-whitelist-processing
description: Use when processing raw 10x Genomics v3 scRNA-seq FASTQ reads (e.g. Arabidopsis
  root sample GSM4626009 from GSE152766) with scKB/kallisto/bustools and a barcode
  whitelist is needed for barcode correction and counting.
when_to_use_negative:
- Input is not 10x Genomics v3 chemistry data — the whitelist and -s 10xv3 setting
  are specific to 10x v3 barcodes
- A reference-based aligner is required instead of pseudoalignment, since scKB relies
  on kallisto pseudoalignment rather than full read alignment
- Data are already processed into count matrices — COPILOT, not scKB, should be applied
  to existing spliced/unspliced matrices
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKB
  role: Wrapper bash script that invokes kallisto and bustools for accelerated alignment
    and transcript count matrix generation; takes -f (read folder), -i (index), -s
    (chemistry), -w (whitelist) arguments
- name: kallisto
  role: Performs pseudoalignment of reads against the cDNA_introns_10xv3.idx transcriptome
    index, called from within scKB
- name: bustools
  role: Performs 10xv3 barcode correction against the whitelist and transcript counting,
    called from within scKB
- name: COPILOT
  role: Downstream consumer of the spliced/unspliced matrices and stats files produced
    by the scKB whitelist-processing run
  repo: github.com/Hsu-Che-Wei/COPILOT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:5bfa123f46afbafed9cd7580575a54da52641d35a894712e0a124c96377a3ffc
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
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# 10x-genomics-barcode-whitelist-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correct and count 10x Genomics v3 cell barcodes during scRNA-seq preprocessing by supplying the 10xv3 barcode whitelist to the bustools step inside the scKB wrapper, enabling spliced/unspliced gene-by-cell count matrix generation from FASTQ reads.

## When to use

Use when processing raw 10x Genomics v3 scRNA-seq FASTQ reads (e.g. Arabidopsis root sample GSM4626009 from GSE152766) with scKB/kallisto/bustools and a barcode whitelist is needed for barcode correction and counting.

## When NOT to use

- Input is not 10x Genomics v3 chemistry data — the whitelist and -s 10xv3 setting are specific to 10x v3 barcodes
- A reference-based aligner is required instead of pseudoalignment, since scKB relies on kallisto pseudoalignment rather than full read alignment
- Data are already processed into count matrices — COPILOT, not scKB, should be applied to existing spliced/unspliced matrices

## Inputs

- Raw 10x Genomics v3 FASTQ reads (e.g. the 80 MB toy_data subset of GSM4626009 from GSE152766) in a folder named to match the scKB sample.name argument
- Prebuilt kallisto transcriptome index including cDNA plus intronic sequences (cDNA_introns_10xv3.idx)
- 10x Genomics v3 barcode whitelist file (10xv3_whitelist.txt)
- Chemistry specification (10xv3) passed to scKB via the -s flag

## Outputs

- Spliced and unspliced gene-by-cell count matrices (.mtx with column and row names) in the sample output folder
- genes.txt listing gene identifiers
- Spliced and unspliced barcodes.txt files listing corrected cell barcodes
- Alignment/counting statistics files (inspect., run_info.) required for downstream COPILOT quality filtering

## How to apply

Place the raw FASTQ subset in a working-directory folder named identically to the scKB sample.name argument, then run the scKB bash wrapper with the prebuilt kallisto transcriptome index (cDNA_introns_10xv3.idx, containing cDNA plus intronic sequences for spliced/unspliced quantification) and the 10x Genomics v3 barcode whitelist (10xv3_whitelist.txt) using chemistry flag -s 10xv3. scKB calls kallisto for pseudoalignment of reads against the index and bustools for barcode correction against the whitelist and transcript counting. Barcode-correction statistics land in the stats files in the sample folder, and corrected counts are emitted as gene-by-cell .mtx matrices with row and column names. The workflow runs roughly 30 times faster than CellRanger 3.1 (~30 s on the 80 MB, 1-million-read toy_data subset).

## Related tools

- **scKB** (Wrapper bash script that invokes kallisto and bustools for accelerated alignment and transcript count matrix generation; takes -f (read folder), -i (index), -s (chemistry), -w (whitelist) arguments)
- **kallisto** (Performs pseudoalignment of reads against the cDNA_introns_10xv3.idx transcriptome index, called from within scKB)
- **bustools** (Performs 10xv3 barcode correction against the whitelist and transcript counting, called from within scKB)
- **COPILOT** (Downstream consumer of the spliced/unspliced matrices and stats files produced by the scKB whitelist-processing run) — github.com/Hsu-Che-Wei/COPILOT

## Examples

```
./scKB -f ./toy_data -i ./cDNA_introns_10xv3.idx -s 10xv3 -w ./10xv3_whitelist.txt
```

## Evaluation signals

- Both spliced and unspliced count matrices (.mtx) exist in the sample output folder and each carries row and column names
- The run produces the expected 8 output files: inspect., run_info., spliced and unspliced barcodes.txt, genes.txt, and the .mtx matrices
- Stats files (inspect., run_info.) are present and usable as input for downstream COPILOT quality filtering
- On the 80 MB toy_data subset (1 million reads), the run completes in roughly 30 s
- Barcodes in barcodes.txt conform to the 10xv3 whitelist after bustools correction

## Limitations

- Barcode handling is chemistry-specific: the 10xv3 whitelist and -s 10xv3 setting only apply to 10x Genomics v3 data; other chemistries need different whitelists
- Pseudoalignment-based counting requires a kallisto index containing cDNA plus intronic sequences; a genome lacking a suitable BSgenome object must be converted for related COPILOT steps
- scKB can fail to run properly during the 'align raw reads to genome via scKB' step, for which the protocol provides troubleshooting notes

## Evidence

- [abstract] scKB is a wrapper script of kallisto and bustools for accelerated alignment and transcript count matrix generation: "scKB is a wrapper script of kallisto and bustools for accelerated alignment and transcript count matrix generation"
- [methods] running scKB on the toy data: "they are about ~ 30 times faster than CellRanger 3.1 in processing time"
- [full_text] scKB invocation from source card: [evidence span withheld — anchor status: not_found]
- [intro] stats files as COPILOT input: "It is recommended to use the raw spliced and unpliced counts matrices produced by scKB pipeline as the input of COPILOT."
- [readme] sample.name directory convention: "which should be the same as the name of directory that contains spliced and unspliced matrices if you are following scKB pipeline to produce raw counts matrices."
