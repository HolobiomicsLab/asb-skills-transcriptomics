---
name: bam-alignment-preprocessing
description: Use when starting from cellranger/STAR-aligned scRNA-seq BAMs and you
  need a combined, per-strand alignment to feed annotation-free TAR discovery or to
  compute the coverage cutoff n.
when_to_use_negative:
- Input is raw FASTQ — use the from_fastqs workflow with STAR/cellranger alignment
  first instead of merging existing BAMs
- You need per-cell barcode-resolved expression (this stage deliberately discards
  cell barcodes to build a genome-wide coverage track)
- A high-quality genome annotation exists and the goal is standard per-gene quantification
  rather than annotation-free TAR discovery
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Counts uniquely mapped reads via `samtools view -q 255` on the combined BAM
    to define the retention threshold n
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: bedtools
  role: Merges predicted transcribed regions (merge -s -d 500) and computes per-region
    strand-aware coverage (-s -counts -split)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Consumes the preprocessed combined alignment to scan read coverage in 50 bp
    bins with a two-state HMM (gamma emissions, Baum-Welch EM)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: R package used alongside groHMM to write/read genomic interval files (e.g.
    refFlat/bed outputs of predicted TARs)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Orchestrates the from_cellranger workflow that runs this preprocessing and
    the downstream HMM rules
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:52e7f04d4b787fc127b0c8083fba1ac3bc77babcdfda2bd974dbc891a98d49ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - SAMtools
  - Snakemake
  - bedtools
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bam-alignment-preprocessing

## Summary

Preprocess STAR-aligned scRNA-seq alignments (BAM) into a single combined, strand-aware input suitable for annotation-free groHMM TAR discovery, including memory-saving downsampling and uniquely-mapped read counting that sets the retention threshold.

## When to use

Use when starting from cellranger/STAR-aligned scRNA-seq BAMs and you need a combined, per-strand alignment to feed annotation-free TAR discovery or to compute the coverage cutoff n.

## When NOT to use

- Input is raw FASTQ — use the from_fastqs workflow with STAR/cellranger alignment first instead of merging existing BAMs
- You need per-cell barcode-resolved expression (this stage deliberately discards cell barcodes to build a genome-wide coverage track)
- A high-quality genome annotation exists and the goal is standard per-gene quantification rather than annotation-free TAR discovery

## Inputs

- STAR/cellranger-aligned single-cell BAM files (from_cellranger workflow starting point)
- Reference genome for coordinate-sorted alignments

## Outputs

- Combined alignment file of uniquely mapped reads (cell barcodes discarded)
- Optionally downsampled combined BAM (e.g. 5% of uniquely aligned reads)
- Count of uniquely mapped reads (SAMtools view -q 255) defining the retention threshold n = reads / 10,000,000
- Strand-specific read coverage per predicted transcribed region (bedtools coverage -s -counts -split)

## How to apply

Merge uniquely mapped reads from the STAR-aligned scRNA-seq data into a single combined alignment file, disregarding individual cell barcodes since groHMM only needs genome-wide read coverage. If memory-constrained (as done for Tabula Muris), downsample the combined file, e.g. keeping 5% of uniquely aligned reads. Count uniquely mapped reads with SAMtools view -q 255, because the TAR retention threshold n is defined as 1/10,000,000 of these reads. Produce per-strand coverage statistics (bedtools coverage -s -counts -split) and merged transcribed regions (bedtools merge -s -d 500) downstream so the preprocessed BAM can drive strand-specific, annotation-free HMM scanning in 50 bp non-overlapping windows.

## Related tools

- **SAMtools** (Counts uniquely mapped reads via `samtools view -q 255` on the combined BAM to define the retention threshold n) — https://github.com/fw262/TAR-scRNA-seq
- **bedtools** (Merges predicted transcribed regions (merge -s -d 500) and computes per-region strand-aware coverage (-s -counts -split)) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Consumes the preprocessed combined alignment to scan read coverage in 50 bp bins with a two-state HMM (gamma emissions, Baum-Welch EM)) — https://github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (R package used alongside groHMM to write/read genomic interval files (e.g. refFlat/bed outputs of predicted TARs)) — https://github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Orchestrates the from_cellranger workflow that runs this preprocessing and the downstream HMM rules) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools view -q 255 combined.bam > combined.uniq.bam; samtools view -c -q 255 combined.uniq.bam
```

## Evaluation signals

- Combined alignment parses as valid BAM/SAM and contains only uniquely mapped reads (MAPQ 255 by SAMtools convention)
- If downsampled, retained read fraction matches the target (e.g. ~5% of uniquely aligned reads for Tabula Muris)
- Uniquely-mapped read count from `samtools view -c -q 255` is consistent and n = count/10,000,000 is reproducible
- Downstream bedtools coverage with -s -counts -split reports nonzero counts for retained regions, confirming strand-aware processing
- Every retained TAR after filtering satisfies coverage >= n, and tar_annotations.refflat parses as valid refFlat

## Limitations

- Merging all cells into one BAM discards cell barcodes, so this stage cannot support per-cell analyses — it only feeds genome-wide TAR discovery
- Memory constraints on large datasets force downsampling (e.g. 5% of reads for Tabula Muris), which reduces sensitivity for lowly transcribed regions
- groHMM R script failures (e.g. 'cannot read: chr*_HMM.bed: No such file or directory') often stem from groHMM/rtracklayer not being installed/loadable in R, aborting BED generation

## Evidence

- [methods] Merge uniquely mapped reads from the STAR-aligned scRNA-seq data into a single combined alignment file, discarding consideration of individual cell barcodes: [evidence span withheld — anchor status: not_found]
- [methods] downsample the combined file, e.g. keeping 5% of uniquely aligned reads as done for Tabula Muris, if memory-constrained: [evidence span withheld — anchor status: not_found]
- [methods] Compute the retention threshold n as 1/10,000,000 of uniquely mapped reads in the combined alignment file, counting those reads via SAMtools view with parameter -q 255: [evidence span withheld — anchor status: not_found]
- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] You will likely see the following error:

cannot read: chr*_HMM.bed: No such file or directory: "cannot read: chr*_HMM.bed: No such file or directory"
