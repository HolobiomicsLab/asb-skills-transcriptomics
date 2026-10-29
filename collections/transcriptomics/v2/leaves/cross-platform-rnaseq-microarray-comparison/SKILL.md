---
name: cross-platform-rnaseq-microarray-comparison
description: Use when the same gene signatures must be evaluated in both an RNA-seq
  and a microarray PDAC dataset (e.g., paired ICGC PACA-AU primary/metastasis/cell-line
  samples, or Moffitt 2015 tissue-compartment datasets) to confirm a signature trend
  is platform-independent.
when_to_use_negative:
- When the goal is to merge or batch-correct RNA-seq and microarray data into a single
  combined matrix — the article explicitly found aggregation across platforms introduced
  more problems than it solved
- When single-cell resolution is needed to attribute signature expression to specific
  cell populations — bulk data cannot fully separate cellular populations
- When a dataset has not been curated for sample purity (e.g., TCGA-PAAD retains 32
  non-PDAC samples among 182) — filtering to whitelisted samples must precede scoring
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: R package supplying the parsed PACA-AU and Moffitt 2015 RNA-seq/microarray
    expression objects, sampInfo metadata, and curated gene_lists used to score signatures
    per platform
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which per-gene z-scoring, mean signature scoring, Wilcoxon rank-sum
    tests, and cross-platform score comparisons are implemented
- name: devtools
  role: Used to install pdacR from GitHub before loading the datasets and gene signatures
  repo: rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:06fd8e69c5203a09fb75bb00becfbc63e137886d971596db10f322aa39eafaaa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-platform-rnaseq-microarray-comparison

## Summary

A method for testing whether PDAC gene signatures (e.g., Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) show consistent behavior across paired bulk RNA-seq and microarray datasets by z-scoring signature genes per sample, computing mean signature scores, and comparing score distributions across matched sample classes on each platform separately.

## When to use

Use when the same gene signatures must be evaluated in both an RNA-seq and a microarray PDAC dataset (e.g., paired ICGC PACA-AU primary/metastasis/cell-line samples, or Moffitt 2015 tissue-compartment datasets) to confirm a signature trend is platform-independent.

## When NOT to use

- When the goal is to merge or batch-correct RNA-seq and microarray data into a single combined matrix — the article explicitly found aggregation across platforms introduced more problems than it solved
- When single-cell resolution is needed to attribute signature expression to specific cell populations — bulk data cannot fully separate cellular populations
- When a dataset has not been curated for sample purity (e.g., TCGA-PAAD retains 32 non-PDAC samples among 182) — filtering to whitelisted samples must precede scoring

## Inputs

- Parsed ICGC PACA-AU RNAseq expression matrix (.rds, genes x samples) with matching sampInfo metadata
- Parsed ICGC PACA-AU microarray expression matrix (.rds, genes x samples) with matching sampInfo metadata
- Moffitt 2015 RNAseq and microarray datasets ($ex genes-by-samples matrices) with tissue-of-origin sampInfo (primary PDAC, PDX, CAF line, cell line, normal pancreas, metastasis)
- Gene signature lists (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) as gene SYMBOLs from pdacR gene_lists populated by R/parse_gene_lists.R

## Outputs

- Per-sample signature score table (samples x 3 signatures with class labels and platform), e.g. icgc_gene_set_scores.csv
- Per-sample signature scores plus Wilcoxon statistics and p-values, e.g. adex_exocrine_immunogenic_scores.csv
- Grouped boxplots of signature scores by tissue type / sample class, rendered per platform

## How to apply

Load the parsed .rds/.RData expression matrices (genes x samples) and sampInfo metadata from the pdacR data/ directory, plus curated signature gene lists (gene SYMBOLs) built by R/parse_gene_lists.R. Identify matched sample classes from sampInfo metadata (primary-primary, primary-metastatic, primary-cell line pairs in PACA-AU; tissue-of-origin tracks in the Moffitt 2015 sets), excluding samples flagged by ICGC_PARSE_Nones. Within each dataset, z-score each signature gene per sample and compute the per-sample signature score as the mean of the z-scored genes, keeping RNA-seq and microarray platforms separate rather than merging them (the article notes aggregation across platforms introduced more problems than it solved). Compare scores across sample classes within each platform — e.g., Wilcoxon rank-sum tests of normal pancreas vs PDAC or primary vs CAF lines — and only accept a conclusion if the direction and significance (p < 0.0001 in the Moffitt examples) replicate on both platforms.

## Related tools

- **pdacR** (R package supplying the parsed PACA-AU and Moffitt 2015 RNA-seq/microarray expression objects, sampInfo metadata, and curated gene_lists used to score signatures per platform) — github.com/rmoffitt/pdacR
- **R** (Language in which per-gene z-scoring, mean signature scoring, Wilcoxon rank-sum tests, and cross-platform score comparisons are implemented)
- **devtools** (Used to install pdacR from GitHub before loading the datasets and gene signatures) — rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); library(pdacR)  # then load parsed PACA-AU RNAseq/microarray .rds objects and gene_lists for per-platform z-scored signature scoring
```

## Evaluation signals

- Each of the three signatures (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) shows a clear reduction in mean-centered score in derived cell lines and metastatic samples versus their paired primary tumors on BOTH PACA-AU platforms (RNAseq and microarray)
- In the Moffitt 2015 datasets, normal pancreata show significantly higher ADEX/Exocrine/Immunogenic scores than PDACs and primaries higher than CAF lines at p < 0.0001 in BOTH the microarray and RNAseq datasets
- Per-sample output table has the expected schema: one row per sample with 3 signature score columns, class labels, and a platform field
- Signature genes resolve to gene SYMBOLs with nonzero coverage in both the RNA-seq and microarray expression matrices
- Samples flagged by ICGC_PARSE_Nones are excluded from pairing and score computation

## Limitations

- Datasets cannot be robustly merged across platforms (bulk RNA-seq vs microarrays); analysis must be run per platform and conclusions rest on replication rather than pooled statistics
- Bulk transcriptomic data limit the ability to fully separate cellular populations, so ADEX/Exocrine signals may reflect non-tumor (e.g., normal pancreas) contamination rather than tumor-cell-intrinsic expression
- Filtering is essential: e.g., TCGA-PAAD has 182 samples but only 150 are confirmed PDAC, and unfiltered analyses mislead signature comparison

## Evidence

- [results] ADEX and Exocrine gene signatures are enriched in normal pancreata compared to PDACs (p < 0.0001), indicating a pancreas cell intrinsic origin for their expression: [evidence span withheld — anchor status: not_found]
- [discussion] our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object.: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
- [methods] TCGA-PAAD has 182 samples, but only 150 are PDAC: "TCGA-PAAD has 182 samples, but only 150 are PDAC."
