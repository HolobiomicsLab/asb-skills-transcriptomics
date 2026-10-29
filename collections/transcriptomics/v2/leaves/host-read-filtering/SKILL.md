---
name: host-read-filtering
description: Use when RNA-seq reads (e.g., TCGA tumor BAMs) must be stripped of host
  and contaminant signal before viral read detection, i.e., when you need *_unmapped.fastq
  inputs for an alignment-free viral caller like viRNAtrap.
when_to_use_negative:
- Input is already a *_unmapped.fastq file produced by an upstream host-alignment
  step — do not re-filter, just pass it to viRNAtrap
- RNA-seq data from organisms other than human — the filtering pipeline and viRNAtrap
  models are built around human reads, and model predictions for other organisms are
  undefined
- Analysis goal requires mapped host reads (e.g., host expression quantification)
  — this skill deliberately discards everything aligning to hg19
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: Bowtie2
  role: Aligns paired-end RNA-seq reads against hg19 and then against phiX (NC_001422);
    reads left unmapped by both alignments are extracted as the viral-detection input
- name: viRNAtrap
  role: Downstream consumer of the *_unmapped.fastq files; its input requirements
    (file naming, one combined FASTQ per sample) dictate the concatenation and validation
    steps
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:e08b15a8dd7c3cb11cdb758b583c405debbd93b66f798fa5f02c06950527b16a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Bowtie2
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# host-read-filtering

## Summary

Extract unmapped RNA-seq reads by aligning paired-end reads from TCGA/GDC BAM files against the hg19 human reference and the phiX phage (NC_001422) contaminant reference using Bowtie2, producing combined *_unmapped.fastq files that serve as the input to viRNAtrap.

## When to use

Use when RNA-seq reads (e.g., TCGA tumor BAMs) must be stripped of host and contaminant signal before viral read detection, i.e., when you need *_unmapped.fastq inputs for an alignment-free viral caller like viRNAtrap.

## When NOT to use

- Input is already a *_unmapped.fastq file produced by an upstream host-alignment step — do not re-filter, just pass it to viRNAtrap
- RNA-seq data from organisms other than human — the filtering pipeline and viRNAtrap models are built around human reads, and model predictions for other organisms are undefined
- Analysis goal requires mapped host reads (e.g., host expression quantification) — this skill deliberately discards everything aligning to hg19

## Inputs

- GDC TCGA RNA-seq BAM file (paired-end reads) for a tumor sample
- Bowtie2 index for the hg19 (1000 Genomes) human genome reference
- Bowtie2 index for the phiX phage genome (NC_001422)

## Outputs

- One combined FASTQ per sample containing read pairs unmapped to both hg19 and phiX, named per the *_unmapped.fastq convention required by viRNAtrap

## How to apply

Build Bowtie2 index files for the hg19 (1000 Genomes) human reference and for the phiX phage genome (NC_001422), which the paper identified as a frequent laboratory contaminant. Align the paired-end reads extracted from the GDC TCGA RNA-seq BAM against the hg19 index with Bowtie2 in paired-end mode, then align the reads left unmapped by hg19 against the phiX index to remove phage contamination. Extract the read pairs unmapped to both references and concatenate the paired-end files into a single combined FASTQ per sample, because viRNAtrap treats each input file as a distinct sample and only reads input files whose names end in *_unmapped.fastq from the input directory. Mapping to the human genome is performed independently by the user, not by viRNAtrap itself; this pre-filtering also reduces the running time of viRNAtrap.

## Related tools

- **Bowtie2** (Aligns paired-end RNA-seq reads against hg19 and then against phiX (NC_001422); reads left unmapped by both alignments are extracted as the viral-detection input)
- **viRNAtrap** (Downstream consumer of the *_unmapped.fastq files; its input requirements (file naming, one combined FASTQ per sample) dictate the concatenation and validation steps) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
bowtie2 -p 8 -x hg19_index -1 sample_R1.fastq -2 sample_R2.fastq --un-conc sample_unmapped.fastq
```

## Evaluation signals

- Output files follow the required *_unmapped.fastq naming convention expected in viRNAtrap's input directory
- Output files contain valid paired-end FASTQ records (R1/R2 pairs retained together after filtering)
- Exactly one combined FASTQ per sample (paired files concatenated), since viRNAtrap treats each input file as a distinct sample
- Reads matching phiX (NC_001422), a frequent contaminant, are absent from the unmapped output
- Downstream viRNAtrap run accepts the files without errors and produces a FASTA of predicted viral contigs (possibly empty) for each input file

## Limitations

- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures — only hg19 and phiX are filtered here
- Mapping reads to the human genome is performed independently by the user and is not part of the viRNAtrap software itself
- Pre-filtering to unaligned reads is done partly to reduce viRNAtrap running time; any misalignment in the host/contaminant step directly degrades viral detection sensitivity

## Evidence

- [intro] Extract reads not aligned to hg19 or phiX before viRNAtrap: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [methods] Mapping to human genome is user-performed: "map reads to the human genome; this is performed independently by the user."
- [readme] Input FASTQ naming and concatenation requirements for viRNAtrap: "one or more input FASTQ files of unmapped reads with file names ending in *_unmapped.fastq, and a path to an output directory"
- [readme] Rationale for concatenating paired-end files: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample"
