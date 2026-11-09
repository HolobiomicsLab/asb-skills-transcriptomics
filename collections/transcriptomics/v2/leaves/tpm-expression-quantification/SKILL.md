---
name: tpm-expression-quantification
description: Use when you have per-gene MRSD scores (*.results.mrsd.txt) and matching
  GTEx TPM values and need to test whether relative expression level adequately predicts
  required sequencing depth.
when_to_use_negative:
- Input RNA-seq data were not generated with the GTEx-like Illumina 75 bp paired-end
  poly-A enrichment workflow, since pre-computed MRSD scores are most accurate only
  for similar workflows
- The goal is to assess suitability for detecting allele-specific expression biases
  or differential gene expression, which the MRSD model cannot directly predict
- MRSD scores are unavailable for the gene because median splice junction coverage
  was 0 reads ('-' entries) — these must be excluded rather than treated as TPM-comparable
  values
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: MRSD
  role: Provides the gene- and tissue-specific minimum required sequencing depth scores
    that TPM values are tested against as a suitability proxy
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: MRSDexplorer
  role: Toolkit whose *.results.mrsd.txt outputs are parsed with pandas as the MRSD
    input for the TPM-vs-MRSD correlation analysis
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Used with pandas to load and merge MRSD and TPM tables and with scipy.stats
    to compute Pearson/Spearman correlations, overlap coefficients, and KS tests;
    matplotlib for rendering Figure 3-style plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:aa58af5da1ea8bec3472127bb3429bd4692272bb345b1e6dd5c7075fe2900afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tpm-expression-quantification

## Summary

Uses GTEx TPM expression values as a candidate proxy for RNA-seq suitability by testing whether relative gene expression predicts MRSD (minimum required sequencing depth) across four human biosamples, quantifying the strength and limits of that proxy via correlation and distribution-overlap statistics.

## When to use

Use when you have per-gene MRSD scores (*.results.mrsd.txt) and matching GTEx TPM values and need to test whether relative expression level adequately predicts required sequencing depth.

## When NOT to use

- Input RNA-seq data were not generated with the GTEx-like Illumina 75 bp paired-end poly-A enrichment workflow, since pre-computed MRSD scores are most accurate only for similar workflows
- The goal is to assess suitability for detecting allele-specific expression biases or differential gene expression, which the MRSD model cannot directly predict
- MRSD scores are unavailable for the gene because median splice junction coverage was 0 reads ('-' entries) — these must be excluded rather than treated as TPM-comparable values

## Inputs

- MRSDexplorer output files (*.results.mrsd.txt) with MRSD scores in millions of reads, including '-' entries for genes with median junction coverage of 0 reads
- GTEx TPM expression values for the same genes in whole blood, LCLs, skeletal muscle, and cultured fibroblasts
- Gene identifier key (GENCODE v19 gene symbol and/or Ensembl gene ID) for merging MRSD and TPM tables

## Outputs

- Per-tissue Pearson and Spearman correlation statistics (r-squared) between TPM and MRSD
- TPM distribution overlap coefficients and Kolmogorov-Smirnov test results for low-MRSD (<100 M) vs high-MRSD (>=100 M) gene groups
- Figure 3-style per-tissue scatter plots of TPM versus MRSD with overlaid density distributions
- Per-tissue correlation statistics table written to disk

## How to apply

Load per-tissue MRSD scores from *.results.mrsd.txt files with Python/pandas, noting scores are in millions of reads and excluding '-' entries (genes whose median splice junction coverage was 0 reads). Load GTEx TPM values for whole blood, LCLs, skeletal muscle, and cultured fibroblasts and merge with MRSD scores on gene identifier. Compute per-tissue Pearson and Spearman correlations between TPM and MRSD with scipy.stats, checking r-squared against the reported values (blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668). Stratify genes into low-MRSD (<100 M) and high-MRSD (>=100 M) groups, quantify TPM distribution overlap using overlap coefficients and Kolmogorov-Smirnov tests, and render per-tissue TPM-vs-MRSD scatter plots with overlaid density distributions (Figure 3 style) plus a correlation statistics table. The rationale is that TPM and MRSD are inversely correlated, but substantial TPM overlap between low- and high-MRSD groups shows relative expression alone is not an adequate proxy for complete transcript coverage.

## Related tools

- **MRSD** (Provides the gene- and tissue-specific minimum required sequencing depth scores that TPM values are tested against as a suitability proxy) — https://github.com/mcgm-mrsd/mrsd-explorer
- **MRSDexplorer** (Toolkit whose *.results.mrsd.txt outputs are parsed with pandas as the MRSD input for the TPM-vs-MRSD correlation analysis) — https://github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Used with pandas to load and merge MRSD and TPM tables and with scipy.stats to compute Pearson/Spearman correlations, overlap coefficients, and KS tests; matplotlib for rendering Figure 3-style plots)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr, spearmanr, ks_2samp; mrsd = pd.read_csv('gtex_whole_blood.results.mrsd.txt', sep='\t', skiprows=15); mrsd = mrsd[mrsd['MRSD(example)'] != '-']
```

## Evaluation signals

- Per-tissue TPM-MRSD r-squared values reproduce the reported figures: blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668
- '-' MRSD entries (median junction coverage of 0 reads) are excluded and not treated as numeric depths
- Low-MRSD (<100 M) and high-MRSD (>=100 M) TPM distributions show substantial overlap, confirming relative expression is not an adequate proxy for complete transcript coverage
- Merged table row counts are consistent between MRSD scores and GTEx TPM values after joining on gene identifier
- Per-tissue scatter plots with density overlays and the correlation statistics table are written to disk

## Limitations

- TPM substantially overlaps between low- and high-MRSD gene groups, so relative expression alone is not an adequate proxy for complete transcript coverage
- Pre-computed MRSD scores are most accurate only for GTEx-like Illumina 75 bp paired-end poly-A enrichment RNA-seq processed with the specific Cummings et al. bioinformatics pipeline
- The MRSD model does not explicitly account for eQTLs, sQTLs, or genomic background transcriptome changes (e.g., interferonopathies), which may alter TPM-MRSD relationships in patient samples

## Evidence

- [full_text] TPM and MRSD are inversely correlated (r2 = 0.613-0.714 across the four tissues): [evidence span withheld — anchor status: not_found]
- [readme] MRSD scores are in millions of reads with '-' entries for zero median junction coverage: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [readme] MRSD is a statistical framework predicting required RNA-seq depth for transcript coverage: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
- [abstract] MRSD applied across four human biosamples including whole blood and LCLs: "We applied MRSD across four human biosamples: whole blood, lymphoblastoid cell lines (LCLs), skeletal muscle, and cultured fibroblasts"
- [intro] MRSD scores guide required sequencing depth for specific splicing aberrations: "MRSD scores (see web resources for access) can be utilized to select the most appropriate biosample to detect specific splicing aberrations and to guide required depth of sequencing."
