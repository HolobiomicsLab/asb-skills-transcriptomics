---
name: strand-specific-genomic-coverage-computation
description: Use when you have a STAR-aligned scRNA-seq combined BAM and groHMM-predicted
  transcribed regions, and need per-region, strand-aware read counts to apply a retention
  cutoff for annotation-free TAR discovery.
when_to_use_negative:
- Input is already a gene-annotated expression matrix where annotation-free TAR discovery
  is unnecessary
- Coverage must be aggregated across strands (e.g. bulk strandedness-agnostic counting),
  since this method deliberately separates sense and antisense signals
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: groHMM
  role: Hidden Markov model that divides the genome into transcribed/untranscribed
    states from read coverage per bin, predicting TARs annotation-free on both strands
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: bedtools
  role: Merges predicted transcribed regions within 500 bp (merge -s -d 500) and computes
    strand-specific split-read coverage per region (coverage -s -counts -split)
- name: SAMtools
  role: Counts uniquely mapped reads (view -q 255) in the combined alignment to derive
    the retention threshold n
- name: rtracklayer
  role: R package used alongside groHMM to load and write the retained TAR annotation
    features (refFlat)
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Workflow engine orchestrating the alignment-to-TAR-annotation stages of the
    TAR-scRNA-seq pipeline
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:466c94d3204a428ce1da4dbfd22346b4aa1ccd246eae104a8ad22efd9db8344d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Snakemake
  - bedtools
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Strand-specific genomic coverage computation

## Summary

A method for computing strand-specific read coverage over predicted transcribed regions in scRNA-seq alignments, used to filter groHMM-predicted TARs by a retention threshold before writing annotation features to refFlat format.

## When to use

Use when you have a STAR-aligned scRNA-seq combined BAM and groHMM-predicted transcribed regions, and need per-region, strand-aware read counts to apply a retention cutoff for annotation-free TAR discovery.

## When NOT to use

- Input is already a gene-annotated expression matrix where annotation-free TAR discovery is unnecessary
- Coverage must be aggregated across strands (e.g. bulk strandedness-agnostic counting), since this method deliberately separates sense and antisense signals

## Inputs

- STAR-aligned scRNA-seq combined BAM alignment (uniquely mapped reads, cell barcodes disregarded)
- groHMM-predicted transcribed regions in BED format (sense and antisense strands)

## Outputs

- Per-region strand-specific read counts over merged TARs
- Retained TAR features written in refFlat format (tar_annotations.refflat)

## How to apply

First merge uniquely mapped reads from the STAR-aligned scRNA-seq data into a single combined alignment file (optionally downsampling, e.g. keeping 5% of uniquely aligned reads as done for Tabula Muris if memory-constrained). Run groHMM strand-specifically in an annotation-free manner, scanning mapped read counts in 50 bp non-overlapping windows with gamma-distributed emission probabilities fit by the Baum-Welch EM algorithm, predicting transcribed regions on both sense and antisense strands. Merge predicted transcribed regions within 500 bp of each other using bedtools merge with parameters -s -d 500. Then compute per-region read coverage with bedtools coverage using parameters -s -counts -split so that strand and spliced-read splits are respected. Compute the retention threshold n as 1/10,000,000 of uniquely mapped reads in the combined alignment, counting those reads with SAMtools view -q 255, and keep only transcribed regions with at least n reads.

## Related tools

- **groHMM** (Hidden Markov model that divides the genome into transcribed/untranscribed states from read coverage per bin, predicting TARs annotation-free on both strands) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **bedtools** (Merges predicted transcribed regions within 500 bp (merge -s -d 500) and computes strand-specific split-read coverage per region (coverage -s -counts -split))
- **SAMtools** (Counts uniquely mapped reads (view -q 255) in the combined alignment to derive the retention threshold n)
- **rtracklayer** (R package used alongside groHMM to load and write the retained TAR annotation features (refFlat)) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Workflow engine orchestrating the alignment-to-TAR-annotation stages of the TAR-scRNA-seq pipeline) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
bedtools coverage -s -counts -split -a merged_tars.bed -b combined.bam > tars_coverage.bed && samtools view -q 255 combined.bam | wc -l
```

## Evaluation signals

- Output tar_annotations.refflat exists and parses as a valid refFlat annotation
- Every retained TAR satisfies the coverage cutoff of at least n = 1/10,000,000 of uniquely mapped reads
- Uniquely mapped read count is reproducible via SAMtools view -q 255 on the combined alignment
- Merged TARs respect the 500 bp gap (-d 500) and strand separation (-s) parameters
- Per-region coverage counts computed with -split match expected spliced-read handling

## Limitations

- If groHMM fails to finish, downstream coverage computation breaks: rule calcHMMrefFlat errors with 'no lines available in input' because chr*_HMM.bed files are missing
- Memory-constrained runs may require downsampling the combined alignment (e.g. keeping 5% of uniquely aligned reads for Tabula Muris), which alters the coverage counts and the threshold n
- groHMM and rtracklayer must be manually installed and loadable in R or the HMM stage silently produces no bed files

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Readme groHMM failure mode: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
- [readme] Readme dependency install: [evidence span withheld — anchor status: not_found]
- [abstract] Readme TAR expression matrix output: "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs"
