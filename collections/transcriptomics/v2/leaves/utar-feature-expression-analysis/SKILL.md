---
name: utar-feature-expression-analysis
description: Use when scRNA-seq alignments must be quantified against annotation-independent
  genomic TARs, and when deciding whether pseudo-bulk uTAR expression or single-cell
  PC loading better flags biologically significant uTARs.
when_to_use_negative:
- The analysis targets annotated genes only (a well-annotated gene expression matrix)
  — this skill is specifically for regions outside existing annotation (uTARs)
- No aligned scRNA-seq data or groHMM TAR intervals exist (e.g. only a raw fastq with
  no from_fastqs/from_cellranger workflow run)
- A pseudo-bulk-only coverage ranking is sufficient — this skill shows coverage rank
  is uncorrelated (r2 = 4.0e-3) with cell-type-defining PC loadings
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: groHMM
  role: Hidden Markov model that divides the genome into transcribed/untranscribed
    states to call TARs from read coverage per bin
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: R package used to load groHMM BED outputs within the workflow
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: SAMtools
  role: Filters uniquely mapped reads (view -q 255) and merges alignments per experiment
    before coverage computation
- name: bedtools
  role: Computes uTAR pseudo-bulk read coverage on merged alignment files
- name: Seurat
  role: Performs PCA on the uTAR expression matrix and supplies PC loading values
    for significance scoring
- name: Snakemake
  role: Orchestrates the from_fastqs / from_cellranger pipelines that implement these
    steps
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:d8bfbb56c20835211e23494637c55316bf31a1cb36c9794d02e4d23e9d363b39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - Seurat
  - Snakemake
  - bedtools
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# utar-feature-expression-analysis

## Summary

A scRNA-seq analysis routine from the TAR-scRNA-seq workflow that builds a single-cell expression matrix over transcriptionally active regions (TARs), identifies unannotated TARs (uTARs) differentially expressed across cell clusters, and evaluates whether highly expressed uTARs (pseudo-bulk coverage) correspond to uTARs that define cell types (PC loadings).

## When to use

Use when scRNA-seq alignments must be quantified against annotation-independent genomic TARs, and when deciding whether pseudo-bulk uTAR expression or single-cell PC loading better flags biologically significant uTARs.

## When NOT to use

- The analysis targets annotated genes only (a well-annotated gene expression matrix) — this skill is specifically for regions outside existing annotation (uTARs)
- No aligned scRNA-seq data or groHMM TAR intervals exist (e.g. only a raw fastq with no from_fastqs/from_cellranger workflow run)
- A pseudo-bulk-only coverage ranking is sufficient — this skill shows coverage rank is uncorrelated (r2 = 4.0e-3) with cell-type-defining PC loadings

## Inputs

- scRNA-seq alignment files (BAM/SAM) from Cell Ranger or from_fastqs Snakemake workflow
- uTAR intervals in BED format produced by groHMM (chr*_HMM.bed)
- Gene annotation files for distinguishing aTARs from uTARs (e.g. reFlat-style reference used by calcHMMrefFlat)

## Outputs

- Single-cell uTAR expression matrix
- Per-uTAR pseudo-bulk read coverage counts from bedtools
- Seurat PC loading values per uTAR (sum of absolute loadings over first 5 PCs)
- Scatter plot with quadrant counts and Pearson correlation statistics (expected r2 = 4.0e-3, p < 2.2e-16)
- Set of differentially expressed uTARs prioritized for gene homology annotation

## How to apply

Identify TARs by running groHMM, which uses a hidden Markov model to divide the genome into transcribed and untranscribed states based on read coverage per bin, and retain uTARs (TARs outside the annotation). Build the uTAR expression matrix from uniquely mapped reads (SAMtools view -q 255), computing per-uTAR pseudo-bulk coverage with bedtools coverage on per-experiment merged alignments. In Seurat, run PCA on the uTAR expression matrix and score each uTAR by the sum of absolute PC loading values across the first 5 PCs. Compare the two significance criteria with a scatter plot of PC loading versus pseudo-bulk coverage, using thresholds loading > 0.5 and coverage > 1e+4 to count quadrant overlap, and compute the Pearson correlation. The paper's result — no correlation (r2 = 4.0e-3, p < 2.2e-16), with only 329 of 3618 uTARs above both thresholds — shows high expression does not imply cell-type-defining power, so differential expression across clusters is the better filter for annotation efforts.

## Related tools

- **groHMM** (Hidden Markov model that divides the genome into transcribed/untranscribed states to call TARs from read coverage per bin) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (R package used to load groHMM BED outputs within the workflow) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **SAMtools** (Filters uniquely mapped reads (view -q 255) and merges alignments per experiment before coverage computation)
- **bedtools** (Computes uTAR pseudo-bulk read coverage on merged alignment files)
- **Seurat** (Performs PCA on the uTAR expression matrix and supplies PC loading values for significance scoring)
- **Snakemake** (Orchestrates the from_fastqs / from_cellranger pipelines that implement these steps) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
samtools view -q 255 sample.bam > unique.bam; samtools merge merged.bam *.unique.bam; bedtools coverage -a utars.bed -b merged.bam > utar_pseudo_bulk_coverage.txt
```

## Evaluation signals

- Quadrant overlap on the loading-vs-coverage scatter plot (thresholds loading = 0.5, coverage = 1e+4) reproduces 329 of 3618 uTARs above both cutoffs as in Fig. 2c
- Pearson correlation between pseudo-bulk coverage and first-5-PC loading sums matches r2 = 4.0e-3 with p-value < 2.2e-16
- groHMM step completes and generates per-chromosome chr*_HMM.bed files (SingleCellHMM_Run_combined_bam_HMM_features.log shows no 'No such file or directory' for chr*_HMM.bed)
- Pseudo-bulk coverage computed only from uniquely mapped reads (SAMtools view -q 255), not multimappers

## Limitations

- groHMM must be manually installed along with rtracklayer; failure to load these packages in R causes the calcHMMrefFlat rule to fail with 'no lines available in input' because chr*_HMM.bed files are never produced
- Degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so quadrant counts and loading distributions vary across experiments
- Total pseudo-bulk expression level of a uTAR is not predictive of its cell-type-defining power (r2 = 4.0e-3), so coverage-based prioritization is a known pitfall

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [intro] uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [abstract] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs): "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"
- [readme] Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
