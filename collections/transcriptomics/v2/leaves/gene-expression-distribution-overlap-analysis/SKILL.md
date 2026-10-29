---
name: gene-expression-distribution-overlap-analysis
description: Use when you have per-gene MRSD scores (*.results.mrsd.txt) plus TPM
  values for the same genes and want to test whether expression level alone predicts
  required sequencing depth or full-transcript coverage.
when_to_use_negative:
- The dataset was generated with long-read, single-cell, or spatially resolved RNA-seq,
  or any workflow other than bulk short-read poly-A enrichment, since MRSD scores
  were derived from GTEx Illumina 75 bp paired-end poly-A data and are most accurate
  only for similar workflows.
- The goal is to detect allele-specific expression biases or differential gene expression,
  which the MRSD model cannot directly be extended to predict.
- The gene of interest carries an MRSD score of '-', meaning median splice-junction
  coverage was 0 reads, so no usable MRSD value exists to correlate or stratify.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MRSD
  role: Statistical framework whose per-gene MRSD scores (from *.results.mrsd.txt)
    supply the RNA-seq suitability metric being compared against TPM
- name: MRSDexplorer
  role: Toolkit used to generate the per-gene MRSD scores consumed by this correlation/overlap
    analysis
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: pandas is used to load and merge MRSD/TPM tables, scipy.stats for Pearson/Spearman
    correlations and KS tests, and matplotlib for Figure 3-style scatter plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:140ae5f6a7694870c3fe8fb061f95e00a32c6257f4548219ed195658e8149567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-expression-distribution-overlap-analysis

## Summary

Tests whether relative gene expression (GTEx TPM) is an adequate proxy for RNA-seq suitability (MRSD scores) across biosamples by computing per-tissue Pearson/Spearman correlations and quantifying TPM distribution overlap between low-MRSD (<100 M reads) and high-MRSD (>=100 M reads) gene groups.

## When to use

Use when you have per-gene MRSD scores (*.results.mrsd.txt) plus TPM values for the same genes and want to test whether expression level alone predicts required sequencing depth or full-transcript coverage.

## When NOT to use

- The dataset was generated with long-read, single-cell, or spatially resolved RNA-seq, or any workflow other than bulk short-read poly-A enrichment, since MRSD scores were derived from GTEx Illumina 75 bp paired-end poly-A data and are most accurate only for similar workflows.
- The goal is to detect allele-specific expression biases or differential gene expression, which the MRSD model cannot directly be extended to predict.
- The gene of interest carries an MRSD score of '-', meaning median splice-junction coverage was 0 reads, so no usable MRSD value exists to correlate or stratify.

## Inputs

- Per-tissue MRSD scores from *.results.mrsd.txt files (millions of reads; '-' entries for genes with zero median junction coverage)
- GTEx TPM expression values for the same genes in whole blood, LCLs, skeletal muscle, and cultured fibroblasts
- Gene identifiers (GENCODE v19 gene symbol and/or Ensembl gene ID) for merging MRSD and TPM tables

## Outputs

- Per-tissue Pearson and Spearman correlation statistics table (TPM vs MRSD, r2 checked against 0.661/0.613/0.714/0.668)
- Overlap coefficients and Kolmogorov-Smirnov test statistics comparing TPM distributions of low-MRSD (<100 M) vs high-MRSD (>=100 M) gene groups
- Figure 3-style per-tissue scatter plots of TPM versus MRSD with overlaid density distributions

## How to apply

Load per-tissue MRSD scores from the *.results.mrsd.txt files with Python/pandas, in millions of reads, excluding '-' entries (genes whose median splice-junction coverage was 0 reads). Load GTEx TPM values for whole blood, LCLs, skeletal muscle, and cultured fibroblasts and merge with MRSD scores on gene identifier. Compute per-tissue Pearson and Spearman correlations between TPM and MRSD with scipy.stats, checking r2 against the reported values (blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668). Stratify genes into low-MRSD (<100 M) and high-MRSD (>=100 M) groups per tissue, then quantify TPM distribution overlap between the groups using overlap coefficients and Kolmogorov-Smirnov tests. Substantial overlap despite significant inverse correlation (r2 = 0.613-0.714) demonstrates that relative expression alone is not an adequate proxy for complete transcript coverage.

## Related tools

- **MRSD** (Statistical framework whose per-gene MRSD scores (from *.results.mrsd.txt) supply the RNA-seq suitability metric being compared against TPM)
- **MRSDexplorer** (Toolkit used to generate the per-gene MRSD scores consumed by this correlation/overlap analysis) — https://github.com/mcgm-mrsd/mrsd-explorer
- **Python** (pandas is used to load and merge MRSD/TPM tables, scipy.stats for Pearson/Spearman correlations and KS tests, and matplotlib for Figure 3-style scatter plots)

## Examples

```
from scipy import stats; r, p = stats.pearsonr(gene_table['TPM'], gene_table['MRSD']); rho, ps = stats.spearmanr(gene_table['TPM'], gene_table['MRSD'])
```

## Evaluation signals

- Per-tissue r2 values reproduce the published range: blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668 (within tolerance).
- All '-' MRSD entries are excluded before correlation, matching the definition that '-' means median splice-junction coverage was 0 reads.
- TPM distributions of low-MRSD (<100 M) and high-MRSD (>=100 M) gene groups show substantial overlap (high overlap coefficients) despite significant inverse Spearman/Pearson correlation.
- Merged table contains MRSD and TPM for genes in both the *.results.mrsd.txt files and the GTEx TPM matrix, keyed on gene identifier.
- Correlation table and Figure 3-style scatter/density plots are written to disk per tissue.

## Limitations

- TPM substantially overlaps between low- and high-MRSD gene groups, so relative expression level alone is not an adequate proxy for complete transcript coverage.
- Pre-computed MRSD scores are most accurate for bulk short-read poly-A enrichment workflows like GTEx's Illumina 75 bp paired-end protocol; other sequencing methodologies are not characterized.
- The current MRSD model does not explicitly account for eQTLs, sQTLs, or genomic background transcriptome changes (e.g., interferonopathies), which may skew TPM-MRSD relationships.
- Genes with '-' MRSD scores (zero median junction coverage) cannot be included in the correlation or stratification.

## Evidence

- [readme] MRSD scores are derived from RNA-seq data available through the Genotype-Tissue Expression project (GTEx: "These MRSD scores are derived from RNA-seq data available through the Genotype-Tissue Expression project (GTEx"
- [readme] '-' entries denote zero median junction coverage: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [methods] MRSD definition as sequencing-depth framework: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
- [discussion] MRSD leverages read-depth variation, unlike expression proxies: "MRSD scoring, conversely, leverages variation in sample read depth to provide quantitative predictions about optimal sequencing depths."
- [abstract] MRSD applied to the four biosamples used here: "We applied MRSD across four human biosamples: whole blood, lymphoblastoid cell lines (LCLs), skeletal muscle, and cultured fibroblasts"
