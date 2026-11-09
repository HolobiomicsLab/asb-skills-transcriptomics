---
name: paired-end-fastq-merging
description: Use when preparing paired-end RNA-seq unmapped reads (e.g., TCGA BAM
  reads failing alignment to hg19 and phiX NC_001422) as viRNAtrap input, since viRNAtrap
  treats each input FASTQ file as a distinct sample.
when_to_use_negative:
- Reads are single-end (unpaired) — there are no R1/R2 files to merge; just name the
  FASTQ *_unmapped.fastq and supply it directly.
- Downstream tools other than viRNAtrap that require R1/R2 pairs in separate files
  — merging would destroy the pairing structure those tools expect.
- Input reads are already mapped viral reads or assembled contigs — this skill applies
  only to pre-viRNAtrap unmapped-read preparation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: viRNAtrap
  role: Downstream consumer of the merged *_unmapped.fastq files; its input convention
    (one file per sample, suffix *_unmapped.fastq) dictates the merging and naming
    of paired-end files
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: Bowtie2
  role: Aligner used to produce the unmapped read pairs (against hg19, then phiX NC_001422)
    that are subsequently merged into a single FASTQ per sample
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:e661a49de27981d8e979b503f8a33c387d3bafcf2eb344dfb6e1faaaf4ddb67f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Bowtie2
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# paired-end-fastq-merging

## Summary

Concatenates the two paired-end FASTQ files of a sample into a single combined FASTQ (named *_unmapped.fastq) of reads unmapped to hg19 and phiX, so that viRNAtrap treats each sample as one input rather than two.

## When to use

Use when preparing paired-end RNA-seq unmapped reads (e.g., TCGA BAM reads failing alignment to hg19 and phiX NC_001422) as viRNAtrap input, since viRNAtrap treats each input FASTQ file as a distinct sample.

## When NOT to use

- Reads are single-end (unpaired) — there are no R1/R2 files to merge; just name the FASTQ *_unmapped.fastq and supply it directly.
- Downstream tools other than viRNAtrap that require R1/R2 pairs in separate files — merging would destroy the pairing structure those tools expect.
- Input reads are already mapped viral reads or assembled contigs — this skill applies only to pre-viRNAtrap unmapped-read preparation.

## Inputs

- GDC TCGA RNA-seq BAM file (paired-end reads) for a tumor sample
- Bowtie2 index for hg19 (1000 Genomes human reference)
- Bowtie2 index for phiX phage genome NC_001422
- Paired-end FASTQ files of reads unmapped to both hg19 and phiX (R1 and R2 files per sample)

## Outputs

- One combined per-sample FASTQ named *_unmapped.fastq (paired R1+R2 reads concatenated) placed in the viRNAtrap input directory

## How to apply

After aligning paired-end reads from the GDC TCGA RNA-seq BAM against Bowtie2 indices for hg19 (1000 Genomes) and then against phiX (NC_001422) to strip this frequent laboratory contaminant, collect the read pairs unmapped to both references. Instead of keeping the R1 and R2 FASTQ files separate, concatenate them into one combined FASTQ per sample, because the viRNAtrap README advises merging paired files: each distinct input file is treated as coming from a distinct sample, so unmerged pairs would be analyzed as two samples. Name the merged file with the required *_unmapped.fastq suffix and place it in a single input directory, since viRNAtrap (via virnatrap-predict --input/--output) reads all files ending in *_unmapped.fastq from that directory and produces one output contig file per input file.

## Related tools

- **viRNAtrap** (Downstream consumer of the merged *_unmapped.fastq files; its input convention (one file per sample, suffix *_unmapped.fastq) dictates the merging and naming of paired-end files) — https://github.com/AuslanderLab/virnatrap.git
- **Bowtie2** (Aligner used to produce the unmapped read pairs (against hg19, then phiX NC_001422) that are subsequently merged into a single FASTQ per sample)

## Examples

```
bowtie2 -x hg19 -1 sample_R1.fastq -2 sample_R2.fastq --un-conc unmapped_hg19 | samtools view -b -f 12 - ; bowtie2 -x phiX_NC_001422 -1 unmapped_hg19_1.fastq -2 unmapped_hg19_2.fastq --un-conc sample_unmapped.fastq
```

## Evaluation signals

- Merged output filename ends with the required *_unmapped.fastq suffix so viRNAtrap will pick it up from the input directory.
- Exactly one combined FASTQ per sample exists in the input directory (not separate R1/R2 files), since viRNAtrap builds one output contig file per input file.
- Records are valid paired-end FASTQ entries and read counts in the merged file equal the sum of unmapped reads from the R1 and R2 files.
- Reads in the merged file are unmapped to both hg19 and phiX NC_001422 (i.e., phiX contaminant reads were removed by the second Bowtie2 pass).
- Running virnatrap-predict on the input directory yields a one-to-one correspondence between input FASTQs and output contig files in the output directory.

## Limitations

- viRNAtrap cannot be told that two files belong to the same sample — merging is mandatory; leaving pairs unmerged makes viRNAtrap treat each file as a distinct sample.
- It is not possible to model all laboratory contaminants, so filtering against hg19 and phiX alone may not remove every non-viral contaminant read.
- virnatrap-predict will not overwrite existing output files; if rerunning with the same input directory, previous output files must first be removed.

## Evidence

- [other] Extract unmapped reads via Bowtie2 against hg19 and phiX from a TCGA RNA-seq BAM: [evidence span withheld — anchor status: not_found]
- [other] concatenate paired-end unmapped reads per sample: [evidence span withheld — anchor status: not_found]
- [readme] viRNAtrap input naming convention: "a user must provide an input directory continaing one or more input FASTQ files of unmapped reads with file names ending in  *_unmapped.fastq"
- [readme] rationale for merging paired-end files: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."
- [intro] phiX contaminant removal before viRNAtrap: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
