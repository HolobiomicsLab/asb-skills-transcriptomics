---
name: utar-coverage-peak-fwhm-extraction
description: Use when a set of differentially expressed uTARs (e.g., Wilcoxon rank-sum
  hits) must be assigned functional or gene-level annotations via nucleotide sequence
  homology of their coverage peaks.
when_to_use_negative:
- The regions of interest are already annotated (aTARs) with gene symbols and do not
  require sequence-homology annotation.
- No aligned BAM files or reference genome FASTA are available from which per-uTAR
  coverage and peak sequences can be extracted.
- The goal is de novo transcript assembly or ORF prediction rather than homology-based
  labeling of differentially expressed uTAR peaks.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: SAMtools
  role: Computes per-base coverage depth over each differentially expressed uTAR (samtools
    depth) and extracts the FASTA sequence over the FWHM peak coordinates (samtools
    faidx)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: BLAST
  role: BLASTn search of each uTAR peak sequence against the NCBI nt database; hits
    filtered at e-value 0.01 and bit score 50, top bit-score hit used as the uTAR
    label
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Workflow engine that orchestrates the uTAR annotation steps within the TAR-scRNA-seq
    pipelines (from_fastqs / from_cellranger)
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:045bdffe0fbe4d2170e01fc91f966245e4f06e370885df1e32ed674c7ed16bff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - BLAST
  - SAMtools
  - Snakemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# utar-coverage-peak-fwhm-extraction

## Summary

Annotates differentially expressed uTARs by extracting the nucleotide sequence of their largest coverage peak (located via LOESS smoothing and FWHM coordinates) and labeling the uTAR by its top BLASTn hit against the NCBI nt database, providing sequence-homology functional labels for unannotated transcribed regions.

## When to use

Use when a set of differentially expressed uTARs (e.g., Wilcoxon rank-sum hits) must be assigned functional or gene-level annotations via nucleotide sequence homology of their coverage peaks.

## When NOT to use

- The regions of interest are already annotated (aTARs) with gene symbols and do not require sequence-homology annotation.
- No aligned BAM files or reference genome FASTA are available from which per-uTAR coverage and peak sequences can be extracted.
- The goal is de novo transcript assembly or ORF prediction rather than homology-based labeling of differentially expressed uTAR peaks.

## Inputs

- List of differentially expressed uTARs (cluster, coordinates, test statistics from Wilcoxon rank-sum test)
- Merged BAM alignment files (e.g., from Cell Ranger or from_fastqs workflow output)
- Indexed reference genome FASTA for samtools faidx
- NCBI nt BLAST database

## Outputs

- utar_blast_annotations.csv with one labeled row per differentially expressed uTAR
- Per-uTAR FWHM peak coordinates and peak FASTA sequences
- BLASTn hit table filtered to e-value <= 0.01 and bit score >= 50

## How to apply

Start from the list of differentially expressed uTARs identified by Wilcoxon rank-sum testing under the dataset's thresholds (>=25–50% of cells expressing the uTAR, >=0.25–1 log-e fold change). Extract per-base total coverage of each uTAR from merged alignment files with SAMtools depth, then smooth coverage with locally weighted (LOESS) smoothing using span = 0.25 and locate the full-width at half-maximum (FWHM) coordinates of the largest peak along the uTAR. Extract the FASTA sequence over the FWHM coordinates using samtools faidx and run BLASTn against the NCBI nucleotide collection (nt), retaining only hits with e-value <= 0.01 and bit score >= 50. Label each uTAR with its retained hit of highest bit score (unknown/undetermined if no hit qualifies); the rationale is that single-cell expression analysis acts as a filter to direct annotation efforts to biologically significant transcripts.

## Related tools

- **SAMtools** (Computes per-base coverage depth over each differentially expressed uTAR (samtools depth) and extracts the FASTA sequence over the FWHM peak coordinates (samtools faidx)) — https://github.com/fw262/TAR-scRNA-seq
- **BLAST** (BLASTn search of each uTAR peak sequence against the NCBI nt database; hits filtered at e-value 0.01 and bit score 50, top bit-score hit used as the uTAR label) — https://github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Workflow engine that orchestrates the uTAR annotation steps within the TAR-scRNA-seq pipelines (from_fastqs / from_cellranger)) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools depth merged.bam > utar.coverage.txt; samtools faidx genome.fa chr1:1500000-1503000 > peak_FWHM.fa; blastn -db nt -query peak_FWHM.fa -evalue 0.01
```

## Evaluation signals

- The output table contains exactly one labeled row per differentially expressed uTAR.
- All labels are assigned only from BLASTn hits passing e-value <= 0.01 and bit score >= 50; uTARs without qualifying hits are labeled unknown/undetermined.
- Top-hit labels match homologs reported in the paper (e.g., GTH1, PRPF8, SNX29, ATF7IP2, CLEC14A, SOX5, RUNX1T1, H3F3C, BST2, DUSP1, NATD1, LOC115926763, LOC115920005).
- FWHM coordinates correspond to the largest peak of the span = 0.25 LOESS-smoothed coverage track within each uTAR.
- Retrieved FASTA sequences fall within the FWHM boundaries of the reported uTAR coordinates.

## Limitations

- uTARs whose peak sequences have no qualifying BLASTn hit (e-value <= 0.01, bit score >= 50) remain unknown/undetermined, so coverage may be incomplete.
- Annotation quality depends on nucleotide sequence homology only; uTARs representing novel or poorly conserved transcripts may be unassignable.
- groHMM/R dependency failures (e.g., empty chr*_HMM.bed files) upstream can break the workflow before uTARs are available for annotation.
- Thresholds (percent of expressing cells, log-e fold change) are dataset-specific, so differentially expressed uTAR sets vary across datasets.

## Evidence

- [other] FWHM/LOESS/BLAST workflow: [evidence span withheld — anchor status: not_found]
- [other] BLASTn filtering: [evidence span withheld — anchor status: not_found]
- [intro] Homology-based uTAR annotation: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [abstract] Tool scope from abstract: "performs single-cell TAR expression analysis to identify biologically significant TARs, and then annotates TARs using gene homology analysis"
