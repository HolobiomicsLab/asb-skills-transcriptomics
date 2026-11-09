---
name: gene-expression-binning-from-bulk-replicates
description: Use when matched bulk RNA-seq replicates and a single-cell count matrix
  are available and you need to partition genes by bulk expression level (true-zero
  silent genes vs. expressed bins) to test whether an imputation method retains biological
  zeros while imputing dropouts.
when_to_use_negative:
- No matched bulk replicates exist for the same population — the true-zero (biologically
  silent) bin cannot be defined without zero-bulk-expression evidence
- The analysis goal is general imputation accuracy (e.g., masked-value recovery or
  clustering accuracy) rather than true-zero retention assessment
- Single-cell data are not preprocessed per the AutoImpute pipeline (gene filter >=3
  reads in >=3 cells, median library-size normalization, top-1000 high-dispersion
  genes, log2 with pseudo-count 1), making zero fractions incomparable to the paper's
  benchmark
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation (hidden units 2000, lambda=1, initial
    learning rate 1e-4, threshold 1e-4) producing the imputed single-cell matrix whose
    zeros are measured per bulk bin
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scImpute
  role: Optional comparison imputation method for per-bin zero-fraction benchmarking
    against AutoImpute
- name: drImpute
  role: Optional comparison imputation method for per-bin zero-fraction benchmarking
    against AutoImpute
- name: MAGIC
  role: Optional comparison imputation method for per-bin zero-fraction benchmarking
    against AutoImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:58fe3fdaf581ebeb57271297ba7344cbcdf84baddc1b59cf9f65e7909e4d64b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Gene-expression binning from bulk replicates

## Summary

Bins genes by matched bulk RNA-seq median expression (with zero-bulk genes assigned to a biologically-silent true-zero bin) so that zero-retention in imputed single-cell data can be measured per expression level, as done for the Trapnell myoblast data in the AutoImpute paper.

## When to use

Use when matched bulk RNA-seq replicates and a single-cell count matrix are available and you need to partition genes by bulk expression level (true-zero silent genes vs. expressed bins) to test whether an imputation method retains biological zeros while imputing dropouts.

## When NOT to use

- No matched bulk replicates exist for the same population — the true-zero (biologically silent) bin cannot be defined without zero-bulk-expression evidence
- The analysis goal is general imputation accuracy (e.g., masked-value recovery or clustering accuracy) rather than true-zero retention assessment
- Single-cell data are not preprocessed per the AutoImpute pipeline (gene filter >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion genes, log2 with pseudo-count 1), making zero fractions incomparable to the paper's benchmark

## Inputs

- Matched bulk RNA-seq count matrix with biological replicates (e.g., three replicates of undifferentiated myoblasts from GSE52529)
- Matched single-cell RNA-seq raw count matrix (.csv, cells as rows, genes as columns) for the same population
- Optional: imputed single-cell matrices from scImpute, drImpute, or MAGIC for method comparison

## Outputs

- Bulk-derived gene-expression bins (biologically-silent true-zero bin plus bins by median bulk expression level)
- Per-bin zero-fraction table in the imputed single-cell data, on natural-log scale (true_zero_fractions.csv)
- Trend verdict: high zero fraction in the silent bin and decreasing zero fraction as bulk median expression increases

## How to apply

Download matched bulk and single-cell RNA-seq data (e.g., Trapnell myoblast data, GEO GSE52529) and TMM-normalize the bulk replicates followed by log2 transformation to check for composition bias. Compute the median gene expression across the three biological replicates of undifferentiated myoblasts: genes with zero bulk expression go into the biologically-silent (true-zero) bin, remaining genes are grouped by median expression level. Preprocess the matched single-cell matrix with the AutoImpute pipeline — gene filtering at >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion gene selection, and log2 transform with pseudo-count 1 — then impute with AutoImpute (overcomplete autoencoder, hidden layer 2000 units, lambda=1, RMSProp, initial learning rate 1e-4, convergence threshold 1e-4). For each bulk-derived expression bin, compute the fraction of zeros (zeros in the set divided by total count of the set) in the imputed single-cell data, report on natural-log scale, and save per-bin zero-fraction values to true_zero_fractions.csv.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation (hidden units 2000, lambda=1, initial learning rate 1e-4, threshold 1e-4) producing the imputed single-cell matrix whose zeros are measured per bulk bin) — github.com/divyanshu-talwar/AutoImpute
- **scImpute** (Optional comparison imputation method for per-bin zero-fraction benchmarking against AutoImpute)
- **drImpute** (Optional comparison imputation method for per-bin zero-fraction benchmarking against AutoImpute)
- **MAGIC** (Optional comparison imputation method for per-bin zero-fraction benchmarking against AutoImpute)

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley && python AutoImpute.py --data blakeley.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- Zero-fraction in the biologically-silent (zero-bulk-expression) bin is the highest of all bins, indicating true zeros are retained rather than imputed
- Zero fraction decreases monotonically (or near-monotonically) as bulk median expression increases, indicating effective dropout imputation
- Composition-bias check passes: TMM normalization followed by log2 transformation applied to the three bulk replicates before binning
- Single-cell matrix dimensions and gene set match the AutoImpute preprocessing (genes with >=3 reads in >=3 cells; top-1000 high-dispersion genes)
- Output CSV (true_zero_fractions.csv) contains one zero-fraction value per bulk-derived bin, on natural-log scale, for each imputation method compared

## Limitations

- True-zero bin validity depends entirely on matched bulk data quality; bulk dropouts or batch effects can mislabel expressed genes as biologically silent
- AutoImpute was the most conservative method (retained the largest fraction of true zeros), so less conservative methods may show lower zero retention in the silent bin
- The AutoImpute.py model requires Python 2.7 (numpy, scikit-learn, tensorflow, matplotlib) and R pre-processing dependencies, which may be difficult to reproduce in modern environments
- Median-based binning across only three biological replicates is sensitive to replicate variability in bulk expression estimates

## Evidence

- [abstract] AutoImpute imputes dropouts with minimal modification to biologically silent genes: "imputes the missing values accordingly with minimal modification to the biologically silent genes"
- [intro] AutoImpute learns the inherent distribution of input scRNA-seq data and preserves biologically silent values: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values"
- [readme] README specifies the input format for AutoImpute.py: a pre-processed count matrix with columns as genes and rows as cells: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] README documents the R pre-processing command used to prepare a dataset for AutoImpute: "Rscript pre-process.R <input-file-name> <dataset-name>"
