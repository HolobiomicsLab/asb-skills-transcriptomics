---
name: transcript-count-matrix-generation
description: Use when raw single-cell RNA-seq FASTQ reads must be pseudoaligned to
  a transcriptome and converted into spliced/unspliced UMI count matrices, especially
  when Cell Ranger runtime is prohibitive or RNA velocity/trajectory inference is
  planned.
when_to_use_negative:
- Input is already a gene-by-cell count matrix rather than raw FASTQ reads
- A full Cell Ranger-compatible secondary analysis output is required rather than
  raw count matrices
- Strict alignment (not pseudoalignment) to the genome is scientifically required
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKB
  role: Wrapper script that orchestrates kallisto pseudoalignment and bustools transcript
    count matrix generation from raw FASTQ reads
- name: kallisto
  role: Read aligner using pseudoalignment for accelerated alignment within the scKB
    pipeline
- name: bustools
  role: Generates transcript count matrices from kallisto pseudoalignment output
- name: Cell Ranger
  role: Reference baseline tool ('cellranger count --nosecondary') against which scKB
    runtime (~30-fold speedup) is benchmarked
- name: COPILOT
  role: Downstream consumer of the raw spliced/unspliced scKB matrices for non-arbitrary
    quality filtering
  repo: https://github.com/Hsu-Che-Wei/COPILOT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:21bba6ca4b563c205c18f36ed1ac65a221fff501cf616bc44219433570bb3eb1
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
  - COPILOT
  - Cell Ranger
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# transcript-count-matrix-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fast generation of spliced and unspliced gene-by-cell UMI count matrices from scRNA-seq FASTQ data using scKB, a wrapper script of kallisto and bustools, which runs about 30 times faster than Cell Ranger 3.1.

## When to use

Use when raw single-cell RNA-seq FASTQ reads must be pseudoaligned to a transcriptome and converted into spliced/unspliced UMI count matrices, especially when Cell Ranger runtime is prohibitive or RNA velocity/trajectory inference is planned.

## When NOT to use

- Input is already a gene-by-cell count matrix rather than raw FASTQ reads
- A full Cell Ranger-compatible secondary analysis output is required rather than raw count matrices
- Strict alignment (not pseudoalignment) to the genome is scientifically required

## Inputs

- Raw scRNA-seq FASTQ files (e.g. 10x Genomics chemistry v3 datasets such as the 386-million-read col0 sample)
- kallisto transcriptome index built from species transcriptome annotation (e.g. TAIR10 for Arabidopsis)

## Outputs

- Gene-by-cell matrix of spliced UMI counts with row and column names
- Gene-by-cell matrix of unspliced UMI counts with row and column names
- Raw spliced/unspliced matrices suitable as direct COPILOT input

## How to apply

Run the scKB pipeline (kallisto pseudoalignment plus bustools transcript count matrix generation) on raw FASTQ files using a kallisto transcriptome index built from the species annotation (e.g. TAIR10 for Arabidopsis), in non-iterative mode when only alignment and counting are needed. scKB produces raw spliced and unspliced counts matrices, which should be fed directly into COPILOT for non-arbitrary quality filtering rather than Cell Ranger outputs. Benchmark runtime against Cell Ranger 'cellranger count --nosecondary' on the same datasets; on the full col0 dataset (386 million reads, 10x Genomics v3), Cell Ranger took 640 min versus 25 min for scKB-COPILOT (scKB 22 min, COPILOT 3 min). The rationale is that kallisto uses pseudoalignment, which is dramatically faster than conventional alignment while retaining comparable counting capability.

## Related tools

- **scKB** (Wrapper script that orchestrates kallisto pseudoalignment and bustools transcript count matrix generation from raw FASTQ reads)
- **kallisto** (Read aligner using pseudoalignment for accelerated alignment within the scKB pipeline)
- **bustools** (Generates transcript count matrices from kallisto pseudoalignment output)
- **Cell Ranger** (Reference baseline tool ('cellranger count --nosecondary') against which scKB runtime (~30-fold speedup) is benchmarked)
- **COPILOT** (Downstream consumer of the raw spliced/unspliced scKB matrices for non-arbitrary quality filtering) — https://github.com/Hsu-Che-Wei/COPILOT

## Examples

```
Rscript scKB.R --sample.name col0 --fastq toy_col0_fastq/ --transcriptome.index TAIR10_kallisto_index/ --mode non-iterative
```

## Evaluation signals

- Runtime should be roughly 25 min for a 386-million-read dataset (scKB 22 min) versus 640 min for Cell Ranger 6.1.2 with --nosecondary, i.e. an approximately 30-fold speedup
- Output spliced and unspliced matrices must be gene-by-cell with both row and column names
- Matrices should be loadable directly by COPILOT (via sample.name directory of spliced/unspliced matrices) without reformatting
- Counting utility and summary metrics should be directly comparable to CellRanger 3.1 outputs
- Pipeline must run with iterative mode disabled when only alignment and counting are needed

## Limitations

- COPILOT supports cell type and developmental stage annotation only for the root of Arabidopsis thaliana
- The required genome may not be available as a BSgenome object and users may need to convert a customized genome
- Pseudoalignment is used instead of conventional read alignment, which may matter for applications requiring alignment-level detail
- scKB may fail to run properly during the 'align raw reads to genome via scKB' step, requiring troubleshooting

## Evidence
