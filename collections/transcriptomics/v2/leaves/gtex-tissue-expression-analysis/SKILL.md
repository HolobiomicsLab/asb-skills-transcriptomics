---
name: gtex-tissue-expression-analysis
description: Use when GTEx TPM values and MRSD scores are available for the same genes
  and you need to test whether relative expression alone predicts required sequencing
  depth for transcript coverage.
when_to_use_negative:
- When the goal is to select the best biosample for a disease gene panel — MRSD scoring
  alone (not TPM correlation) is the appropriate analysis
- When MRSD scores were not generated from GTEx-style Illumina 75 bp paired-end poly-A
  enriched RNA-seq, since pre-computed GTEx-derived MRSDs are most accurate only for
  similar workflows
- When assessing differential expression or allele-specific expression suitability
  — the article states the MRSD model cannot be directly extended to these applications
edam_operation: http://edamontology.org/operation_2949
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3170
tools:
- name: MRSD
  role: Provides the gene- and tissue-specific sequencing-depth metric that TPM is
    evaluated against as a proxy
- name: MRSDexplorer
  role: Generates the *.results.mrsd.txt score files loaded and merged with GTEx TPM
    values
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Used with pandas, scipy.stats, and matplotlib to merge data, compute correlations/overlap
    statistics, and render per-tissue scatter plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:2225cf78b01e14648f0cf33c86c36cb12b4ec7e41111438e4148c11cd68a0477
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_2949
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gtex-tissue-expression-analysis

## Summary

Uses GTEx TPM expression values merged with MRSD scores (from *.results.mrsd.txt files) to test whether relative gene expression is an adequate proxy for RNA-seq suitability, via correlation, stratification, and distribution-overlap analyses across four human biosamples.

## When to use

Use when GTEx TPM values and MRSD scores are available for the same genes and you need to test whether relative expression alone predicts required sequencing depth for transcript coverage.

## When NOT to use

- When the goal is to select the best biosample for a disease gene panel — MRSD scoring alone (not TPM correlation) is the appropriate analysis
- When MRSD scores were not generated from GTEx-style Illumina 75 bp paired-end poly-A enriched RNA-seq, since pre-computed GTEx-derived MRSDs are most accurate only for similar workflows
- When assessing differential expression or allele-specific expression suitability — the article states the MRSD model cannot be directly extended to these applications

## Inputs

- MRSD score files (*.results.mrsd.txt) with scores in millions of reads, '-' where median junction coverage is 0
- GTEx TPM expression values for whole blood, LCLs, skeletal muscle, and cultured fibroblasts
- Gene identifier mapping (GENCODE v19 gene symbol and/or Ensembl gene ID)

## Outputs

- Per-tissue Pearson and Spearman correlation statistics table (TPM vs MRSD)
- Per-tissue TPM overlap coefficients and Kolmogorov-Smirnov test results for low- (<100 M) vs high- (>=100 M) MRSD gene groups
- Figure 3-style per-tissue scatter plots of TPM versus MRSD with overlaid density distributions

## How to apply

Load per-tissue MRSD scores from the *.results.mrsd.txt files with Python/pandas, treating scores as millions of reads and excluding '-' entries (genes whose median splice junction coverage was 0 reads). Load GTEx TPM expression for whole blood, LCLs, skeletal muscle, and cultured fibroblasts and merge with MRSD scores on the gene identifier (GENCODE v19 gene symbol/Ensembl ID). Compute per-tissue Pearson and Spearman correlations between TPM and MRSD with scipy.stats, checking r-squared against the reported values (blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668). Stratify genes into low-MRSD (<100 M reads) and high-MRSD (>=100 M reads) groups per tissue, quantify TPM distribution overlap using overlap coefficients and Kolmogorov-Smirnov tests, and render Figure 3-style per-tissue TPM-vs-MRSD scatter plots with overlaid density distributions. Substantial overlap between the low- and high-MRSD TPM distributions indicates TPM is not an adequate proxy for complete transcript coverage.

## Related tools

- **MRSD** (Provides the gene- and tissue-specific sequencing-depth metric that TPM is evaluated against as a proxy)
- **MRSDexplorer** (Generates the *.results.mrsd.txt score files loaded and merged with GTEx TPM values) — github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Used with pandas, scipy.stats, and matplotlib to merge data, compute correlations/overlap statistics, and render per-tissue scatter plots)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Per-tissue Pearson r-squared of TPM vs MRSD falls in the reported range 0.613-0.714 (blood 0.661, LCLs 0.613, muscle 0.714, fibroblasts 0.668)
- TPM distributions of low-MRSD (<100 M) and high-MRSD (>=100 M) gene groups show substantial overlap, reproduced by overlap coefficients and non-significant/weakly separated KS tests
- Genes with '-' MRSD entries (median splice junction coverage of 0 reads) are excluded from the merged analysis table
- Rendered scatter plots match Figure 3-style layout with per-tissue panels and overlaid density distributions
- Merged gene counts match the '*.results.mrsd.txt' header line 'Number of genes successfully cross-referenced'

## Limitations

- TPM substantially overlaps between low- and high-MRSD gene groups, so relative expression alone is not an adequate proxy for complete transcript coverage
- Pre-computed GTEx-derived MRSD scores are most accurate for Illumina 75 bp paired-end poly-A enrichment workflows; other sequencing approaches require regenerating scores
- Genes with '-' MRSD entries lack usable scores because median splice junction coverage was 0 reads

## Evidence

- [results] TPM-MRSD inverse correlation: [evidence span withheld — anchor status: not_found]
- [readme] GTEx-derived MRSD scope: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
- [readme] Zero-coverage exclusion: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [abstract] MRSD metric definition: "We have derived a metric, minimum required sequencing depth (MRSD), that estimates the depth of sequencing required from RNA sequencing to achieve user-specified sequencing coverage of a gene"
