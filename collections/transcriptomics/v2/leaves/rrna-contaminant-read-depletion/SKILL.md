---
name: rrna-contaminant-read-depletion
description: Use when raw Ribo-seq FastQ reads have been adapter-trimmed and length-filtered,
  and abundant rRNA/tRNA contamination must be depleted before HISAT2/Bowtie2 alignment
  and htseq-count CDS quantification.
when_to_use_negative:
- Input reads are already rRNA-depleted (e.g. delivered post-filtering) and only alignment/counting
  remains
- No contaminant reference FASTA for the organism of study can be built or obtained,
  since Bowtie2 needs a matched rRNA/tRNA reference
- The dataset is not ribosome-profiling data but ordinary RNA-seq, where Ribo-seq-specific
  RPF handling (25-35 nt selection, P-site offset tools) is out of scope
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bowtie 2
  role: Aligns trimmed reads against the rRNA/tRNA/viral contaminant reference to
    flag and remove contaminant reads
  repo: https://github.com/equipeGST/RiboDoc
- name: FastQC
  role: Performs quality control on reads after contaminant depletion to verify the
    depletion step
  repo: https://github.com/equipeGST/RiboDoc
- name: RiboDoc
  role: Docker/Singularity pipeline that orchestrates the contaminant removal step
    between Cutadapt trimming and HISAT2/Bowtie2 alignment
  repo: https://github.com/equipeGST/RiboDoc
- name: Silva
  role: Source database for generating the organism rRNA contaminant FASTA reference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:7ad36cbe5522859801b74c8a178556ae1768ce2dab24359e9269f10b04d1842a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - Cutadapt
  - DESeq2
  - FastQC
  - HISAT2
  - RiboDoc
  - Silva
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rRNA contaminant read depletion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes contaminating reads (rRNA, tRNA, viral RNA) from ribosome-profiling samples by aligning trimmed reads against a contaminant reference FASTA with Bowtie2 within the RiboDoc pipeline, so that only non-contaminant footprints proceed to genome/transcriptome alignment and CDS counting.

## When to use

Use when raw Ribo-seq FastQ reads have been adapter-trimmed and length-filtered, and abundant rRNA/tRNA contamination must be depleted before HISAT2/Bowtie2 alignment and htseq-count CDS quantification.

## When NOT to use

- Input reads are already rRNA-depleted (e.g. delivered post-filtering) and only alignment/counting remains
- No contaminant reference FASTA for the organism of study can be built or obtained, since Bowtie2 needs a matched rRNA/tRNA reference
- The dataset is not ribosome-profiling data but ordinary RNA-seq, where Ribo-seq-specific RPF handling (25-35 nt selection, P-site offset tools) is out of scope

## Inputs

- Adapter-trimmed, length-filtered Ribo-seq reads in gzipped FastQ format (*.fastq.gz, e.g. [CONDITION]_[NAME].[REPLICATE].fastq.gz)
- Contaminant reference FASTA (organism rRNA sequences, e.g. generated from Silva)

## Outputs

- Depleted (non-contaminant) reads passed to genome/transcriptome alignment
- FastQC QC report on the depleted reads
- Per-step read counts recorded in the RiboDoc Analysis_Report (e.g. Yeast_RiboSeq.Analysis_Report.25-35.txt)

## How to apply

Build a contaminant FASTA (organism rRNAs from the Silva database, e.g. for S. cerevisiae R64-1-1 alongside the Ensembl genome FASTA and GFF3) and place it in the project's database/ subfolder. Configure the RiboDoc container so that, after Cutadapt trimming and filtering, reads are aligned against the contaminant reference with Bowtie2 and aligned (contaminant) reads are discarded. Run a FastQC step on the depleted reads to confirm the depletion worked, then proceed to joint genome/transcriptome alignment. The Yeast_RiboSeq.Analysis_Report.25-35.txt file documents read numbers at each step, letting you quantify how much material was removed as contaminant.

## Related tools

- **Bowtie 2** (Aligns trimmed reads against the rRNA/tRNA/viral contaminant reference to flag and remove contaminant reads) — https://github.com/equipeGST/RiboDoc
- **FastQC** (Performs quality control on reads after contaminant depletion to verify the depletion step) — https://github.com/equipeGST/RiboDoc
- **RiboDoc** (Docker/Singularity pipeline that orchestrates the contaminant removal step between Cutadapt trimming and HISAT2/Bowtie2 alignment) — https://github.com/equipeGST/RiboDoc
- **Silva** (Source database for generating the organism rRNA contaminant FASTA reference)

## Examples

```
docker run --rm -v /path/to/project/:/data/ equipegst/ribodoc CPU_NUMBER MEMORY_AMOUNT
```

## Evaluation signals

- The RiboDoc Analysis_Report (e.g. Yeast_RiboSeq.Analysis_Report.25-35.txt) shows a drop in read numbers at the contaminant-removal step relative to the post-trimming count, quantifying depleted reads
- FastQC run on the depleted reads shows reduced overrepresented sequences consistent with rRNA/tRNA removal
- The project folder follows the required layout (fastq/ and database/ subfolders, lowercase names) with the contaminant FASTA present in database/, otherwise the step fails
- Downstream alignment rates improve: after depletion, HISAT2/Bowtie2 align a larger fraction of remaining reads to the genome/transcriptome
- Final outputs (count_matrix_by_transcript_IDs.csv and DESeq2 html report in RESULTS/DESeq2_CDS.25-35) are produced, implying the depletion step completed without error

## Limitations

- The example GEO GSE173856 yeast dataset aligns little material; low-input datasets leave few reads after contaminant removal, which is why RiboDoc advises TRiP rather than riboWaltz for such datasets
- Contaminant removal is only as good as the reference: the contaminant FASTA must be generated for the correct organism (e.g. S. cerevisiae rRNAs from Silva)
- Bowtie2-based removal requires multimapping/unmapped filtering upstream (Cutadapt read-length filtering, default 25-35 nt selection) to be meaningful, so it cannot rescue badly trimmed libraries

## Evidence
