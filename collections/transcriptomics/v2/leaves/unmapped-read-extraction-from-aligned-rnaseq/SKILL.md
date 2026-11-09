---
name: unmapped-read-extraction-from-aligned-rnaseq
description: Use when you have TCGA (or similar) paired-end RNA-seq FASTQs that have
  been aligned to hg19, and you need the unaligned fraction as FASTQ input for alignment-free
  viral read detection.
when_to_use_negative:
- When the analysis target is the human (host) transcriptome itself — this step deliberately
  discards all reads aligned to hg19, so it is unsuitable for host expression or variant
  work
- When reads from non-human/non-viral organisms are expected — the viRNAtrap 48 bp
  model's predictions for such sequences are undefined per the article's stated scope
- When you need to keep paired mates in separate R1/R2 files — viRNAtrap would treat
  each mate file as a distinct sample, splitting the sample's downstream viral contig
  calls
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0091
tools:
- name: viRNAtrap
  role: Downstream consumer of the *_unmapped.fastq files; scores unmapped reads with
    the pretrained 48 bp TensorFlow model and assembles predicted viral contigs
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:0a6f509daab96e5799db3a6b7a1a7c8800925358c27d35654b8f29b2c36c964e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - NumPy
  - viRNAtrap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# unmapped-read-extraction-from-aligned-rnaseq

## Summary

Prepares viRNAtrap input by extracting RNA-seq reads that fail to align to the human genome (hg19) and the phiX phage contaminant, writing them as *_unmapped.fastq files with paired-end mates concatenated per sample so each sample is a single input file.

## When to use

Use when you have TCGA (or similar) paired-end RNA-seq FASTQs that have been aligned to hg19, and you need the unaligned fraction as FASTQ input for alignment-free viral read detection.

## When NOT to use

- When the analysis target is the human (host) transcriptome itself — this step deliberately discards all reads aligned to hg19, so it is unsuitable for host expression or variant work
- When reads from non-human/non-viral organisms are expected — the viRNAtrap 48 bp model's predictions for such sequences are undefined per the article's stated scope
- When you need to keep paired mates in separate R1/R2 files — viRNAtrap would treat each mate file as a distinct sample, splitting the sample's downstream viral contig calls

## Inputs

- Paired-end RNA-seq FASTQ files from TCGA tumor samples (e.g., CESC, HNSC, LIHC) with alignments against hg19 available
- hg19 human genome reference (used for the user-performed mapping step)
- phiX phage reference sequence (contaminant filter)

## Outputs

- One *_unmapped.fastq file per sample, containing paired-end reads not aligned to hg19 or phiX, stored in a single input directory
- Per-sample unmapped-read directory ready for virnatrap-predict

## How to apply

First map the raw paired-end RNA-seq reads to the human genome (hg19) independently of viRNAtrap — the README explicitly states mapping 'is performed independently by the user' — and additionally discard reads matching the phiX phage, which the paper identified as a frequent laboratory contaminant. Write the reads not aligned to hg19 or phiX into FASTQ files named with the suffix *_unmapped.fastq in a single input directory, since viRNAtrap discovers its inputs by that filename suffix. Concatenate the two mate files of each paired-end sample into one combined *_unmapped.fastq file, because viRNAtrap treats each distinct input file as if it comes from a distinct sample — separate mate files would artificially split one sample into two. Keep one *_unmapped.fastq per sample in a directory with no extra FASTQ files, then point virnatrap-predict at that directory as --input. This pre-filtering step reduces the running time of viRNAtrap, which is the stated rationale for applying it only to unaligned RNA reads.

## Related tools

- **viRNAtrap** (Downstream consumer of the *_unmapped.fastq files; scores unmapped reads with the pretrained 48 bp TensorFlow model and assembles predicted viral contigs) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
bwa mem hg19.fa sample_R1.fq sample_R2.fq | samtools view -b -f 4 - > sample.unmapped.bam  # then concat mates into sample_unmapped.fastq; virnatrap-predict --input input_fastq/ --output output_contigs/
```

## Evaluation signals

- Every extracted file in the input directory has the *_unmapped.fastq suffix expected by virnatrap-predict
- No reads retained in the *_unmapped.fastq files align to hg19 or to phiX (re-alignment of a sample should return no high-confidence human hits)
- Paired-end samples appear as exactly one concatenated input file each, matching the 1:1 input-file-to-output-file correspondence virnatrap-predict enforces
- The number of extracted unmapped reads per sample is consistent with the alignment rate against hg19 (unmapped fraction plus phiX-filtered reads)
- Downstream validation: per-sample viral detections across TCGA CESC/HNSC/LIHC reach TPR > 95% against clinical annotations, confirming the unmapped-read input preserved the viral signal

## Limitations

- Mapping to the human genome must be performed independently by the user; viRNAtrap does not perform the hg19 alignment itself
- phiX filtering only removes that specific identified frequent contaminant — it is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- Concatenating mates is required: treating each paired-end file separately makes viRNAtrap call them distinct samples

## Evidence

- [intro] we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] Given one or more input files of RNAseq reads (could be paired or unpaired), map reads to the human genome; this is performed independently by the user.: "map reads to the human genome; this is performed independently by the user."
- [readme] the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample.: "concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample"
- [intro] The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap): "The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap)"
