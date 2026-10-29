---
name: bowtie2-reference-indexing
description: Use when RNA-seq reads (e.g., GDC TCGA tumor BAMs) must be filtered against
  hg19 and the phiX contaminant to produce *_unmapped.fastq files feeding viRNAtrap
  or similar alignment-free viral detection.
when_to_use_negative:
- Input is already a directory of *_unmapped.fastq files meeting the viRNAtrap naming
  convention — re-filtering is unnecessary.
- The goal is standard human transcriptome quantification, where mapped (not unmapped)
  reads are the desired output; this skill deliberately discards hg19- and phiX-aligned
  reads.
- Running viRNAtrap's deep-learning prediction or contig assembly itself — mapping
  is explicitly the user's independent pre-processing step, not part of the included
  software.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Bowtie2
  role: Aligns paired-end reads from the TCGA BAM against the hg19 (1000 Genomes)
    index and then the phiX NC_001422 index, so reads unmapped to both references
    can be extracted.
- name: viRNAtrap
  role: Downstream consumer of the concatenated *_unmapped.fastq files; its input
    convention dictates the naming and per-sample concatenation of the filter output.
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:e47548e7f6df787675bc87cf5d508961c937088614d3ea2dde7559d9ca11d716
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Bowtie2
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bowtie2-reference-indexing

## Summary

Build Bowtie2 indexes for the hg19 (1000 Genomes) human reference and the phiX phage genome (NC_001422), then align paired-end TCGA RNA-seq reads against them in sequence so that read pairs unmapped to both references can be extracted as the FASTQ input required by the viRNAtrap viral-read framework.

## When to use

Use when RNA-seq reads (e.g., GDC TCGA tumor BAMs) must be filtered against hg19 and the phiX contaminant to produce *_unmapped.fastq files feeding viRNAtrap or similar alignment-free viral detection.

## When NOT to use

- Input is already a directory of *_unmapped.fastq files meeting the viRNAtrap naming convention — re-filtering is unnecessary.
- The goal is standard human transcriptome quantification, where mapped (not unmapped) reads are the desired output; this skill deliberately discards hg19- and phiX-aligned reads.
- Running viRNAtrap's deep-learning prediction or contig assembly itself — mapping is explicitly the user's independent pre-processing step, not part of the included software.

## Inputs

- GDC TCGA RNA-seq BAM file (paired-end reads) for a tumor sample
- hg19 (1000 Genomes) human genome reference FASTA
- phiX phage genome FASTA (accession NC_001422)

## Outputs

- Per-sample combined FASTQ of read pairs unmapped to both hg19 and phiX, named *_unmapped.fastq
- Bowtie2 index files for hg19 and phiX NC_001422

## How to apply

Obtain the GDC TCGA RNA-seq BAM for the sample and build Bowtie2 index files for the hg19 (1000 Genomes) human reference and the phiX phage genome, accession NC_001422. Align the paired-end reads from the BAM against the hg19 index with Bowtie2 in paired-end mode; this human-genome mapping step is performed independently by the user, not by viRNAtrap itself, and its purpose is to reduce viRNAtrap running time. Collect the reads left unmapped by the hg19 alignment and align them against the phiX NC_001422 index to remove this frequent laboratory contaminant. Extract the read pairs that remain unmapped to both references and concatenate the two paired-end files into one combined FASTQ per sample, because viRNAtrap treats each distinct input file as a distinct sample. Name each output so it ends in *_unmapped.fastq, matching the input-directory convention viRNAtrap requires.

## Related tools

- **Bowtie2** (Aligns paired-end reads from the TCGA BAM against the hg19 (1000 Genomes) index and then the phiX NC_001422 index, so reads unmapped to both references can be extracted.)
- **viRNAtrap** (Downstream consumer of the concatenated *_unmapped.fastq files; its input convention dictates the naming and per-sample concatenation of the filter output.) — https://github.com/AuslanderLab/virnatrap.git

## Evaluation signals

- Output files follow the required *_unmapped.fastq naming convention expected in the viRNAtrap input directory.
- Output files contain valid paired-end FASTQ records.
- Each sample yields exactly one combined FASTQ, since viRNAtrap treats each distinct input file as a distinct sample.
- Spotted checks confirm retained reads are unmapped to both hg19 and phiX NC_001422, not just one reference.
- A one-to-one correspondence holds between input files and downstream viRNAtrap output contig files.

## Limitations

- Only phiX is filtered as a laboratory contaminant; it is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures.
- Paired-end files for the same sample stored separately must be concatenated, or viRNAtrap will treat them as two distinct samples.
- The hg19 mapping must be performed independently by the user; viRNAtrap does not include this step, so index choice (hg19 1000 Genomes) directly affects what is deemed 'unmapped'.

## Evidence

- [intro] Human/phiX filtering rationale: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] Mapping is user's independent step: "map reads to the human genome; this is performed independently by the user."
- [readme] Concatenation requirement for paired files: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."
- [readme] Required input naming convention: "an input directory continaing one or more input FASTQ files of unmapped reads with file names ending in"
- [intro] Filtering reduces runtime: "The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap), to detect viral reads and assemble predicted viral contigs"
