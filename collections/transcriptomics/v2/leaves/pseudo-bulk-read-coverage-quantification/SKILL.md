---
name: pseudo-bulk-read-coverage-quantification
description: Use when you have scRNA-seq alignments and a set of TAR/uTAR intervals
  and need aggregate (pseudo-bulk) expression per region, e.g., to rank uTARs by total
  coverage and compare against cell-type-defining PC loadings.
when_to_use_negative:
- You need per-cell expression patterns or differential expression across clusters
  — use the single-cell TAR expression matrix and Seurat workflow instead; pseudo-bulk
  coverage discards cell-level structure.
- The alignment contains multi-mapped reads and you have not applied the uniquely-mapped
  filter (-q 255); coverage counts will be inflated.
- TAR/uTAR intervals do not yet exist — groHMM must first run to identify TARs from
  the alignments (see the calcHMMrefFlat/groHMM workflow).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SAMtools
  role: Filters alignments to uniquely mapped reads per experiment via 'samtools view
    -q 255' and merges them into experiment-level alignment files prior to coverage
    computation
- name: bedtools
  role: Computes uTAR pseudo-bulk coverage by intersecting the uTAR BED intervals
    with the merged alignment files (bedtools coverage)
- name: Seurat
  role: Performs PCA on the uTAR expression matrix to compute each uTAR's loading
    as the sum of absolute loadings across the first 5 PCs, for comparison against
    pseudo-bulk coverage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:b1d41f7a6306f59ac970debdc0fb056d3903768e96c32536309b71cae0a52f75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Seurat
  - bedtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pseudo-bulk-read-coverage-quantification

## Summary

Quantifies total expression level of transcriptionally active regions (TARs), especially unannotated TARs (uTARs), by merging uniquely mapped scRNA-seq reads per experiment and computing region-level read coverage. In this paper it is used to test whether highly expressed uTARs are the same ones that best define cell types via PCA loadings (they are not: r2 = 4.0e-3).

## When to use

Use when you have scRNA-seq alignments and a set of TAR/uTAR intervals and need aggregate (pseudo-bulk) expression per region, e.g., to rank uTARs by total coverage and compare against cell-type-defining PC loadings.

## When NOT to use

- You need per-cell expression patterns or differential expression across clusters — use the single-cell TAR expression matrix and Seurat workflow instead; pseudo-bulk coverage discards cell-level structure.
- The alignment contains multi-mapped reads and you have not applied the uniquely-mapped filter (-q 255); coverage counts will be inflated.
- TAR/uTAR intervals do not yet exist — groHMM must first run to identify TARs from the alignments (see the calcHMMrefFlat/groHMM workflow).

## Inputs

- scRNA-seq alignment files (SAM/BAM, e.g., cellranger count output)
- uTAR/TAR interval coordinates (BED, e.g., groHMM-derived TAR bed files)
- uTAR-by-cell single-cell expression matrix (for the parallel Seurat PCA on first 5 PCs)

## Outputs

- Per-uTAR pseudo-bulk read coverage values (one total coverage per region per experiment)
- Scatter plot of uTAR PC loading vs pseudo-bulk coverage with quadrant counts (thresholds: loading = 0.5, coverage = 1e+4)
- Pearson correlation statistics between coverage and loading (expected r2 = 4.0e-3, p < 2.2e-16)

## How to apply

For each experiment, merge uniquely mapped reads only using SAMtools view with the mapping-quality filter -q 255, producing a merged alignment file per experiment. Compute pseudo-bulk coverage of each uTAR by running bedtools coverage on the merged alignment files against the uTAR interval BED. Independently, in Seurat, run PCA on the uTAR expression matrix and score each uTAR by the sum of absolute loadings across the first 5 PCs. Cross-tabulate with thresholds at loading = 0.5 (horizontal) and coverage = 1e+4 reads (vertical) to count uTARs per quadrant; the expected overlap of high-coverage, high-loading uTARs is 329 of 3618. The rationale is that pseudo-bulk coverage measures raw expression magnitude, which the paper shows is uncorrelated (Pearson r2 = 4.0e-3, p < 2.2e-16) with the cell-type-defining information captured by PC loadings.

## Related tools

- **SAMtools** (Filters alignments to uniquely mapped reads per experiment via 'samtools view -q 255' and merges them into experiment-level alignment files prior to coverage computation)
- **bedtools** (Computes uTAR pseudo-bulk coverage by intersecting the uTAR BED intervals with the merged alignment files (bedtools coverage))
- **Seurat** (Performs PCA on the uTAR expression matrix to compute each uTAR's loading as the sum of absolute loadings across the first 5 PCs, for comparison against pseudo-bulk coverage)

## Examples

```
samtools view -b -q 255 sample.bam > unique.bam; bedtools coverage -a uTARs.bed -b unique.bam > uTAR_pseudobulk_coverage.bed
```

## Evaluation signals

- Quadrant counts must reproduce Fig. 2c: 329 of 3618 uTARs exceed both thresholds (loading > 0.5 and coverage > 1e+4).
- Pearson correlation between total pseudo-bulk expression and first-5-PC loading must give r2 = 4.0e-3 with p-value < 2.2e-16.
- Coverage should be computed only on reads passing the -q 255 uniquely-mapped filter; verify merged BAMs per experiment before bedtools coverage.
- Result should show essentially no overlap between highest-expressed uTARs and the uTARs that best define cell types — large coverage does not imply large loading.

## Limitations

- groHMM/rtracklayer must be manually installed and loadable in R; if not, the workflow fails with 'chr*_HMM.bed: No such file or directory' and no groHMM bed files (and thus no uTAR intervals) are produced for coverage.
- The workflow assumes alignments start from either raw FASTQs (from_fastqs) or already-processed cellranger count output (from_cellranger); other starting points are not supported.
- Pseudo-bulk coverage captures only total expression magnitude and, per the paper's finding, is essentially uncorrelated with cell-type-defining information (r2 = 4.0e-3), so it should not be used alone to prioritize biologically significant uTARs.

## Evidence

- [readme] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs): "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"
- [intro] uTARs differentially expressed across clusters are retrieved and annotated: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [intro] groHMM hidden Markov model on read coverage per bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Manual installation requirement for groHMM and rtracklayer: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
