---
name: precision-recall-fscore-benchmarking
description: Use when you need to judge whether an scRNA-seq preprocessing step (e.g.,
  dropout imputation with scImpute, MAGIC, or SAVER) improves downstream differential-expression
  agreement with a bulk RNA-seq ground truth, via F-score comparison.
when_to_use_negative:
- When no bulk RNA-seq reference or other trusted ground-truth DE list exists for
  the same biological comparison — precision/recall/F-score cannot be computed
- When the goal is clustering accuracy or trajectory inference rather than differential-expression
  concordance; other evaluation criteria are needed
- When dropout rates in the raw data are severely high (e.g., droplet microfluidics),
  where the article notes imputation efficiency degrades and F-score comparisons may
  not be informative
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: scImpute
  role: Primary imputation method under evaluation; imputes only values with dropout
    probability above drop_thre=0.5 before DE analysis
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Comparator graph-based diffusion imputation method whose output DE lists are
    scored in the same precision/recall/F-score framework
- name: SAVER
  role: Comparator Bayesian gene-pooling imputation method scored against the bulk
    RNA-seq reference alongside scImpute
- name: DESeq2
  role: Differential-expression tool applied to each raw/imputed matrix and to bulk
    RNA-seq to derive the reference and test gene lists
- name: MAST
  role: Second differential-expression tool used to confirm the F-score ranking across
    imputation methods
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:6445a3f18319bb4f0ea5ccef3fcd0df5604703d818b0d03ba7c47077ccfc9fa3
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
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# precision-recall-fscore-benchmarking

## Summary

Benchmark imputation methods for scRNA-seq by scoring their differential-expression gene lists against a bulk RNA-seq reference using precision, recall, and F-score. This skill quantifies whether imputation (e.g., scImpute vs. raw, MAGIC, SAVER) improves agreement between single-cell and bulk DE results.

## When to use

Use when you need to judge whether an scRNA-seq preprocessing step (e.g., dropout imputation with scImpute, MAGIC, or SAVER) improves downstream differential-expression agreement with a bulk RNA-seq ground truth, via F-score comparison.

## When NOT to use

- When no bulk RNA-seq reference or other trusted ground-truth DE list exists for the same biological comparison — precision/recall/F-score cannot be computed
- When the goal is clustering accuracy or trajectory inference rather than differential-expression concordance; other evaluation criteria are needed
- When dropout rates in the raw data are severely high (e.g., droplet microfluidics), where the article notes imputation efficiency degrades and F-score comparisons may not be informative

## Inputs

- Raw scRNA-seq count matrix (e.g., GSE75748 ESC/DEC), genes as rows and cells as columns
- Imputed count matrices from scImpute (drop_thre=0.5), MAGIC, and SAVER
- Bulk RNA-seq DESeq2 differential-expression gene list used as the reference standard

## Outputs

- Per-method DE gene lists from DESeq2 and MAST at matched FDR thresholds
- Precision, recall, and F-score per imputation method against the bulk RNA-seq reference
- Comparison table ranking scImpute, MAGIC, SAVER, and raw data by DE-list concordance

## How to apply

Load the raw scRNA-seq count matrix (e.g., GSE75748 human ESC vs. definitive endoderm, genes as rows and cells as columns) and generate one DE gene list per preprocessing condition. Impute dropouts with scImpute using drop_thre = 0.5 (only values with dropout probability above this threshold are imputed), Kcluster set to the number of cell subpopulations, labeled mode where cell-type labels are available, and ncores = 10; run MAGIC and SAVER as comparators. Run differential expression with DESeq2 and MAST on each raw and imputed matrix at matched FDR thresholds, then compute precision, recall, and F-score of each scRNA-seq DE list against the bulk RNA-seq DESeq2 reference list. The rationale is that scImpute imputes only likely dropouts, whereas MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, potentially distorting precision/recall trade-offs. Correct application is confirmed when the comparison table shows scImpute yielding roughly a 10% F-score gain over unimputed data and a more favorable precision/recall balance than MAGIC and SAVER.

## Related tools

- **scImpute** (Primary imputation method under evaluation; imputes only values with dropout probability above drop_thre=0.5 before DE analysis) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Comparator graph-based diffusion imputation method whose output DE lists are scored in the same precision/recall/F-score framework)
- **SAVER** (Comparator Bayesian gene-pooling imputation method scored against the bulk RNA-seq reference alongside scImpute)
- **DESeq2** (Differential-expression tool applied to each raw/imputed matrix and to bulk RNA-seq to derive the reference and test gene lists)
- **MAST** (Second differential-expression tool used to confirm the F-score ranking across imputation methods)

## Examples

```
scimpute(count_path="raw_count.csv", infile="csv", outfile="csv", out_dir="./", drop_thre=0.5, Kcluster=2, ncores=10)
```

## Evaluation signals

- The comparison table reproduces the reported pattern: scImpute achieves approximately 10% higher F-scores than DE analysis on raw unimputed scRNA-seq data
- scImpute shows a more favorable precision/recall trade-off than MAGIC and SAVER, which alter all gene expression levels including those unaffected by dropouts
- All methods are evaluated at matched FDR thresholds and with both DESeq2 and MAST, so the ranking is consistent across the two DE tools
- Each DE gene list is compared against the same bulk RNA-seq DESeq2 reference list, so precision + recall-derived F-scores are computed on an identical reference set

## Limitations

- MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases and eliminating meaningful biological variation, which complicates fair comparison
- Imputation efficiency degrades when dropout rates in the raw data are severely high (droplet-based technologies); noted as a future direction
- The choice of parameter K (number of cell populations) is left to the user without a definitive selection procedure, affecting scImpute's imputed output and hence F-scores
- No software version for scImpute is specified, so exact reproducibility of the F-score gains is not guaranteed

## Evidence

- [abstract] scImpute targets dropouts only: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [intro] Comparator methods alter all values: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [methods] drop_thre parameter semantics: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] scimpute call signature with thresholds: "labeled = FALSE,          # cell type labels not available
         drop_thre = 0.5,          # threshold set on dropout probability
         Kcluster = 2,             # 2 cell subpopulations"
