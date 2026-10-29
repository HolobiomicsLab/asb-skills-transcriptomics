---
name: true-zero-fraction-quantification
description: Use when matched bulk and single-cell RNA-seq data are available and
  you must test whether an imputation method preserves biologically silenced (true-zero)
  genes while still correcting dropout events.
when_to_use_negative:
- No matched bulk RNA-seq data exist to define which single-cell zeros are biological
  true zeros versus dropouts
- The goal is overall expression-recovery or clustering accuracy rather than true-zero
  preservation behavior of an imputer
- The single-cell matrix has not been imputed yet (the analysis evaluates zeros in
  imputed output, not the raw matrix)
edam_operation: http://edamontology.org/operation_3287
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation of the preprocessed single-cell matrix
    whose imputed zero fractions are quantified per bulk-derived bin
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scImpute
  role: Comparison imputation method benchmarked on true-zero retention
- name: MAGIC
  role: Comparison imputation method benchmarked on true-zero retention
- name: drImpute
  role: Comparison imputation method benchmarked on true-zero retention
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:23a67c8066504b5726434f48079417594cd72e5f7a6ea0599d524efe1e5416d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - scImpute
  edam_operation: http://edamontology.org/operation_3287
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# true-zero-fraction-quantification

## Summary

Quantify how an imputation method treats biologically silent true zeros versus dropout zeros by binning genes according to matched bulk RNA-seq expression and computing per-bin zero fractions in the imputed single-cell data. Used to benchmark whether an imputer (e.g. AutoImpute) retains genuine zeros while still imputing dropouts.

## When to use

Use when matched bulk and single-cell RNA-seq data are available and you must test whether an imputation method preserves biologically silenced (true-zero) genes while still correcting dropout events.

## When NOT to use

- No matched bulk RNA-seq data exist to define which single-cell zeros are biological true zeros versus dropouts
- The goal is overall expression-recovery or clustering accuracy rather than true-zero preservation behavior of an imputer
- The single-cell matrix has not been imputed yet (the analysis evaluates zeros in imputed output, not the raw matrix)

## Inputs

- Matched bulk RNA-seq replicate matrix (e.g. Trapnell myoblast, GEO GSE52529)
- Raw single-cell count matrix (cells x genes, .csv or .mat)
- Imputed single-cell matrices from the methods under comparison (AutoImpute, scImpute, MAGIC, drImpute)

## Outputs

- Per-bin zero-fraction table mapping bulk-derived expression bins (including the biologically-silent bin) to zero fractions in imputed data (e.g. true_zero_fractions.csv)
- Qualitative verdict: does the imputer retain true zeros (high zero fraction in silent bin) while imputing dropouts (decreasing zero fraction with increasing bulk expression)?

## How to apply

Download matched bulk and single-cell data (e.g. Trapnell myoblast data from GEO GSE52529), TMM-normalize and log2-transform bulk replicates, then compute median expression across replicates and bin genes: zero-bulk-expression genes form the biologically-silent bin, the rest are grouped by median expression level. Preprocess the single-cell matrix (gene filtering at >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion gene selection, log2 transform with pseudo-count 1), then impute it — e.g. with AutoImpute using an overcomplete autoencoder, 2000 hidden units, lambda=1, initial learning rate 1e-4, convergence threshold 1e-4. For each bulk-derived bin, compute the fraction of zeros in the imputed single-cell matrix and report per-bin zero fractions (on natural-log scale, as in the paper). Correct application shows a high zero fraction in the biologically-silent bin and a decreasing zero fraction as bulk median expression increases, indicating true zeros are retained while dropouts are imputed.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation of the preprocessed single-cell matrix whose imputed zero fractions are quantified per bulk-derived bin) — https://github.com/divyanshu-talwar/AutoImpute
- **scImpute** (Comparison imputation method benchmarked on true-zero retention)
- **MAGIC** (Comparison imputation method benchmarked on true-zero retention)
- **drImpute** (Comparison imputation method benchmarked on true-zero retention)

## Examples

```
python AutoImpute.py --data trapnell.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- Biologically-silent bin (zero bulk expression) has the highest zero fraction among all bins in the imputed matrix
- Zero fraction decreases monotonically (trend-wise) as bulk median expression increases
- AutoImpute retains the largest fraction of true-zero genes compared with scImpute, MAGIC, and drImpute
- Zero fractions are reported on natural-log scale as in the paper, and saved per-bin values (e.g. true_zero_fractions.csv) match the bin definitions derived from TMM-normalized bulk medians

## Limitations

- Requires matched bulk and single-cell data from the same biological system; without bulk data the biologically-silent bin cannot be defined
- Conclusion is limited to the tested AutoImpute configuration (2000 hidden units, lambda=1, learning rate 1e-4, threshold 1e-4); other hyperparameters may change zero retention
- Results depend on the upstream preprocessing (gene filter >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion genes, log2 with pseudo-count 1)

## Evidence

- [abstract] true-zero retention rationale: "imputes the missing values accordingly with minimal modification to the biologically silent genes"
- [intro] AutoImpute mechanism: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [readme] AutoImpute input format for the imputed matrix whose zeros are quantified: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
