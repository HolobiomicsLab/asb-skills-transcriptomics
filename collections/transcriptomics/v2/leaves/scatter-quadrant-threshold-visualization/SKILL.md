---
name: scatter-quadrant-threshold-visualization
description: Use when two independent feature-ranking metrics (e.g., expression magnitude
  vs. variance-based loadings) must be compared to decide whether they identify the
  same significant features, and you need quadrant counts plus a correlation statistic
  as evidence.
when_to_use_negative:
- Input is already a ranked or filtered feature list without raw per-feature metric
  values — thresholds and quadrant counts cannot be recomputed
- The two metrics are not computed on the same feature universe (e.g., aTARs vs. uTARs),
  making quadrant overlap meaningless
- The goal is differential expression testing of TARs across clusters rather than
  comparing two continuous feature-ranking metrics
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: SAMtools
  role: Extract uniquely mapped reads (view -q 255) from scRNA-seq alignments, merged
    per experiment, before pseudo-bulk coverage calculation
- name: bedtools
  role: Compute uTAR pseudo-bulk coverage using bedtools coverage on the merged alignment
    files
- name: Seurat
  role: Perform PCA on the uTAR expression matrix and extract each uTAR's summed absolute
    loadings across the first 5 PCs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:7d2e5681e3ea3aba57761252fdc5000d96c763461a2d50dfd16f8ca37ac01bc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Seurat
  - bedtools
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scatter-quadrant-threshold-visualization

## Summary

Build a thresholded scatter plot comparing two ranking metrics for genomic features (here, uTAR pseudo-bulk read coverage vs. summed absolute Seurat PC loadings across the first 5 PCs), count features in each quadrant, and quantify agreement via Pearson correlation. Used to test whether two feature-significance criteria select overlapping feature sets.

## When to use

Use when two independent feature-ranking metrics (e.g., expression magnitude vs. variance-based loadings) must be compared to decide whether they identify the same significant features, and you need quadrant counts plus a correlation statistic as evidence.

## When NOT to use

- Input is already a ranked or filtered feature list without raw per-feature metric values — thresholds and quadrant counts cannot be recomputed
- The two metrics are not computed on the same feature universe (e.g., aTARs vs. uTARs), making quadrant overlap meaningless
- The goal is differential expression testing of TARs across clusters rather than comparing two continuous feature-ranking metrics

## Inputs

- BAM alignment files from scRNA-seq experiments (for uniquely mapped read extraction and pseudo-bulk coverage)
- uTAR BED interval files (transcriptionally active regions from groHMM)
- uTAR single-cell expression matrix (cells x uTARs) for Seurat PCA
- Threshold values for each axis (loading = 0.5, coverage = 1e+4)

## Outputs

- Scatter plot of uTAR PC loadings vs. pseudo-bulk read coverage with threshold lines (Fig. 2c style)
- Per-quadrant counts of uTARs (expected overlap: 329 of 3618)
- Pearson correlation statistics (r2 and p-value) between total expression level and PC loading

## How to apply

First compute each metric on the same feature set: merge uniquely mapped reads (SAMtools view -q 255) per experiment and compute uTAR pseudo-bulk coverage with bedtools coverage on merged alignments; separately, in Seurat, run PCA on the uTAR expression matrix and sum each uTAR's absolute PC loading values across the first 5 PCs. Build a scatter plot of PC loading (y) versus pseudo-bulk coverage (x), drawing a horizontal line at loading = 0.5 and a vertical line at coverage = 1e+4, then count uTARs in each quadrant. Compute the Pearson correlation between the two metrics and report r2 and p-value; in this study r2 = 4.0e-3 (p < 2.2e-16) with 329 of 3618 uTARs exceeding both thresholds, demonstrating that highly expressed uTARs are not necessarily those that best define cell types. Validate by checking that reproduced quadrant counts and correlation statistics match the reported Fig. 2c values.

## Related tools

- **SAMtools** (Extract uniquely mapped reads (view -q 255) from scRNA-seq alignments, merged per experiment, before pseudo-bulk coverage calculation)
- **bedtools** (Compute uTAR pseudo-bulk coverage using bedtools coverage on the merged alignment files)
- **Seurat** (Perform PCA on the uTAR expression matrix and extract each uTAR's summed absolute loadings across the first 5 PCs)

## Examples

```
samtools view -q 255 sample.bam > unique.bam; bedtools coverage -a utars.bed -b unique_merged.bam > utar_cov.txt  # then in R: pca <- Seurat::RunPCA(seurat_obj); loading <- rowSums(abs(pca@feature.loadings[1:5,]))
```

## Evaluation signals

- Quadrant count above both thresholds (loading > 0.5 and coverage > 1e+4) reproduces 329 shared uTARs out of 3618
- Pearson correlation between pseudo-bulk coverage and 5-PC summed absolute loading yields r2 = 4.0e-3 with p-value < 2.2e-16
- The near-zero r2 confirms no correlation, matching the reported finding that highly expressed uTARs are not those that best define cell types
- Scatter plot reproduces the layout of Fig. 2c with horizontal line at loading = 0.5 and vertical line at coverage = 1e+4
- Pseudo-bulk coverage values derive only from uniquely mapped reads (MAPQ 255 filter via SAMtools view)

## Limitations

- Very large feature counts (n = 3618) make even tiny correlations statistically significant (p < 2.2e-16), so p-value alone is misleading; effect size (r2) must be reported
- The comparison reflects one experiment's uTAR set; agreement between expression level and PC loadings may vary by organism, tissue, cell type, and developmental stage
- Requires the full snakemake/groHMM upstream pipeline to have completed; a groHMM failure yields missing chr*_HMM.bed files and empty downstream inputs

## Evidence

- [results] pseudo-bulk vs PC-loading comparison: [evidence span withheld — anchor status: not_found]
- [methods] workflow thresholds: [evidence span withheld — anchor status: not_found]
- [methods] expected overlap: [evidence span withheld — anchor status: not_found]
- [methods] read filtering: [evidence span withheld — anchor status: not_found]
- [methods] PCA loading definition: [evidence span withheld — anchor status: not_found]
