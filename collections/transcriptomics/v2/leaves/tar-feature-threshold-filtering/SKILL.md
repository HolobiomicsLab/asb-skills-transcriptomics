---
name: tar-feature-threshold-filtering
description: Use when building a TAR annotation for scRNA-seq in the absence of a
  quality gene annotation and you must remove low-support groHMM-predicted regions
  by an alignment-derived read-count cutoff.
when_to_use_negative:
- A high-quality gene annotation already exists and standard gene-level quantification
  (e.g. cellranger count) is sufficient — this step is for annotation-independent
  TAR discovery
- The goal is per-cell expression quantification — this stage discards cell barcodes
  and individual cell information by design
- Input is already a filtered feature list or expression matrix rather than an alignment
  from which a read-depth threshold can be computed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: groHMM
  role: Two-state (transcribed/untranscribed) HMM that scans mapped read counts in
    50 bp windows to predict candidate transcribed regions on both strands before
    threshold filtering
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: bedtools
  role: Merges predicted regions within 500 bp on the same strand (-s -d 500) and
    computes per-region counts (-s -counts -split) for threshold application
- name: SAMtools
  role: Counts uniquely mapped reads (view -q 255) in the combined BAM to define the
    retention threshold n = 1/10,000,000 of the library
- name: rtracklayer
  role: R package used alongside groHMM in the HMM R script; required for the groHMM
    bed-file generation step feeding the filter
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Orchestrates the workflow rules (e.g. calcHMMrefFlat) that chain groHMM prediction,
    bedtools merging/coverage, and threshold filtering into refFlat output
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:a685ef785aa84390b27dc68c0c62477c889b423284d868bfbf49a2ceca869f49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Snakemake
  - bedtools
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tar-feature-threshold-filtering

## Summary

Filters annotation-free groHMM-predicted transcribed regions down to biologically supported TARs by applying a retention cutoff of n = 1/10,000,000 of uniquely mapped reads, then exports the retained features in refFlat format (tar_annotations.refflat) for downstream single-cell TAR expression analysis.

## When to use

Use when building a TAR annotation for scRNA-seq in the absence of a quality gene annotation and you must remove low-support groHMM-predicted regions by an alignment-derived read-count cutoff.

## When NOT to use

- A high-quality gene annotation already exists and standard gene-level quantification (e.g. cellranger count) is sufficient — this step is for annotation-independent TAR discovery
- The goal is per-cell expression quantification — this stage discards cell barcodes and individual cell information by design
- Input is already a filtered feature list or expression matrix rather than an alignment from which a read-depth threshold can be computed

## Inputs

- Combined alignment file (BAM) of uniquely mapped reads from STAR-aligned scRNA-seq data, per strand
- groHMM-predicted transcribed regions per chromosome (e.g. chr*_HMM.bed) on sense and antisense strands
- Reference genome chromosome sizes for coordinate handling

## Outputs

- tar_annotations.refflat — retained TAR features in refFlat annotation format
- Per-region read counts from bedtools coverage (-s -counts -split) used for threshold comparison

## How to apply

After merging uniquely mapped STAR-aligned reads into one combined BAM (cell barcodes are deliberately ignored), scan mapped read counts in 50 bp non-overlapping windows with groHMM's two-state (transcribed/untranscribed) hidden Markov model on both strands, using a gamma emission distribution fit by Baum-Welch EM. Merge predicted transcribed regions on the same strand within 500 bp via bedtools merge (-s -d 500) and compute per-region coverage with bedtools coverage (-s -counts -split). Compute the retention threshold n as 1/10,000,000 of uniquely mapped reads, counted with SAMtools view -q 255 on the combined alignment. Keep only transcribed regions with at least n reads and write the retained TAR features to refFlat (tar_annotations.refflat). The threshold scales automatically with library depth, so the cutoff adapts to the dataset being analyzed.

## Related tools

- **groHMM** (Two-state (transcribed/untranscribed) HMM that scans mapped read counts in 50 bp windows to predict candidate transcribed regions on both strands before threshold filtering) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **bedtools** (Merges predicted regions within 500 bp on the same strand (-s -d 500) and computes per-region counts (-s -counts -split) for threshold application)
- **SAMtools** (Counts uniquely mapped reads (view -q 255) in the combined BAM to define the retention threshold n = 1/10,000,000 of the library)
- **rtracklayer** (R package used alongside groHMM in the HMM R script; required for the groHMM bed-file generation step feeding the filter) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Orchestrates the workflow rules (e.g. calcHMMrefFlat) that chain groHMM prediction, bedtools merging/coverage, and threshold filtering into refFlat output) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
bedtools merge -i chr1_HMM.bed -s -d 500 | bedtools coverage -a - -b combined.bam -s -counts -split
```

## Evaluation signals

- tar_annotations.refflat exists and parses as a valid refFlat annotation
- Every retained TAR satisfies the coverage cutoff of at least n = 1/10,000,000 of uniquely mapped reads (counted via samtools view -q 255)
- Merged regions have been consolidated within 500 bp on the same strand, consistent with bedtools merge -s -d 500
- TARs are present on both sense and antisense strands, reflecting the strand-specific groHMM prediction
- Downstream SingleCellHMM rules can read the refFlat without the 'no lines available in input' error

## Limitations

- Memory-constrained runs may require downsampling the combined alignment (Tabula Muris kept 5% of uniquely aligned reads), which changes the absolute read counts feeding the threshold
- If the groHMM R script fails (e.g. groHMM or rtracklayer not installed/loadable in R), chr*_HMM.bed files are never produced and the filter step receives empty input
- Cell barcodes are discarded during merging, so the resulting TAR annotation cannot be used directly for per-cell quantification without a subsequent expression matrix step

## Evidence

- [methods] threshold definition: [evidence span withheld — anchor status: not_found]
- [methods] merge and coverage parameters: [evidence span withheld — anchor status: not_found]
- [intro] HMM prediction rationale: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] common failure mode: "cannot read: chr*_HMM.bed: No such file or directory"
- [abstract] output purpose: "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"
