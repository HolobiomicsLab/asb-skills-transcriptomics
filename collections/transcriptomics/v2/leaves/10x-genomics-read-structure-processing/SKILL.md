---
name: 10x-genomics-read-structure-processing
description: Use when raw 10x Genomics Chromium (e.g., 10xv3) scRNA-seq FASTQs must
  be converted into gene-by-cell spliced/unspliced count matrices quickly, especially
  for Arabidopsis with a TAIR10-based index.
when_to_use_negative:
- Input reads are not from 10x Genomics Chromium chemistry — the scKB-compatible layout
  requires barcode/UMI reads separated from cDNA reads in the 10xv3 arrangement
- A full splice-aware alignment or isoform-level quantification is required — scKB
  uses kallisto pseudoalignment to produce transcript count matrices, not base-level
  alignments
- The goal is quality filtering rather than raw matrix generation — use COPILOT on
  the scKB-produced raw spliced/unspliced matrices instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: scKB
  role: Wrapper script that drives the alignment and transcript count matrix generation
    from the 10x-structured FASTQs
- name: kallisto
  role: Performs pseudoalignment of the cDNA reads against the cDNA_introns_10xv3.idx
    (TAIR10 with introns) index
- name: bustools
  role: Performs barcode counting to produce the gene-by-cell spliced and unspliced
    count matrices
- name: COPILOT
  role: Downstream consumer of the raw spliced/unspliced matrices; the scKB output
    directory must match the copilot sample.name convention
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Cell Ranger
  role: Popular alignment baseline that scKB outpaces by ~30x in processing time on
    the same 10x data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:2f348e9247743b291d5430ff1775ba3ec510989cba3a954dea5c484ceca835ff
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# 10x-genomics-read-structure-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process raw 10x Genomics Chromium scRNA-seq reads (barcode/UMI reads separated from cDNA reads) through the scKB wrapper of kallisto and bustools to rapidly generate gene-by-cell spliced and unspliced UMI count matrices, as demonstrated on the 386-million-read Arabidopsis thaliana col0 dataset.

## When to use

Use when raw 10x Genomics Chromium (e.g., 10xv3) scRNA-seq FASTQs must be converted into gene-by-cell spliced/unspliced count matrices quickly, especially for Arabidopsis with a TAIR10-based index.

## When NOT to use

- Input reads are not from 10x Genomics Chromium chemistry — the scKB-compatible layout requires barcode/UMI reads separated from cDNA reads in the 10xv3 arrangement
- A full splice-aware alignment or isoform-level quantification is required — scKB uses kallisto pseudoalignment to produce transcript count matrices, not base-level alignments
- The goal is quality filtering rather than raw matrix generation — use COPILOT on the scKB-produced raw spliced/unspliced matrices instead

## Inputs

- SRA runs from a 10x Genomics Chromium scRNA-seq experiment (e.g., SRR12046119 and SRR12046120, GSM4626009, 386 million reads)
- FASTQ files with 10x read structure (barcode/UMI reads separated from cDNA reads)
- kallisto index built from a TAIR10-based Arabidopsis transcriptome with introns (cDNA_introns_10xv3.idx)

## Outputs

- Gene-by-cell spliced count matrix with gene row names and cell barcode column names
- Gene-by-cell unspliced count matrix with gene row names and cell barcode column names
- Raw count matrices formatted as COPILOT input (directory name matching the copilot sample.name argument)

## How to apply

Download the SRA runs (e.g., SRR12046119 and SRR12046120 for the col0 sample GSM4626009) and dump reads to FASTQ. Convert the FASTQs into the scKB-compatible layout, with barcode/UMI reads separated from cDNA reads, and place them in a directory whose name matches the copilot 'sample.name' convention. Run the scKB wrapper, which drives kallisto pseudoalignment of the cDNA reads against the cDNA_introns_10xv3.idx index (TAIR10-based Arabidopsis transcriptome with introns) and bustools barcode counting; scKB is a wrapper script of kallisto and bustools that runs ~30 times faster than CellRanger 3.1. Export the gene-by-cell spliced and unspliced matrices with proper row (gene) and column (cell barcode) names, since these raw spliced and unpliced matrices are the recommended COPILOT input.

## Related tools

- **scKB** (Wrapper script that drives the alignment and transcript count matrix generation from the 10x-structured FASTQs)
- **kallisto** (Performs pseudoalignment of the cDNA reads against the cDNA_introns_10xv3.idx (TAIR10 with introns) index)
- **bustools** (Performs barcode counting to produce the gene-by-cell spliced and unspliced count matrices)
- **COPILOT** (Downstream consumer of the raw spliced/unspliced matrices; the scKB output directory must match the copilot sample.name convention) — https://github.com/Hsu-Che-Wei/COPILOT
- **Cell Ranger** (Popular alignment baseline that scKB outpaces by ~30x in processing time on the same 10x data)

## Examples

```
devtools::install_github('Hsu-Che-Wei/COPILOT')
```

## Evaluation signals

- The scKB alignment and matrix generation step completes in roughly the reported ~22 minutes for the 386-million-read col0 dataset (~30x faster than CellRanger 3.1)
- Both spliced and unspliced matrices are produced as quality-filterable count matrices
- Matrices carry proper row (gene) and column (cell barcode) names required for downstream COPILOT input
- The output directory name matches the copilot 'sample.name' convention so COPILOT can locate the raw matrices
- Utility and summary statistics are directly comparable to what CellRanger 3.1 would produce for the same sample

## Limitations

- scKB can fail to run properly during the 'align raw reads to genome via scKB' step, requiring troubleshooting
- Kallisto-based pseudoalignment produces count matrices rather than full alignments, so isoform-resolved or base-level analyses are not supported
- COPILOT downstream quality control and annotation only supports the root of Arabidopsis thaliana for cell type and developmental stage annotation
- Customized genomes must be converted into a BSgenome object when the required genome is not already available

## Evidence

- [abstract] scKB wrapper drives kallisto and bustools: "scKB is a wrapper script of kallisto and bustools for accelerated alignment and transcript count matrix generation"
- [readme] Speed advantage over Cell Ranger: "Since scKB and COPILOT incorporates kallisto, which is a read aligner using peudoalignment, they are about ~ 30 times faster than CellRanger 3.1 in processing time."
- [readme] Raw scKB matrices as COPILOT input: "It is recommended to use the raw spliced and unpliced counts matrices produced by scKB pipeline as the input of COPILOT."
- [readme] sample.name directory convention: "which should be the same as the name of directory that contains spliced and unspliced matrices if you are following scKB pipeline"
- [methods] TAIR10 transcriptome annotation: "Name of transcriptome annotation file. (e.g. TAIR10 for _Arabidopsis_)"
