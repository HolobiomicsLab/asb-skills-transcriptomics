---
name: hmm-based-transcribed-region-prediction
description: 'Use when scRNA-seq must be analyzed without a high-quality gene annotation:
  STAR- or CellRanger-aligned read alignments are available and genome-wide transcribed
  regions (TARs) must be predicted de novo from read coverage.'
when_to_use_negative:
- A high-quality genome annotation already exists and the analysis only needs standard
  gene-level quantification — the TAR-discovery stage is designed for annotation-independent
  analysis
- Input is a single-cell expression matrix rather than a genome alignment; groHMM
  requires per-bin read coverage along the genome, so re-running this skill on a matrix
  is meaningless
- Per-cell resolution is required at this stage — this step intentionally discards
  cell barcodes to build a combined genome-level track
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: groHMM
  role: HMM-based, strand-specific, annotation-free prediction of transcribed regions
    in 50 bp genome bins using gamma emission probabilities and Baum-Welch fitting
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: bedtools
  role: Merge predicted transcribed regions within 500 bp (-s -d 500) and compute
    per-region read counts (-s -counts -split)
- name: SAMtools
  role: Count uniquely mapped reads with `view -q 255` to set the retention threshold
    n = 1/10,000,000 of uniquely mapped reads
- name: rtracklayer
  role: Used in R alongside groHMM to load/write genomic feature files (bed/refFlat)
    in the TAR-discovery script
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Orchestrates the TAR-discovery workflow rules, including the calcHMMrefFlat
    step that runs groHMM
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:9f31a6c383faac6e3d6f628d7157a76aa3f8e826ce2a151991e165d8afaec21f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Snakemake
  - bedtools
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# hmm-based-transcribed-region-prediction

## Summary

Annotation-free identification of transcriptionally active regions (TARs) from scRNA-seq alignments using groHMM's two-state hidden Markov model (transcribed vs. untranscribed), followed by merging, coverage filtering, and refFlat output for downstream single-cell TAR expression analysis.

## When to use

Use when scRNA-seq must be analyzed without a high-quality gene annotation: STAR- or CellRanger-aligned read alignments are available and genome-wide transcribed regions (TARs) must be predicted de novo from read coverage.

## When NOT to use

- A high-quality genome annotation already exists and the analysis only needs standard gene-level quantification — the TAR-discovery stage is designed for annotation-independent analysis
- Input is a single-cell expression matrix rather than a genome alignment; groHMM requires per-bin read coverage along the genome, so re-running this skill on a matrix is meaningless
- Per-cell resolution is required at this stage — this step intentionally discards cell barcodes to build a combined genome-level track

## Inputs

- Combined (cell-barcode-merged) BAM of uniquely mapped scRNA-seq reads from STAR alignment
- Optional downsampled combined alignment (e.g. 5% of uniquely aligned reads) when memory-constrained

## Outputs

- tar_annotations.refflat — refFlat-format annotation of retained TAR features
- Per-chromosome groHMM bed files (chr*_HMM.bed) of predicted transcribed regions

## How to apply

Merge uniquely mapped reads from the STAR-aligned scRNA-seq data into a single combined BAM (cell barcodes are deliberately discarded); if memory-constrained, downsample the combined file (e.g. keep 5% of uniquely aligned reads as done for Tabula Muris). Run groHMM in a strand-specific, annotation-free manner to scan mapped read counts in 50 bp non-overlapping windows, modeling emission probabilities with a gamma distribution and fitting parameters with the Baum-Welch EM algorithm, predicting transcribed regions on both sense and antisense strands. Merge predicted transcribed regions within 500 bp of each other with `bedtools merge -s -d 500`, then compute per-region coverage with `bedtools coverage -s -counts -split`. Compute a retention threshold n equal to 1/10,000,000 of uniquely mapped reads (counted with `samtools view -q 255`) and keep only TARs with at least n reads, writing the retained features to refFlat format (tar_annotations.refflat).

## Related tools

- **groHMM** (HMM-based, strand-specific, annotation-free prediction of transcribed regions in 50 bp genome bins using gamma emission probabilities and Baum-Welch fitting) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **bedtools** (Merge predicted transcribed regions within 500 bp (-s -d 500) and compute per-region read counts (-s -counts -split))
- **SAMtools** (Count uniquely mapped reads with `view -q 255` to set the retention threshold n = 1/10,000,000 of uniquely mapped reads)
- **rtracklayer** (Used in R alongside groHMM to load/write genomic feature files (bed/refFlat) in the TAR-discovery script) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Orchestrates the TAR-discovery workflow rules, including the calcHMMrefFlat step that runs groHMM) — github.com/fw262/TAR-scRNA-seq

## Examples

```
bedtools merge -s -d 500 -i chr1_HMM.bed
```

## Evaluation signals

- tar_annotations.refflat exists and parses as a valid refFlat annotation
- Every retained TAR satisfies the coverage cutoff of at least n = 1/10,000,000 of uniquely mapped reads (SAMtools view -q 255 count)
- Predicted transcribed regions exist on both sense and antisense strands (strand-specific groHMM run)
- Each retained TAR is at least one merged region: neighboring predictions within 500 bp were collapsed via bedtools merge -s -d 500
- chr*_HMM.bed files exist before the merge step — their absence signals the groHMM R script did not finish

## Limitations

- groHMM R script can silently fail to produce chr*_HMM.bed files (e.g. 'no lines available in input' in calcHMMrefFlat), typically because groHMM and rtracklayer were not installed and loadable in R
- Memory constraints on large combined alignments may require downsampling (e.g. 5% of uniquely aligned reads for Tabula Muris), which changes the read-depth floor n
- Predicted TARs are genome-level features independent of cell barcodes, so this stage alone does not yield per-cell expression

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [methods] Run groHMM in a strand-specific, annotation-free manner to scan mapped read counts along the genome in 50 bp non-overlapping windows, modeling emission probabilities with a gamma distribution: [evidence span withheld — anchor status: not_found]
- [methods] Merge predicted transcribed regions within 500 bp of each other using bedtools merge with parameters -s -d 500.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the retention threshold n as 1/10,000,000 of uniquely mapped reads in the combined alignment file, counting those reads via SAMtools view with parameter -q 255.: [evidence span withheld — anchor status: not_found]
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R. This error indicates that the groHMM R script did not finish running: [evidence span withheld — anchor status: not_found]
