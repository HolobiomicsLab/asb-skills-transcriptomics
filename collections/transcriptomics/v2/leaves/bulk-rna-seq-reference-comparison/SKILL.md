---
name: bulk-rna-seq-reference-comparison
description: Use when a paired bulk RNA-seq dataset exists for the same cell-type
  or condition contrast as scRNA-seq data, and you need to quantify whether imputation
  improves DE gene-list concordance rather than relying on qualitative checks.
when_to_use_negative:
- No bulk RNA-seq data exist for the same biological contrast, so there is no reference
  DE gene list to compare against
- Goal is to evaluate imputation quality for clustering or trajectory inference rather
  than differential expression accuracy
- The scRNA-seq data come from a protocol with severely high dropout rates (e.g. droplet
  microfluidics), where imputation efficiency is a known limitation of scImpute
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scImpute
  role: Primary imputation of dropouts on the raw count matrix; values with dropout
    probability above drop_thre = 0.5 are imputed, applied before DESeq2/MAST DE testing
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Comparator graph-based diffusion imputation method whose full-matrix smoothing
    is expected to give a worse precision/recall trade-off
- name: SAVER
  role: Comparator Bayesian gene-pooling imputation method that alters all gene expression
    levels, included as a baseline in the F-score comparison
- name: DESeq2
  role: Differential expression tool used both to derive the bulk RNA-seq reference
    DE list and scRNA-seq DE lists
- name: MAST
  role: Second DE framework applied to raw and imputed matrices to confirm the ranking
    of imputation methods
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:7a8f9e85ec92b5c23158c85ad7607689cf679d134e3c4f43621b23a090f6c463
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - DESeq2
  - MAGIC
  - MAST
  - SAVER
  - scImpute
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bulk-rna-seq-reference-comparison

## Summary

Benchmark scRNA-seq differential expression gene lists (raw vs. imputed by scImpute, MAGIC, SAVER) against bulk RNA-seq DESeq2 results on the same biological comparison, using precision, recall, and F-score to judge imputation quality. In the ESC vs. definitive endoderm case (GSE75748), scImpute-imputed lists gain roughly 10% F-score over raw data.

## When to use

Use when a paired bulk RNA-seq dataset exists for the same cell-type or condition contrast as scRNA-seq data, and you need to quantify whether imputation improves DE gene-list concordance rather than relying on qualitative checks.

## When NOT to use

- No bulk RNA-seq data exist for the same biological contrast, so there is no reference DE gene list to compare against
- Goal is to evaluate imputation quality for clustering or trajectory inference rather than differential expression accuracy
- The scRNA-seq data come from a protocol with severely high dropout rates (e.g. droplet microfluidics), where imputation efficiency is a known limitation of scImpute

## Inputs

- Raw scRNA-seq whole-genome count matrix (genes x cells), e.g. GSE75748 ESC/DEC data
- Bulk RNA-seq count data for the same ESC vs. definitive endoderm comparison (for DESeq2 reference DE list)
- Cell-type labels for the labeled scImpute mode (ESC vs. DEC)

## Outputs

- Imputed count matrices (scimpute_count.csv and equivalents for MAGIC/SAVER) with same dimensions as input
- DE gene lists per method (raw, scImpute, MAGIC, SAVER) from DESeq2 and MAST at matched FDR thresholds
- Comparison table of precision, recall, and F-score per imputation method against the bulk RNA-seq DESeq2 reference

## How to apply

Load the whole-genome scRNA-seq raw count matrix (genes as rows, cells as columns; here GSE75748 H1 ESC vs. definitive endoderm) and impute it with scImpute using drop_thre = 0.5 so only values with dropout probability above the threshold are imputed, Kcluster matching the number of ESC/DEC subpopulations, labeled mode where cell-type labels are available, and ncores = 10 for parallel computation. Impute the same matrix with MAGIC (graph-based diffusion) and SAVER (Bayesian gene-pooling) as comparator methods, noting that both alter all gene expression levels including those unaffected by dropouts. Run differential expression analysis with DESeq2 and MAST on each raw and imputed matrix at matched FDR thresholds to derive DE gene lists. Compute each list's precision, recall, and F-score against the bulk RNA-seq DESeq2 reference gene list for the same contrast. Rationale: scImpute only targets likely dropouts, so imputed lists should match bulk biology more faithfully; verify the reported pattern of ~10% F-score gain for scImpute over raw data and a better precision/recall trade-off than MAGIC and SAVER.

## Related tools

- **scImpute** (Primary imputation of dropouts on the raw count matrix; values with dropout probability above drop_thre = 0.5 are imputed, applied before DESeq2/MAST DE testing) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Comparator graph-based diffusion imputation method whose full-matrix smoothing is expected to give a worse precision/recall trade-off)
- **SAVER** (Comparator Bayesian gene-pooling imputation method that alters all gene expression levels, included as a baseline in the F-score comparison)
- **DESeq2** (Differential expression tool used both to derive the bulk RNA-seq reference DE list and scRNA-seq DE lists)
- **MAST** (Second DE framework applied to raw and imputed matrices to confirm the ranking of imputation methods)

## Examples

```
scimpute(count_path="raw_count.csv", infile="csv", outfile="csv", out_dir="./", labeled=TRUE, drop_thre=0.5, Kcluster=2, ncores=10)
```

## Evaluation signals

- scImpute-imputed DE lists achieve approximately 10% higher F-score against the bulk RNA-seq DESeq2 reference than lists from raw unimputed data
- Precision/recall trade-off for scImpute is more favorable than for MAGIC and SAVER across the matched FDR thresholds
- Imputed output matrix has the same dimension as the raw input count matrix, and only values with dropout probability above drop_thre are changed
- scImpute returns indices of detected outlier cells, which should be excluded from imputation; these cells must be accounted for downstream
- Ranking of methods is consistent across both DE frameworks (DESeq2 and MAST)

## Limitations

- Choice of the parameter K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen from clustering of the raw data and the desired resolution
- Imputation efficiency degrades when dropout rates in raw data are severely high, as with droplet-based technologies; noted as a future direction
- Computation can be large for many cells; a filtering step on cells based on biological knowledge improves efficiency but such prior knowledge is not always specified
- MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases and eliminating meaningful biological variation
- No specific software version for scImpute is stated

## Evidence

- [abstract] scImpute selectively imputes dropouts: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [intro] MAGIC/SAVER alter all values: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [methods] drop_thre filter step: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] Input matrix orientation: "Its only input is the raw count matrix with rows representing genes and columns representing cells."
- [readme] Parallel computing recommendation: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
