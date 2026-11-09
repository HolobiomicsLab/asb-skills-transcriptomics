---
name: rnaseq-read-alignment
description: Use when a human RNA-seq BAM (e.g., from TCGA/GDC) must be reduced to
  reads unmapped to hg19 and phiX so the resulting *_unmapped.fastq files can be fed
  to viRNAtrap for viral read detection.
when_to_use_negative:
- Input is already a directory of *_unmapped.fastq files — the hg19/phiX filtering
  is already done; proceed directly to viRNAtrap prediction.
- Reads are from organisms other than human — the hg19/phiX reference set and the
  pre-filtering rationale are specific to human tumor RNA-seq.
- The goal is full read-to-transcriptome quantification or expression profiling —
  this skill discards mapped reads rather than using alignments downstream.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Bowtie2
  role: Aligns paired-end RNA-seq reads sequentially against hg19 and then the phiX
    NC_001422 index; reads left unmapped by both runs are extracted as the viRNAtrap
    input.
- name: viRNAtrap
  role: Downstream consumer of the concatenated unmapped-read FASTQs; requires input
    files named *_unmapped.fastq and treats each file as a distinct sample.
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:008f117e91a64e3a13cfe1c5114fa2a8b85e995d7eef4b496d24f851afc332eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Bowtie2
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rnaseq-read-alignment

## Summary

Pre-process human RNA-seq BAMs (e.g., GDC TCGA) into unmapped-read FASTQ inputs for viRNAtrap by sequentially aligning reads against the hg19 human genome and the phiX phage contaminant with Bowtie2, then extracting read pairs unmapped to both references.

## When to use

Use when a human RNA-seq BAM (e.g., from TCGA/GDC) must be reduced to reads unmapped to hg19 and phiX so the resulting *_unmapped.fastq files can be fed to viRNAtrap for viral read detection.

## When NOT to use

- Input is already a directory of *_unmapped.fastq files — the hg19/phiX filtering is already done; proceed directly to viRNAtrap prediction.
- Reads are from organisms other than human — the hg19/phiX reference set and the pre-filtering rationale are specific to human tumor RNA-seq.
- The goal is full read-to-transcriptome quantification or expression profiling — this skill discards mapped reads rather than using alignments downstream.

## Inputs

- GDC TCGA RNA-seq BAM file (paired-end reads) per tumor sample
- hg19 (1000 Genomes) human genome reference for Bowtie2 index building
- phiX phage genome accession NC_001422 for Bowtie2 index building

## Outputs

- One combined *_unmapped.fastq file per sample containing paired-end reads unmapped to both hg19 and phiX
- Valid paired-end FASTQ records in a directory serving as the viRNAtrap input directory

## How to apply

Build Bowtie2 index files for the hg19 (1000 Genomes) human reference and for the phiX phage genome (accession NC_001422), which the paper identifies as a frequent laboratory contaminant. Align the paired-end reads from the GDC TCGA BAM against the hg19 index using Bowtie2 in paired-end mode, then take the reads left unmapped and align them against the phiX index to remove this contaminant. Extract the read pairs that remain unmapped to both references and concatenate the two paired-end files into one combined FASTQ per sample, because viRNAtrap treats each distinct input file as if it comes from a distinct sample and the user is advised to merge paired files. Name each output so it ends in *_unmapped.fastq, since viRNAtrap reads input files with names ending in *_unmapped.fastq from its input directory. Note that the article states mapping reads to the human genome is performed independently by the user — viRNAtrap itself does not perform this alignment step.

## Related tools

- **Bowtie2** (Aligns paired-end RNA-seq reads sequentially against hg19 and then the phiX NC_001422 index; reads left unmapped by both runs are extracted as the viRNAtrap input.)
- **viRNAtrap** (Downstream consumer of the concatenated unmapped-read FASTQs; requires input files named *_unmapped.fastq and treats each file as a distinct sample.) — https://github.com/AuslanderLab/virnatrap.git

## Evaluation signals

- Every output file in the viRNAtrap input directory ends with the required *_unmapped.fastq suffix.
- Output FASTQs contain valid paired-end records with one combined file per sample (no split R1/R2 pairs left in the directory).
- Re-aligning the extracted reads against hg19 and phiX with Bowtie2 yields an (near-)zero mapping rate, confirming both filters were applied.
- The concatenation preserved read-pair ordering/mates so the FASTQ remains a valid paired-end record set.
- The viRNAtrap run consumes the file without sample-mixing errors, i.e., each input file maps one-to-one to an output contig file.

## Limitations

- Read mapping to the human genome must be performed independently by the user; viRNAtrap does not include this step.
- Only phiX is filtered as a contaminant; it is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures.
- If paired-end files are not concatenated per sample, viRNAtrap will treat each file as coming from a distinct sample.
- virnatrap-predict will not overwrite existing output files, so reruns require removing previous outputs first.

## Evidence

- [intro] Extract reads not aligned to the human genome (hg19) or phiX phage contaminant before applying viRNAtrap: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] Human genome mapping is performed independently by the user: "map reads to the human genome; this is performed independently by the user."
- [readme] Concatenate paired-end files into one combined input file per sample: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."
- [readme] Input files must end in *_unmapped.fastq: "a user must provide an input directory continaing one or more input FASTQ files of unmapped reads with file names ending in"
