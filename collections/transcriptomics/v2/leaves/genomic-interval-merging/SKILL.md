---
name: genomic-interval-merging
description: Use when strand-specific predicted intervals (e.g., groHMM TAR predictions
  on BED files) must be consolidated into non-overlapping features within a distance
  threshold such as 500 bp before coverage filtering.
when_to_use_negative:
- A high-quality gene annotation already exists and the analysis goal is gene-level
  quantification rather than annotation-free TAR discovery
- Intervals must be merged regardless of strand (e.g., whole-genome bin consolidation),
  since the -s flag here explicitly keeps sense and antisense predictions separate
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: groHMM
  role: Produces the per-strand predicted transcribed intervals in 50 bp windows that
    are subsequently merged
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: bedtools
  role: Performs the strand-aware distance-based merge (bedtools merge -s -d 500)
    and downstream per-region coverage counting (bedtools coverage -s -counts -split)
- name: SAMtools
  role: Counts uniquely mapped reads with view -q 255 to set the retention threshold
    n = 1/10,000,000 of uniquely mapped reads
- name: rtracklayer
  role: R package used to load groHMM results and write retained TAR features to refFlat
    format
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Orchestrates the merge, coverage, and retention-filter rules in the TAR-scRNA-seq
    workflow (from_fastqs or from_cellranger)
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:a7254a826027eb290b531a7500f4874ca90f5c94cc9cc4daa1d8e7d7d9caed3f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# genomic-interval-merging

## Summary

Merges adjacent or overlapping predicted genomic intervals—here, strand-specific groHMM-predicted transcribed regions—into consolidated features while preserving strand identity, as a post-processing step in annotation-free TAR discovery from scRNA-seq alignments.

## When to use

Use when strand-specific predicted intervals (e.g., groHMM TAR predictions on BED files) must be consolidated into non-overlapping features within a distance threshold such as 500 bp before coverage filtering.

## When NOT to use

- A high-quality gene annotation already exists and the analysis goal is gene-level quantification rather than annotation-free TAR discovery
- Intervals must be merged regardless of strand (e.g., whole-genome bin consolidation), since the -s flag here explicitly keeps sense and antisense predictions separate

## Inputs

- Strand-specific groHMM-predicted transcribed-region BED files (e.g., chr*_HMM.bed) on both sense and antisense strands
- Combined uniquely mapped BAM alignment from STAR-aligned scRNA-seq data (optionally downsampled, e.g., 5% for Tabula Muris)

## Outputs

- Merged TAR interval set within 500 bp per strand
- Per-TAR read coverage counts (bedtools coverage -s -counts -split)
- Retained TAR features in refFlat format (tar_annotations.refflat)

## How to apply

After groHMM predicts transcribed regions in 50 bp non-overlapping windows on both sense and antisense strands of the STAR-aligned combined scRNA-seq alignment, merge predictions lying within 500 bp of each other using bedtools merge with parameters -s -d 500, where -s preserves strand so sense and antisense TARs are not collapsed. Then compute per-region read coverage with bedtools coverage using -s -counts -split, and apply a retention threshold n equal to 1/10,000,000 of uniquely mapped reads (counted via SAMtools view -q 255) in the combined alignment. Retain only regions with at least n reads and export the surviving TAR features in refFlat format (tar_annotations.refflat). The rationale is that groHMM fragments transcription into short bins, so distance-based, strand-aware merging reconstructs coherent transcriptionally active regions suitable for downstream single-cell TAR expression matrices.

## Related tools

- **groHMM** (Produces the per-strand predicted transcribed intervals in 50 bp windows that are subsequently merged) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **bedtools** (Performs the strand-aware distance-based merge (bedtools merge -s -d 500) and downstream per-region coverage counting (bedtools coverage -s -counts -split))
- **SAMtools** (Counts uniquely mapped reads with view -q 255 to set the retention threshold n = 1/10,000,000 of uniquely mapped reads)
- **rtracklayer** (R package used to load groHMM results and write retained TAR features to refFlat format) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Orchestrates the merge, coverage, and retention-filter rules in the TAR-scRNA-seq workflow (from_fastqs or from_cellranger)) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
bedtools merge -s -d 500 -i groHMM_predictions.bed > merged_tars.bed && bedtools coverage -s -counts -split -a merged_tars.bed -b combined.bam > merged_tars_counts.bed
```

## Evaluation signals

- Output file tar_annotations.refflat exists and parses as a valid refFlat annotation
- Every retained TAR satisfies the coverage cutoff of at least n = 1/10,000,000 uniquely mapped reads (SAMtools view -q 255 count)
- Merged intervals do not join features on opposite strands, confirming the -s flag was honored
- No two retained TARs on the same strand lie within 500 bp of each other, confirming the -d 500 merge parameter was applied
- If groHMM bed files (chr*_HMM.bed) are missing, the merge step cannot proceed—check SingleCellHMM_Run_combined_bam_HMM_features.log for 'cannot read: chr*_HMM.bed'

## Limitations

- Merge fails silently downstream if groHMM bed files were never generated, typically because groHMM/rtracklayer were not installed and loadable in R
- Memory constraints may require downsampling the combined uniquely mapped alignment (e.g., keeping 5% of reads as done for Tabula Muris), which lowers absolute coverage counts and interacts with the n-read retention threshold
- Strand-specific merging means antisense transcription remains separate; it is not consolidated with sense-strand TARs

## Evidence

- [methods] Merge predicted transcribed regions within 500 bp of each other using bedtools merge with parameters -s -d 500.: [evidence span withheld — anchor status: not_found]
- [methods] Keep only transcribed regions with at least n reads and write the retained TAR features to refFlat format (tar_annotations.refflat).: [evidence span withheld — anchor status: not_found]
- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R.: [evidence span withheld — anchor status: not_found]
