---
name: linear-runtime-scaling-analysis
description: Use when estimating or validating SkewC gene body coverage compute cost
  for a 10x Genomics barcoded BAM (e.g. pbmc_8k), or when planning how many cells
  can be processed in a given time budget.
when_to_use_negative:
- Your goal is gene body coverage quality assessment (skewness clustering) rather
  than runtime estimation — use the SkewC analysis skill (2_SkewC.sh, trim clustering)
  instead
- Your BAM files are already split by cell and you only need the quality metrics,
  not a performance benchmark across subsets
- You need precise scaling estimates for non-10x data whose BAM size differs greatly
  from pbmc_8k without re-benchmarking, since performance depends on the BAM file
  size
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Provides 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh (and geneBodyCoverage.pl)
    used to split the barcoded BAM and measure gene body coverage runtime
  repo: github.com/LSBDT/SkewC
- name: Perl
  role: geneBodyCoverage.pl is the Perl script that computes gene body coverage and
    creates the reference index; its per-GB runtime is the quantity being scaled
  repo: github.com/LSBDT/SkewC
- name: R
  role: Used to fit the linear regression of runtime on cell count and verify the
    ~3 h average for the 500-cell subset
  repo: github.com/LSBDT/SkewC
- name: docker
  role: Container runtime option for executing the SkewC workflow when admin authority
    is available
  repo: https://www.docker.com
- name: singularity
  role: Recommended container runtime; build a SIF from docker://moirai2/skewc:latest
    to run the benchmark
  repo: https://singularity.lbl.gov
- name: udocker
  role: Alternative user-space container runtime for Linux systems without admin privileges
  repo: https://github.com/indigo-dc/udocker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:469d34c6ad8593e127c7a93481935d4f8364d95ade6106839f357f5d1f64fbdd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - R
  - SkewC
  - docker
  - singularity
  - udocker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# linear-runtime-scaling-analysis

## Summary

Benchmark and verify the wall-clock runtime of the SkewC gene body coverage step (geneBodyCoverage.pl) across subsets of increasing cell count, using 10x Genomics pbmc_8k barcoded BAM files, and confirm the reported linear association between cell number and runtime (~3 h for a 500-cell subset on 10 cores).

## When to use

Use when estimating or validating SkewC gene body coverage compute cost for a 10x Genomics barcoded BAM (e.g. pbmc_8k), or when planning how many cells can be processed in a given time budget.

## When NOT to use

- Your goal is gene body coverage quality assessment (skewness clustering) rather than runtime estimation — use the SkewC analysis skill (2_SkewC.sh, trim clustering) instead
- Your BAM files are already split by cell and you only need the quality metrics, not a performance benchmark across subsets
- You need precise scaling estimates for non-10x data whose BAM size differs greatly from pbmc_8k without re-benchmarking, since performance depends on the BAM file size

## Inputs

- 10x Genomics barcoded BAM file (e.g. pbmc_8k)
- barcodes.tsv.gz from 10x Genomics outs/filtered_feature_bc_matrix/
- SkewC reference BED file (hg38_Gencode_V28.norRNAtRNA.bed for human, mm10_Gencode_VM18.norRNA.tRNA.bed for mouse)
- Per-cell split BAM files in subsets of increasing cell count (including a 500-cell subset)

## Outputs

- Runtime-versus-cell-count table (runtime_vs_cells.tsv) with wall-clock times per subset
- Linear regression fit of runtime on cell count (in R)
- Gene body coverage output files per cell under the coverage/ directory

## How to apply

Download the public pbmc_8k BAM file plus its barcodes.tsv.gz from outs/filtered_feature_bc_matrix/ and split it into per-cell BAM files with 0_split10XbyBarcode.sh. Assemble subsets of increasing cell count, including a 500-cell subset. Build the geneBodyCoverage.pl reference index once up front (indexing takes ~5-10 minutes; hg38's 13 MB bed file yields an ~82 MB index) and do not run geneBodyCoverage.pl in parallel while the index is being created. Then run 1_geneBodyCoverage.sh with species hg38 and reference hg38_Gencode_V28.norRNAtRNA.bed on every subset in parallel on 10 cores, recording wall-clock time for each run, and aggregate into a runtime-versus-cell-count table (e.g. runtime_vs_cells.tsv). Fit a linear regression of runtime on cell count in R; the expected result is a linear association with a 500-cell subset averaging ~3 h. Rationale: geneBodyCoverage.pl was written to imitate RSeQC's geneBody_coverage.py while reducing runtime about 10-fold, with computation time around 9 min per 1 GB of BAM (tested on E-MTAB-2600 mapped to mm10), and performance depends on both BAM file and reference genome size in GB.

## Related tools

- **SkewC** (Provides 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh (and geneBodyCoverage.pl) used to split the barcoded BAM and measure gene body coverage runtime) — github.com/LSBDT/SkewC
- **Perl** (geneBodyCoverage.pl is the Perl script that computes gene body coverage and creates the reference index; its per-GB runtime is the quantity being scaled) — github.com/LSBDT/SkewC
- **R** (Used to fit the linear regression of runtime on cell count and verify the ~3 h average for the 500-cell subset) — github.com/LSBDT/SkewC
- **docker** (Container runtime option for executing the SkewC workflow when admin authority is available) — https://www.docker.com
- **singularity** (Recommended container runtime; build a SIF from docker://moirai2/skewc:latest to run the benchmark) — https://singularity.lbl.gov
- **udocker** (Alternative user-space container runtime for Linux systems without admin privileges) — https://github.com/indigo-dc/udocker

## Examples

```
bash 1_geneBodyCoverage.sh hg38 input coverage
```

## Evaluation signals

- Plotting runtime versus cell count yields an approximately linear relationship (linear regression in R shows a significant positive slope with near-linear fit)
- The 500-cell subset averages ~3 h wall-clock on 10 cores, consistent with the reported performance evaluation
- Reference index is built once (5-10 minutes; hg38 index ~82 MB) and reused across iterations rather than rebuilt in parallel runs
- Runtimes are internally consistent with the ~9 min per 1 GB BAM figure reported for geneBodyCoverage.pl
- runtime_vs_cells.tsv contains a measured wall-clock entry for every subset, including the 500-cell subset

## Limitations

- Performance depends on both the BAM file size and the reference genome size in GB, so runtime estimates are dataset-specific and must be re-benchmarked for other datasets
- geneBodyCoverage.pl must not be run in parallel while it is creating the reference index at the first iteration; concurrent index creation corrupts or slows the benchmark
- SkewC setup requires git and at least one of docker, udocker, or singularity to be installed; the stringi/libicui18n.so.57 R package error terminates execution and is not fixable from the user's side

## Evidence

- [discussion] linear association between cell count and runtime: "Performance evaluation analysis of SkewC shows a linear association between the number of cells and SkewC runtime."
- [discussion] 500-cell runtime benchmark: "running SkewC on a BAM file containing 500 cells took on average 3 h to compute the gene body coverage."
- [readme] indexing and parallelization caveat: "geneBodyCoverage.pl will create an index file under a reference directory (default='reference') at the beginning of first iteration."
- [readme] per-GB computation time: "Computation time of geneBodyCoverage.pl is around 9min per 1GB."
- [methods] purpose of the perl script: "geneBodyCoverage.pl was written to reduce running time of geneBodyCoverage step by about 10 folds."
