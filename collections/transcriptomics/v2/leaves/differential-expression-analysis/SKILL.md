---
name: differential-expression-analysis
description: Use when raw or imputed scRNA-seq count matrices (genes as rows, cells
  as columns, e.g. GSE75748 ESC/DEC) need DE gene lists between subpopulations, or
  when imputation impact on DE accuracy must be benchmarked against a bulk RNA-seq
  reference.
when_to_use_negative:
- Input is bulk RNA-seq data — the dropout phenomenon motivating imputation is a key
  characteristic of scRNA-seq, not bulk averages across cells
- The goal is clustering of cell subpopulations or dimension reduction rather than
  differential expression gene lists
- Data come from droplet microfluidics with severely high dropout rates, where imputation
  efficiency degrades and downstream DE lists may be unreliable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Imputes dropout values (only those with dropout probability above drop_thre)
    in the raw count matrix prior to differential gene expression analysis, improving
    DE accuracy
  repo: https://github.com/Vivianstats/scImpute
- name: DESeq2
  role: Differential expression tool run on raw and imputed scRNA-seq matrices and
    on the bulk RNA-seq reference data to derive DE gene lists
- name: MAST
  role: Second differential expression tool run on raw and imputed matrices at matched
    FDR thresholds to derive DE gene lists
- name: MAGIC
  role: Comparator graph-based diffusion imputation method whose altered expression
    levels make its DE lists less concordant with bulk references
- name: SAVER
  role: Comparator Bayesian gene-pooling imputation method that alters all gene expression
    levels including those unaffected by dropouts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:6556554b9424e8c8a456b3525afb16122bb901a22b396beaaf12fdcf4f0c8050
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differential-expression-analysis

## Summary

Differential gene expression analysis on single-cell RNA-seq count matrices — run before or after dropout imputation (scImpute) to derive DE gene lists between cell populations, and evaluate their concordance with bulk RNA-seq DE references via precision, recall, and F-score.

## When to use

Use when raw or imputed scRNA-seq count matrices (genes as rows, cells as columns, e.g. GSE75748 ESC/DEC) need DE gene lists between subpopulations, or when imputation impact on DE accuracy must be benchmarked against a bulk RNA-seq reference.

## When NOT to use

- Input is bulk RNA-seq data — the dropout phenomenon motivating imputation is a key characteristic of scRNA-seq, not bulk averages across cells
- The goal is clustering of cell subpopulations or dimension reduction rather than differential expression gene lists
- Data come from droplet microfluidics with severely high dropout rates, where imputation efficiency degrades and downstream DE lists may be unreliable

## Inputs

- Raw scRNA-seq count matrix (genes as rows, cells as columns), e.g. GSE75748 human ESC vs definitive endoderm (ESC/DEC) data
- Optional imputed count matrices from scImpute, MAGIC, and SAVER for comparison
- Cell-type labels for labeled imputation mode (when available)
- Bulk RNA-seq DE gene list (e.g. DESeq2 results) as the concordance reference

## Outputs

- DE gene lists per condition/imputation method at matched FDR thresholds
- Precision, recall, and F-score table comparing each DE gene list against the bulk RNA-seq reference
- Column indices of outlier cells excluded from imputation (returned by `scimpute`)

## How to apply

Load the raw count matrix with genes as rows and cells as columns, and optionally impute dropouts first with scImpute's `scimpute` function using drop_thre = 0.5 (only values with dropout probability above this threshold are imputed), Kcluster equal to the number of cell subpopulations, and labeled mode where cell-type labels are available. Run differential expression with DESeq2 and MAST on each raw and imputed matrix, deriving DE gene lists at matched FDR thresholds across methods. Compare each DE gene list against bulk RNA-seq DESeq2 results to compute precision, recall, and F-score per imputation method. The rationale is that scImpute only imputes likely dropouts, whereas MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases; in the ESC vs definitive endoderm (ESC/DEC) comparison this makes scImpute-imputed DE lists roughly 10% more concordant (F-score) with bulk than DE on raw data.

## Related tools

- **scImpute** (Imputes dropout values (only those with dropout probability above drop_thre) in the raw count matrix prior to differential gene expression analysis, improving DE accuracy) — https://github.com/Vivianstats/scImpute
- **DESeq2** (Differential expression tool run on raw and imputed scRNA-seq matrices and on the bulk RNA-seq reference data to derive DE gene lists)
- **MAST** (Second differential expression tool run on raw and imputed matrices at matched FDR thresholds to derive DE gene lists)
- **MAGIC** (Comparator graph-based diffusion imputation method whose altered expression levels make its DE lists less concordant with bulk references)
- **SAVER** (Comparator Bayesian gene-pooling imputation method that alters all gene expression levels including those unaffected by dropouts)

## Examples

```
scimpute(count_path="raw_count.csv", infile="csv", outfile="csv", out_dir="./", labeled=TRUE, drop_thre=0.5, Kcluster=2, ncores=10)
```

## Evaluation signals

- scImpute-imputed DE gene lists achieve approximately 10% higher F-scores against the bulk RNA-seq reference than DE on the raw (unimputed) data
- The precision/recall trade-off for scImpute-imputed DE lists is more favorable than for MAGIC- and SAVER-imputed lists
- The imputed output matrix has the same dimensions as the input raw count matrix (same genes and cells)
- FDR thresholds are matched across DE runs (DESeq2, MAST, raw vs imputed) so F-score comparisons are fair
- Only values with dropout probability larger than drop_thre were changed — non-dropout expression levels are preserved, not smoothed as by MAGIC/SAVER

## Limitations

- MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases and eliminating meaningful biological variation
- Choice of the K (number of cell populations) parameter is left to the user without a definitive selection procedure
- Imputation efficiency degrades when dropout rates in raw data are severely high, e.g. droplet-based technologies
- Computation efficiency can be largely improved if a filtering step on cells is performed based on biological knowledge, but such a step is not specified
- No software version for scImpute is specified

## Evidence

- [abstract] scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis, and aid the study of gene expression dynamics.: "scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis"
- [intro] Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases and eliminating meaningful biological variation.: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [readme] `scImpute` can be applied to raw read count matrix before the users perform downstream analyses such as differential gene expression analysis.: "`scImpute` can be applied to raw read count matrix before the users perform downstream analyses such as"
- [methods] Only the values that have dropout probability larger than `drop_thre` are imputed by `scImpute`.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
