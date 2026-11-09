---
name: phage-contaminant-removal
description: 'Use when preparing RNA-seq FASTQ input for viRNAtrap or similar viral-read
  detection: TCGA BAM files must be stripped of human (hg19) alignments and phiX phage
  contaminant reads before viral prediction.'
when_to_use_negative:
- Input FASTQ files already represent fully unmapped reads that still contain phiX-aligned
  reads would need this filter, but reads already reduced to *_unmapped.fastq after
  both hg19 and phiX filtering should not be re-filtered
- 'Reads from non-human organisms: viRNAtrap models are trained on human vs viral
  sequences, so this hg19/phiX removal pipeline is not appropriate for samples from
  other host species'
- Single-read (unpaired) data processed as separate files per sample would violate
  viRNAtrap's one-file-per-sample assumption if not merged appropriately
edam_operation: http://edamontology.org/operation_3695
tools:
- name: Bowtie2
  role: Aligns paired-end RNA-seq reads first against hg19 and then the remaining
    unmapped reads against the phiX NC_001422 index to remove the frequent laboratory
    contaminant
- name: viRNAtrap
  role: Downstream consumer of the merged *_unmapped.fastq files; defines the required
    input naming convention and one-combined-file-per-sample practice
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:fed058b9c3e5aacae4e521d6f951cd9725932a3fa52a0b6c8c93798aaf14b97b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Bowtie2
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phage-contaminant-removal

## Summary

Pre-processing step from the viRNAtrap TCGA workflow that removes phiX phage contamination: TCGA RNA-seq reads are first unmapped from hg19, then remaining read pairs are aligned against the phiX (NC_001422) reference with Bowtie2 and read pairs unmapped to both references are extracted as the *_unmapped.fastq input files.

## When to use

Use when preparing RNA-seq FASTQ input for viRNAtrap or similar viral-read detection: TCGA BAM files must be stripped of human (hg19) alignments and phiX phage contaminant reads before viral prediction.

## When NOT to use

- Input FASTQ files already represent fully unmapped reads that still contain phiX-aligned reads would need this filter, but reads already reduced to *_unmapped.fastq after both hg19 and phiX filtering should not be re-filtered
- Reads from non-human organisms: viRNAtrap models are trained on human vs viral sequences, so this hg19/phiX removal pipeline is not appropriate for samples from other host species
- Single-read (unpaired) data processed as separate files per sample would violate viRNAtrap's one-file-per-sample assumption if not merged appropriately

## Inputs

- GDC TCGA RNA-seq BAM file (paired-end)
- Bowtie2 index for the hg19 (1000 Genomes) human genome reference
- Bowtie2 index for the phiX phage genome (NC_001422)

## Outputs

- One combined paired-end FASTQ file per sample named *_unmapped.fastq, containing read pairs unmapped to both hg19 and phiX

## How to apply

Obtain the GDC TCGA RNA-seq BAM and align paired-end reads against a Bowtie2 index of the hg19 (1000 Genomes) human reference; the alignment step is performed independently by the user since viRNAtrap expects already-unmapped reads. Build a separate Bowtie2 index for the phiX phage genome (accession NC_001422), identified in the article as a frequent laboratory contaminant, and align the hg19-unmapped reads against it in paired-end mode. Extract read pairs that remain unmapped to both references and concatenate the R1/R2 FASTQ files into one combined file per sample, because viRNAtrap treats each distinct input file as if it comes from a distinct sample. Name each output following the required *_unmapped.fastq convention, since viRNAtrap only reads input files ending in that suffix from its input directory.

## Related tools

- **Bowtie2** (Aligns paired-end RNA-seq reads first against hg19 and then the remaining unmapped reads against the phiX NC_001422 index to remove the frequent laboratory contaminant)
- **viRNAtrap** (Downstream consumer of the merged *_unmapped.fastq files; defines the required input naming convention and one-combined-file-per-sample practice) — https://github.com/AuslanderLab/virnatrap.git

## Evaluation signals

- All output files follow the required *_unmapped.fastq naming convention expected by viRNAtrap
- Each sample yields exactly one combined FASTQ (R1/R2 concatenated), not separate paired files, since viRNAtrap treats each distinct input file as a distinct sample
- FASTQ records remain valid paired-end entries after extraction (correct 4-line record structure, matching mate identifiers)
- Reads aligned to phiX NC_001422 are absent from the final unmapped FASTQ, confirming contaminant removal
- Running viRNAtrap-predict on the filtered input produces a corresponding output contig file in the output directory (empty file if zero predicted viral contigs)

## Limitations

- Filtering only against hg19 and phiX does not model all possible laboratory contaminants, as the article notes it is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- viRNAtrap does not overwrite existing output files, so rerunning with the same inputs requires manually removing previous output files
- Human mapping is performed independently by the user; viRNAtrap itself does not perform the hg19 alignment

## Evidence

- [intro] Extract reads not aligned to the human genome (hg19) or phiX phage contaminant before applying viRNAtrap: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] viRNAtrap input naming and directory convention: "a user must provide an input directory continaing one or more input FASTQ files of unmapped reads with file names ending in *_unmapped.fastq"
- [readme] Merge paired-end files per sample before viRNAtrap input: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."
- [readme] Human mapping performed independently by the user: "Given one or more input files of RNAseq reads (could be paired or unpaired), map reads to the human genome; this is performed independently by the user."
