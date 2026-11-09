---
name: bulk-single-cell-matched-analysis
description: Use when matched bulk and single-cell RNA-seq data exist for the same
  biological system and the goal is to distinguish biologically-driven true zeros
  from technical dropout zeros after imputation.
when_to_use_negative:
- No matched bulk RNA-seq data exist for the same cell type/system, so true-zero vs
  dropout zeros cannot be distinguished
- The analysis goal is general imputation accuracy on masked entries (subsampling-based
  recovery) rather than biological true-zero retention
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Autoencoder-based imputation of the preprocessed single-cell matrix; its conservatism
    toward biologically silent genes is the property under test
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scImpute
  role: Comparison imputation method for per-bin zero-fraction benchmarking
- name: MAGIC
  role: Comparison imputation method for per-bin zero-fraction benchmarking
- name: drImpute
  role: Comparison imputation method for per-bin zero-fraction benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:4f67a110f89237d45dee60a5a461ee0d81856481119408e6accfa891e8306d1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bulk-single-cell-matched-analysis

## Summary

Assess whether an scRNA-seq imputation method (e.g. AutoImpute) retains biologically silent true zeros while imputing dropouts, by binning matched single-cell genes by bulk RNA-seq median expression and computing per-bin zero fractions in the imputed matrix.

## When to use

Use when matched bulk and single-cell RNA-seq data exist for the same biological system and the goal is to distinguish biologically-driven true zeros from technical dropout zeros after imputation.

## When NOT to use

- No matched bulk RNA-seq data exist for the same cell type/system, so true-zero vs dropout zeros cannot be distinguished
- The analysis goal is general imputation accuracy on masked entries (subsampling-based recovery) rather than biological true-zero retention

## Inputs

- Matched bulk RNA-seq count matrix with biological replicates (e.g. Trapnell myoblast replicates, GSE52529)
- Matched single-cell RNA-seq count matrix (genes x cells)
- Optional: imputed matrices from comparison methods (scImpute, MAGIC, drImpute)

## Outputs

- Per-bin zero-fraction values on natural-log scale (true_zero_fractions.csv)
- Verification of the retention/imputation trend across bulk expression bins

## How to apply

Download matched bulk and single-cell data (e.g. Trapnell myoblast data, GEO GSE52529) and compute median bulk expression across replicates (TMM normalization plus log2 transform), binning genes with zero bulk expression into a biologically-silent bin and grouping remaining genes by median expression level. Preprocess the single-cell matrix per the AutoImpute pipeline: gene filtering at >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion gene selection, and log2 transform with pseudo-count 1. Impute with AutoImpute (overcomplete autoencoder, RMSProp, hidden layer of 2000 units, lambda=1, initial learning rate 1e-4, convergence threshold 1e-4) and optionally scImpute, drImpute, or MAGIC for comparison. For each bulk-derived expression bin, compute the fraction of zeros (zeros in the set divided by total count of the set) in the imputed single-cell data on a natural-log scale. Correct application shows a high zero fraction in the biologically-silent bin and a decreasing zero fraction as bulk median expression increases, indicating true zeros are retained while dropouts are imputed.

## Related tools

- **AutoImpute** (Autoencoder-based imputation of the preprocessed single-cell matrix; its conservatism toward biologically silent genes is the property under test) — https://github.com/divyanshu-talwar/AutoImpute
- **scImpute** (Comparison imputation method for per-bin zero-fraction benchmarking)
- **MAGIC** (Comparison imputation method for per-bin zero-fraction benchmarking)
- **drImpute** (Comparison imputation method for per-bin zero-fraction benchmarking)

## Examples

```
python AutoImpute.py --data trapnell.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- Highest zero fraction occurs in the biologically-silent bin (bulk expression = 0)
- Zero fraction decreases monotonically (or near-monotonically) as bulk median expression bin increases
- AutoImpute retains the largest fraction of true zeros among compared methods while still reducing zeros for expressed genes
- Per-bin values are reported on natural-log scale as in the paper and saved to true_zero_fractions.csv
- Preprocessing reproducibility: single-cell matrix filtered at >=3 reads in >=3 cells, top-1000 high-dispersion genes retained

## Limitations

- Analysis requires matched bulk/single-cell data from the same system; unmatched datasets cannot support the true-zero binning
- AutoImpute requires a pre-processed count matrix with genes as columns and cells as rows; applying it to unprocessed or wrongly oriented matrices fails
- The pre-processing R script in the repository must be used for any dataset other than the bundled example

## Evidence

- [abstract] true-zero retention: "imputes the missing values accordingly with minimal modification to the biologically silent genes"
- [intro] true-zero retention mechanism: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [readme] input format requirement: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] imputation invocation: "python AutoImpute.py --data <dataset_name>.{mat, csv}"
