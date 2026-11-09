---
name: samtools-faidx-sequence-retrieval
description: Use when you have genomic coordinates (e.g., FWHM of the LOESS-smoothed
  coverage peak of a differentially expressed uTAR) and need the underlying FASTA
  sequence for homology-based annotation.
when_to_use_negative:
- The uTARs of interest are not differentially expressed across cell clusters — annotation
  effort is only directed to biologically significant transcripts
- The input is already a sequence/feature table and no coordinate-to-sequence extraction
  is needed
- No indexed reference genome is available (samtools faidx requires a .fai-indexed
  FASTA)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SAMtools
  role: depth extracts uTAR coverage; faidx retrieves the FASTA sequence over FWHM
    peak coordinates
  repo: github.com/fw262/TAR-scRNA-seq
- name: BLAST
  role: BLASTn searches each peak FASTA against the NCBI nt database to assign homology-based
    labels
  repo: github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: workflow engine chaining depth, LOESS/FWHM peak calling, faidx extraction,
    and BLASTn steps
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:eb4187c562c861016732b5ddffc54b8de31e568d20506857a9370e412a7fbe57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - SAMtools
  - Snakemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# samtools-faidx-sequence-retrieval

## Summary

Retrieve nucleotide FASTA sequence over uTAR coverage-peak coordinates with samtools faidx so differentially expressed unannotated TARs can be labeled by nucleotide sequence homology (BLASTn against nt). This bridges groHMM-derived TAR coordinates to sequence-based functional annotation.

## When to use

Use when you have genomic coordinates (e.g., FWHM of the LOESS-smoothed coverage peak of a differentially expressed uTAR) and need the underlying FASTA sequence for homology-based annotation.

## When NOT to use

- The uTARs of interest are not differentially expressed across cell clusters — annotation effort is only directed to biologically significant transcripts
- The input is already a sequence/feature table and no coordinate-to-sequence extraction is needed
- No indexed reference genome is available (samtools faidx requires a .fai-indexed FASTA)

## Inputs

- Indexed reference genome FASTA (samtools-faidx compatible, with .fai index)
- FWHM peak coordinates per differentially expressed uTAR (chr:start-end BED-like list)
- Merged alignment files (BAM) for SAMtools depth coverage extraction

## Outputs

- Per-peak FASTA nucleotide sequences over uTAR FWHM coordinates
- utar_blast_annotations.csv — one labeled row per differentially expressed uTAR with top-hit annotation

## How to apply

First identify differentially expressed uTARs (Wilcoxon rank-sum test under dataset thresholds such as >=25–50% of cells expressing the uTAR and >=0.25–1 log-e fold change). Extract total per-uTAR coverage from merged alignments with SAMtools depth, smooth with LOESS (span = 0.25), and locate the full-width at half-maximum (FWHM) coordinates of the largest peak. Then run samtools faidx with the reference genome and the chr:start-end FWHM interval to pull the exact nucleotide sequence. Feed that FASTA to BLASTn against the NCBI nt database, retaining hits with e-value <= 0.01 and bit score >= 50, and label each uTAR by its highest bit-score hit (unknown/undetermined if no hit qualifies). The rationale is that single-cell expression analysis acts as a filter directing annotation efforts only to biologically significant, cluster-differential uTARs.

## Related tools

- **SAMtools** (depth extracts uTAR coverage; faidx retrieves the FASTA sequence over FWHM peak coordinates) — github.com/fw262/TAR-scRNA-seq
- **BLAST** (BLASTn searches each peak FASTA against the NCBI nt database to assign homology-based labels) — github.com/fw262/TAR-scRNA-seq
- **Snakemake** (workflow engine chaining depth, LOESS/FWHM peak calling, faidx extraction, and BLASTn steps) — github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools faidx genome.fa chr1:12345-67890 > peak_fw1.fa; blastn -db nt -query peak_fw1.fa -evalue 0.01
```

## Evaluation signals

- Output table contains exactly one labeled row per differentially expressed uTAR
- All retained BLAST hits satisfy e-value <= 0.01 and bit score >= 50
- Top-hit labels match homologs reported in the paper (e.g., GTH1, PRPF8, SNX29, ATF7IP2, CLEC14A, SOX5, RUNX1T1, H3F3C, BST2, DUSP1, NATD1)
- Extracted FASTA sequence length equals the FWHM interval length and matches the reference at those coordinates
- uTARs with no qualifying hits are explicitly labeled unknown/undetermined rather than dropped

## Limitations

- uTARs whose peak sequence has no nt hit passing e-value <= 0.01 and bit score >= 50 remain undetermined
- Depends on groHMM having successfully generated the HMM bed files; a groHMM/rtracklayer install failure yields missing chr*_HMM.bed and downstream steps cannot run
- Label quality depends on dataset-specific DE thresholds (25–50% expressing cells, 0.25–1 log-e fold change)

## Evidence

- [other] Extract the nucleotide FASTA sequence over the FWHM coordinates with samtools faidx.: [evidence span withheld — anchor status: not_found]
- [intro] uTARs annotated by homology: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [abstract] expression filter rationale: "This procedure uses single-cell expression analyses as a filter to direct annotation efforts to biologically significant transcripts"
- [readme] groHMM bed dependency: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
