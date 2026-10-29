---
name: differential-expression-utar-filtering
description: Use when a single-cell TAR expression analysis has produced differentially
  expressed uTARs (e.g., from Wilcoxon rank-sum testing across clusters) that need
  functional or gene-level annotations via sequence homology.
when_to_use_negative:
- The regions of interest are already annotated TARs (aTARs) inside the existing genome
  annotation — this skill targets unannotated (uTAR) regions only
- The goal is only to identify differentially expressed TARs and not to assign sequence-homology
  annotations
- No indexed reference genome or nt BLAST database is available for the organism,
  so sequence extraction and homology search cannot be performed
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Extract per-uTAR coverage with samtools depth and pull peak FASTA sequences
    over FWHM coordinates with samtools faidx
- name: BLAST
  role: Run BLASTn on each uTAR peak sequence against the NCBI nt database to assign
    top-hit annotations (e-value <= 0.01, bit score >= 50)
- name: Snakemake
  role: Orchestrates the workflow steps (from_fastqs / from_cellranger) that surface
    the differentially expressed uTARs feeding this annotation step
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:ffe3c4c09bc90507a5e5eed3403f41c1c8706fd84833cf0555231ffedd3de765
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# differential-expression-utar-filtering

## Summary

Retrieves unannotated transcriptionally active regions (uTARs) that are differentially expressed across cell clusters and labels them by nucleotide sequence homology: coverage peaks are BLASTn-searched against the NCBI nt database so each uTAR receives a gene-level or functional annotation.

## When to use

Use when a single-cell TAR expression analysis has produced differentially expressed uTARs (e.g., from Wilcoxon rank-sum testing across clusters) that need functional or gene-level annotations via sequence homology.

## When NOT to use

- The regions of interest are already annotated TARs (aTARs) inside the existing genome annotation — this skill targets unannotated (uTAR) regions only
- The goal is only to identify differentially expressed TARs and not to assign sequence-homology annotations
- No indexed reference genome or nt BLAST database is available for the organism, so sequence extraction and homology search cannot be performed

## Inputs

- List of differentially expressed uTARs (from Wilcoxon rank-sum testing across cell clusters with dataset-specific % expressing and log-e fold-change thresholds)
- Merged alignment files (BAM) from which SAMtools depth coverage can be computed per uTAR
- Indexed reference genome FASTA (for samtools faidx extraction over FWHM coordinates)
- NCBI nucleotide collection (nt) BLAST database

## Outputs

- utar_blast_annotations.csv with one labeled row per differentially expressed uTAR
- Per-uTAR labels from the top bit-score BLASTn hit (e-value <= 0.01, bit score >= 50), or 'unknown/undetermined' when no hit qualifies

## How to apply

First, take the differentially expressed uTARs identified with the Wilcoxon rank-sum test under each dataset's thresholds (>=25-50% of cells in the cluster expressing the uTAR and >=0.25-1 log-e fold change over other cells, per dataset). Extract each uTAR's total coverage from the merged alignment files with SAMtools depth, smooth it with LOESS (span = 0.25), and locate the full-width at half-maximum (FWHM) coordinates of the largest peak along the uTAR. Extract the FASTA over those FWHM coordinates with samtools faidx and run BLASTn against the NCBI nucleotide collection (nt) database, retaining hits with maximum e-value 0.01 and minimum bit score 50. Label each uTAR by its retained hit with the highest bit score, marking uTARs without qualifying hits as unknown/undetermined, and write the results to utar_blast_annotations.csv. The rationale is that single-cell expression analysis acts as a filter to direct annotation efforts to biologically significant transcripts uncovered outside the genome annotation.

## Related tools

- **SAMtools** (Extract per-uTAR coverage with samtools depth and pull peak FASTA sequences over FWHM coordinates with samtools faidx)
- **BLAST** (Run BLASTn on each uTAR peak sequence against the NCBI nt database to assign top-hit annotations (e-value <= 0.01, bit score >= 50))
- **Snakemake** (Orchestrates the workflow steps (from_fastqs / from_cellranger) that surface the differentially expressed uTARs feeding this annotation step) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools faidx genome.fa chr1:1000-2000 > peak.fa && blastn -db nt -query peak.fa -evalue 0.01 -out utar_hits.tsv
```

## Evaluation signals

- utar_blast_annotations.csv contains exactly one labeled row per differentially expressed uTAR
- Every assigned label comes from a BLASTn hit passing e-value <= 0.01 and bit score >= 50; uTARs without qualifying hits are labeled unknown/undetermined
- Top-hit labels are consistent with homologs reported in the paper (e.g., GTH1, PRPF8, SNX29, ATF7IP2, CLEC14A, SOX5, RUNX1T1, H3F3C, BST2, DUSP1, NATD1, LOC115926763, LOC115920005)
- Peak coordinates correspond to the full-width at half-maximum of the largest LOESS-smoothed (span = 0.25) coverage peak along each uTAR
- Fasta sequences extracted with samtools faidx exactly span the FWHM coordinates of each peak

## Limitations

- uTARs with no BLASTn hit meeting e-value <= 0.01 and bit score >= 50 are left as unknown/undetermined — homology search cannot assign labels without a database match
- Differential-expression thresholds are dataset-specific (>=25-50% of cells expressing, >=0.25-1 log-e fold change), so filters must be re-tuned per dataset
- Labels are assigned only from the peak sequence at the largest coverage peak, not the entire uTAR, so homology outside the FWHM region is not captured

## Evidence

- [methods] BLASTn against nt database: [evidence span withheld — anchor status: not_found]
- [intro] Differential expression retrieval and homology annotation: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [abstract] Expression analysis as annotation filter: "This procedure uses single-cell expression analyses as a filter to direct annotation efforts to biologically significant transcripts"
- [readme] Workflow availability: "We have written Snakemake workflows to perform this analysis."
