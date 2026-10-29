---
name: single-cell-pca-dimensionality-reduction
description: Use when you have a single-cell expression matrix over unannotated transcriptionally
  active regions (uTARs) and need to determine which regions carry cell-type-defining
  signal via PC loadings.
when_to_use_negative:
- Input is already a processed Seurat object or gene-by-cell matrix from a standard
  genome-annotated pipeline rather than TAR/uTAR intervals
- The goal is marker-gene differential expression rather than assessing which uTARs
  define cell types via dimensional structure
- The analysis target is annotated genes (aTARs) where expression-level ranking alone
  is typically sufficient
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Performs PCA on the uTAR expression matrix; PC loadings over the first 5 PCs
    summarize cell-type-defining signal per uTAR
- name: SAMtools
  role: Filters uniquely mapped reads (view -q 255) and merges alignments per experiment
    to build pseudo-bulk uTAR coverage
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: bedtools
  role: Computes uTAR pseudo-bulk read coverage over merged alignment files using
    bedtools coverage
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Identifies the transcriptionally active regions (TARs/uTARs) that form the
    features of the expression matrix undergoing PCA
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:e83f18af2c6d596b6e494a52fb56f7400c91c2379c3aa643e8f9234aef72e7c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - SAMtools
  - Seurat
  - bedtools
  - groHMM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-pca-dimensionality-reduction

## Summary

Applies PCA in Seurat to a single-cell TAR/uTAR expression matrix and summarizes each uTAR by the sum of its absolute loadings across the first 5 PCs, to test which uTARs best define cell types independent of their total expression level.

## When to use

Use when you have a single-cell expression matrix over unannotated transcriptionally active regions (uTARs) and need to determine which regions carry cell-type-defining signal via PC loadings.

## When NOT to use

- Input is already a processed Seurat object or gene-by-cell matrix from a standard genome-annotated pipeline rather than TAR/uTAR intervals
- The goal is marker-gene differential expression rather than assessing which uTARs define cell types via dimensional structure
- The analysis target is annotated genes (aTARs) where expression-level ranking alone is typically sufficient

## Inputs

- Single-cell uTAR expression matrix (cells x uTARs) from TAR-scRNA-seq
- SAM/BAM alignments of scRNA-seq reads (uniquely mapped, MAPQ 255) per experiment
- uTAR genomic intervals (BED format) from groHMM transcriptionally active region calls

## Outputs

- Per-uTAR PC loading values (sum of absolute loadings over first 5 PCs)
- Scatter plot of uTAR PC loadings vs pseudo-bulk read coverage with quadrant counts (loading = 0.5, coverage = 1e+4)
- Pearson correlation statistics between pseudo-bulk coverage and PC loading (r2, p-value)

## How to apply

Build the uTAR expression matrix by merging uniquely mapped reads (SAMtools view with -q 255) per experiment and computing per-uTAR coverage with bedtools coverage on the merged alignments. In Seurat, perform PCA on the uTAR expression matrix and compute each uTAR's loading as the sum of absolute PC loading values across the first 5 PCs. Plot loadings against pseudo-bulk read coverage with a horizontal line at loading = 0.5 and a vertical line at coverage = 1e+4, counting uTARs per quadrant (expected overlap: 329/3618 uTARs above both thresholds). Finally, compute the Pearson correlation between total expression level and PC loading to demonstrate that highly expressed uTARs are not necessarily cell-type-defining (r2 = 4.0e-3, p < 2.2e-16).

## Related tools

- **Seurat** (Performs PCA on the uTAR expression matrix; PC loadings over the first 5 PCs summarize cell-type-defining signal per uTAR)
- **SAMtools** (Filters uniquely mapped reads (view -q 255) and merges alignments per experiment to build pseudo-bulk uTAR coverage) — https://github.com/fw262/TAR-scRNA-seq
- **bedtools** (Computes uTAR pseudo-bulk read coverage over merged alignment files using bedtools coverage) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Identifies the transcriptionally active regions (TARs/uTARs) that form the features of the expression matrix undergoing PCA) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools view -q 255 sample.bam > sample.uniq.bam && bedtools merge -i all.uniq.sam > merged.bam && bedtools coverage -a utars.bed -b merged.bam
```

## Evaluation signals

- Quadrant overlap count: 329 of 3618 uTARs exceed both thresholds (loading > 0.5 and coverage > 1e+4), matching Fig. 2c
- Pearson correlation between total expression level and PC loading reproduces r2 = 4.0e-3 with p-value < 2.2e-16
- All PCs are computed on the uTAR expression matrix in Seurat using only uniquely mapped reads (SAMtools view -q 255)
- Loading summary uses the sum of absolute PC loading values across exactly the first 5 PCs

## Limitations

- PC loading magnitude is uncorrelated with pseudo-bulk expression level (r2 = 4.0e-3), so high read coverage cannot substitute for loading-based filtering
- groHMM-based TAR calls require groHMM and rtracklayer R packages to be manually installed; failure yields missing chr*_HMM.bed files and downstream rule errors such as calcHMMrefFlat failing with 'no lines available in input'

## Evidence

- [methods] Merge uniquely mapped reads (SAMtools view with -q 255) from the scRNA-seq alignments per experiment and compute uTAR pseudo-bulk coverage using bedtools coverage on the merged alignment files.: [evidence span withheld — anchor status: not_found]
- [methods] In Seurat, perform PCA on the uTAR expression matrix and calculate each uTAR's PC loading as the sum of absolute PC loading values across the first 5 PCs.: [evidence span withheld — anchor status: not_found]
- [results] There is no correlation between the total expression level of uTARs (pseudo-bulk read coverage) and their loading values in the first 5 PCs (r2 = 4.0e-3, p-value < 2.2e-16).: [evidence span withheld — anchor status: not_found]
- [readme] Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
