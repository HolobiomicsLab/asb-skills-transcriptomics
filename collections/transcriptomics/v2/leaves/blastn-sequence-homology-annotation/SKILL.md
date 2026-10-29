---
name: blastn-sequence-homology-annotation
description: Use when you have differentially expressed uTARs (Wilcoxon rank-sum,
  e.g. >=25-50% of cluster cells expressing, >=0.25-1 log-e fold change) and need
  gene-level or functional annotations from nucleotide sequence homology.
when_to_use_negative:
- The TARs of interest are already annotated (aTARs) with existing gene symbols, so
  homology-based labeling adds no information
- You need transcript-level annotation rather than gene-level homology labels from
  genomic peak sequence
- No merged alignment files or reference genome are available to extract peak FASTA
  sequences from
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: BLAST
  role: Runs BLASTn of peak FASTA sequences against the nt database with e-value <=
    0.01 and bit score >= 50 filters to assign homology labels
- name: SAMtools
  role: Computes per-base coverage (samtools depth) over uTARs and extracts peak FASTA
    sequences (samtools faidx)
- name: Snakemake
  role: Wraps the uTAR retrieval, coverage extraction, peak calling, and BLASTn annotation
    steps into a reproducible workflow
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:fa241934c8ba4fa6f2d52157a8f8988e86a7f20a4255d5a1dcff23592020c0ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - BLAST
  - SAMtools
  - Snakemake
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# blastn-sequence-homology-annotation

## Summary

Labels differentially expressed uTARs by running BLASTn on the nucleotide sequence under each uTAR's dominant coverage peak against the NCBI nt database, assigning each uTAR the name of its top qualifying homology hit. This converts annotation-free transcribed regions into functionally interpretable gene-level labels.

## When to use

Use when you have differentially expressed uTARs (Wilcoxon rank-sum, e.g. >=25-50% of cluster cells expressing, >=0.25-1 log-e fold change) and need gene-level or functional annotations from nucleotide sequence homology.

## When NOT to use

- The TARs of interest are already annotated (aTARs) with existing gene symbols, so homology-based labeling adds no information
- You need transcript-level annotation rather than gene-level homology labels from genomic peak sequence
- No merged alignment files or reference genome are available to extract peak FASTA sequences from

## Inputs

- List of differentially expressed uTARs with genomic coordinates (from Wilcoxon rank-sum testing of aTAR/uTAR expression matrix)
- Merged alignment files (BAM) for the dataset
- Reference genome FASTA with samtools faidx index
- NCBI nucleotide collection (nt) BLAST database

## Outputs

- utar_blast_annotations.csv: one labeled row per differentially expressed uTAR, with the top hit name (or unknown/undetermined) as annotation

## How to apply

Start from the list of differentially expressed uTARs identified with the Wilcoxon rank-sum test under dataset-specific thresholds. Extract total per-base coverage of each uTAR from the merged alignment files with SAMtools depth, smooth it with LOESS (span = 0.25), and locate the full-width at half-maximum (FWHM) coordinates of the largest coverage peak along each uTAR, since the peak region carries the most reliable transcribed sequence. Extract the FASTA sequence over the FWHM coordinates with samtools faidx and run BLASTn against the NCBI nucleotide collection (nt) database, retaining hits with maximum e-value 0.01 and minimum bit score 50. Label each uTAR by the retained hit with the highest bit score, and label uTARs with no qualifying hit as unknown/undetermined. The rationale is that peak-localized sequences minimize noise from low-coverage flanks while the e-value/bit-score cutoffs ensure only confident homologs drive annotation.

## Related tools

- **BLAST** (Runs BLASTn of peak FASTA sequences against the nt database with e-value <= 0.01 and bit score >= 50 filters to assign homology labels)
- **SAMtools** (Computes per-base coverage (samtools depth) over uTARs and extracts peak FASTA sequences (samtools faidx))
- **Snakemake** (Wraps the uTAR retrieval, coverage extraction, peak calling, and BLASTn annotation steps into a reproducible workflow) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools faidx genome.fa chr1:5000-5600 | blastn -db nt -evalue 0.01 -outfmt '6 qseqid sseqid bitscore evalue stitle' | awk '$2>=50'
```

## Evaluation signals

- Output table utar_blast_annotations.csv contains exactly one labeled row per differentially expressed uTAR
- Every assigned label traces to a BLASTn hit passing e-value <= 0.01 and bit score >= 50; uTARs with no such hit are marked unknown/undetermined
- Top-hit labels are consistent with homologs reported in the paper (e.g., GTH1, PRPF8, SNX29, ATF7IP2, CLEC14A, SOX5, RUNX1T1, H3F3C, BST2, DUSP1)
- Each queried FASTA sequence spans the FWHM coordinates of the largest LOESS-smoothed (span = 0.25) coverage peak of its uTAR
- Annotation failures trace to groHMM/bed generation issues rather than BLAST filtering (e.g., missing chr*_HMM.bed files) when rows are unlabeled

## Limitations

- uTARs with no BLASTn hit passing e-value 0.01 / bit score 50 are labeled unknown/undetermined, so annotation coverage depends on nt database content
- Labels derive only from the single largest coverage peak (FWHM), so a uTAR with multiple distinct transcribed regions may be under-annotated
- groHMM HMM bed files must be generated successfully first; failure of the groHMM R script (e.g., empty input tables) leaves no TARs to annotate downstream

## Evidence

- [intro] uTARs annotated by homology workflow: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [readme] README homology annotation step: "generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs, and then annotates"
- [readme] README workflow reproducibility: "We have written Snakemake workflows to perform this analysis. Select your workflow by that files which you start with."
