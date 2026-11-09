---
name: genomic-feature-correlation-analysis
description: Use when you have per-region expression estimates (e.g., uTAR pseudo-bulk
  coverage) plus a PCA loading matrix from single-cell expression data and need to
  ask whether the most highly expressed features are also the ones that define cell
  types.
when_to_use_negative:
- You already have a curated gene annotation and want standard gene-level differential
  expression rather than annotation-independent TAR-level significance comparisons
- Your goal is to identify differentially expressed uTARs across cell clusters or
  annotate them by sequence homology (that is the upstream/downstream TAR-discovery
  step, not the correlation assessment)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Filters and merges uniquely mapped reads (view -q 255) per experiment to create
    merged alignment input for pseudo-bulk coverage
- name: bedtools
  role: Computes per-uTAR pseudo-bulk read coverage on merged alignment files
- name: Seurat
  role: Performs PCA on the uTAR expression matrix to derive summed absolute PC loadings
    across the first 5 PCs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:62d17dc35fa403013c90349a80613b2b494889bffc5f258950a95456f9e50b6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Seurat
  - bedtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genomic-feature-correlation-analysis

## Summary

Tests whether genomic features (e.g., uTARs) deemed significant by aggregate pseudo-bulk expression coverage are the same as those deemed significant by their cell-type-defining power (summed PC loadings), by computing Pearson correlation and threshold-based quadrant overlap.

## When to use

Use when you have per-region expression estimates (e.g., uTAR pseudo-bulk coverage) plus a PCA loading matrix from single-cell expression data and need to ask whether the most highly expressed features are also the ones that define cell types.

## When NOT to use

- You already have a curated gene annotation and want standard gene-level differential expression rather than annotation-independent TAR-level significance comparisons
- Your goal is to identify differentially expressed uTARs across cell clusters or annotate them by sequence homology (that is the upstream/downstream TAR-discovery step, not the correlation assessment)

## Inputs

- BAM alignment files of scRNA-seq reads mapped to TAR/uTAR coordinates (uniquely mapped reads, MAPQ 255)
- BED file of transcriptionally active region (uTAR) coordinates
- Single-cell uTAR expression matrix in Seurat (counts) suitable for PCA

## Outputs

- Per-uTAR pseudo-bulk read coverage values
- Per-uTAR summed absolute PC loadings across the first 5 PCs
- Quadrant counts from the loading-vs-coverage scatter plot (e.g., 329/3618 overlapping uTARs above loading = 0.5 and coverage = 1e+4)
- Pearson correlation statistic between expression level and PC loading (r2 and p-value)

## How to apply

First build pseudo-bulk coverage per feature: merge uniquely mapped reads (SAMtools view with -q 255) across the scRNA-seq alignments per experiment, then run bedtools coverage on the merged alignments against the feature BED (e.g., uTAR coordinates) to get total read coverage per uTAR. Second, in Seurat, perform PCA on the uTAR single-cell expression matrix and compute each feature's loading as the sum of absolute PC loading values across the first 5 PCs. Third, plot PC loadings versus pseudo-bulk coverage with thresholds (horizontal line at loading = 0.5, vertical line at coverage = 1e+4) and count uTARs in each quadrant; the reported overlap was 329 of 3618 uTARs above both thresholds (Fig. 2c). Finally, compute the Pearson correlation between total expression level and PC loading; the expected result is a weak relationship (r2 = 4.0e-3, p-value < 2.2e-16), demonstrating that highly expressed uTARs are not necessarily those that best define cell types. The rationale is that expression magnitude and cell-type information content are independent axes of significance, so filtering on coverage alone would miss cell-type-defining unannotated regions.

## Related tools

- **SAMtools** (Filters and merges uniquely mapped reads (view -q 255) per experiment to create merged alignment input for pseudo-bulk coverage)
- **bedtools** (Computes per-uTAR pseudo-bulk read coverage on merged alignment files)
- **Seurat** (Performs PCA on the uTAR expression matrix to derive summed absolute PC loadings across the first 5 PCs)

## Examples

```
samtools view -q 255 -b exp.bam > uniq.bam; bedtools coverage -a utars.bed -b uniq.bam > utar_pseudobulk_coverage.txt
```

## Evaluation signals

- Quadrant counts must match reported values: 329 of 3618 uTARs above both the loading = 0.5 and coverage = 1e+4 thresholds (Fig. 2c reproduction)
- Pearson correlation must reproduce r2 = 4.0e-3 with p-value < 2.2e-16
- Pseudo-bulk coverage must derive only from uniquely mapped reads (SAMtools -q 255 filter applied before bedtools coverage)
- PC loadings must be computed as the sum of absolute loading values across the first 5 PCs, not PC1 alone
- Scatter plot includes both threshold lines (loading = 0.5, coverage = 1e+4) so quadrant membership is auditable

## Limitations

- Weak correlation (r2 = 4.0e-3 despite p < 2.2e-16 with 3618 features) means expression level should not be used as a proxy for cell-type-defining power when prioritizing uTARs
- The degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so thresholds calibrated in one experiment may not transfer
- The groHMM/R dependencies can fail silently during HMM bed generation; check logs such as SingleCellHMM_Run_combined_bam_HMM_features.log if inputs are missing

## Evidence

- [abstract] performs single-cell TAR expression analysis to identify biologically significant TARs: "performs single-cell TAR expression analysis to identify biologically significant TARs"
- [intro] uTAR information content varies by organism/tissue/cell type: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [readme] Workflow selection depends on starting files: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)"
