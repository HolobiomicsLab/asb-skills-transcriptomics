---
name: imputation-method-benchmark-comparison
description: Use when comparing how much different imputation strategies (scImpute,
  drImpute, MAGIC, AutoImpute) improve cell-type clustering of sparse scRNA-seq count
  matrices, measured by ARI against original cell annotations.
when_to_use_negative:
- Datasets so large that third-party imputers exhaust system memory (in the paper,
  scImpute/drImpute/MAGIC could not be run on ~68,000 PBMC cells, so only AutoImpute
  was evaluated there) — restrict the comparison accordingly.
- Raw (non-log-transformed, non-preprocessed) count matrices — AutoImpute.py expects
  a pre-processed count single-cell matrix; raw data must first pass through the R
  pre-processing pipeline.
- Evaluating imputation quality by expression-value recovery on masked entries rather
  than downstream clustering — this skill scores ARI of K-means clustering only.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation of the sparse scRNA-seq count matrix;
    its outputs are one arm of the benchmark and are expected to yield the best ARI
    on five of the eight datasets
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides the K-means clustering (K set to the number of unique cell-types
    per dataset) whose labels are scored against original annotations with ARI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:5b70b112733ea9ecba3f2e5872a00c6772d0eec1a8326ab88a7298781f364642
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# imputation-method-benchmark-comparison

## Summary

Benchmarks single-cell RNA-seq dropout imputation methods (scImpute, drImpute, MAGIC, AutoImpute) against a no-imputation baseline by running K-means on log2-transformed expression matrices and scoring cell-clustering accuracy with Adjusted Rand Index, exactly as in the AutoImpute paper's Table 1.

## When to use

Use when comparing how much different imputation strategies (scImpute, drImpute, MAGIC, AutoImpute) improve cell-type clustering of sparse scRNA-seq count matrices, measured by ARI against original cell annotations.

## When NOT to use

- Datasets so large that third-party imputers exhaust system memory (in the paper, scImpute/drImpute/MAGIC could not be run on ~68,000 PBMC cells, so only AutoImpute was evaluated there) — restrict the comparison accordingly.
- Raw (non-log-transformed, non-preprocessed) count matrices — AutoImpute.py expects a pre-processed count single-cell matrix; raw data must first pass through the R pre-processing pipeline.
- Evaluating imputation quality by expression-value recovery on masked entries rather than downstream clustering — this skill scores ARI of K-means clustering only.

## Inputs

- Pre-processed, log2-transformed scRNA-seq expression matrices (cells as rows, genes as columns; .csv or .mat format) for eight datasets: Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, Zeisel
- Original cell-type annotation labels per dataset (ground truth for ARI)
- Imputed matrices produced by scImpute, drImpute, MAGIC, and AutoImpute

## Outputs

- ari_table.csv: a table with rows for the eight datasets and columns for no-imputation, scImpute, drImpute, MAGIC, and AutoImpute
- Per dataset-method pair Adjusted Rand Index values comparing K-means cluster labels to original cell annotations

## How to apply

Load the pre-processed, log2-transformed expression matrices (cells as rows, genes as columns) for each dataset (e.g. Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, Zeisel) from the package data directory. Impute each matrix with AutoImpute (python AutoImpute.py, a TensorFlow overcomplete autoencoder trained ~7000 iterations at learning rate 0.0001 with lambda_val 1) and with scImpute, drImpute, and MAGIC. Run scikit-learn K-means with the initialization parameter K set to the number of unique cell-types per dataset, once on the unimputed log-transformed matrix and once per imputed matrix. Compute the Adjusted Rand Index between K-means cluster labels and the original cell-type annotations for every dataset-method pair, then assemble the ARI values into a table with rows for datasets and columns for no-imputation, scImpute, drImpute, MAGIC, and AutoImpute (ari_table.csv). The rationale is that a good imputation should preserve or sharpen biologically driven structure, so AutoImpute's autoencoder objective — learning the input distribution while minimally modifying biologically silent genes — should raise ARI above the no-imputation baseline.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation of the sparse scRNA-seq count matrix; its outputs are one arm of the benchmark and are expected to yield the best ARI on five of the eight datasets) — github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides the K-means clustering (K set to the number of unique cell-types per dataset) whose labels are scored against original annotations with ARI)

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --iterations 7000 --imputed_save imputed_matrix
```

## Evaluation signals

- All ARI values in ari_table.csv fall within [0, 1] (valid range for Adjusted Rand Index).
- The ordering of ARI values reproduces Table 1 of the AutoImpute paper.
- AutoImpute attains the best ARI among all evaluated imputation methods on five of the eight datasets.
- AutoImpute's ARI exceeds the no-imputation K-means baseline on every tested dataset.
- K-means is run with K equal to the number of unique cell-types in each dataset, so cluster-label cardinalities should match the number of annotated cell-types.

## Limitations

- Memory limits prevent running scImpute, drImpute, and MAGIC on very large datasets (~68,000 PBMC cells); only AutoImpute was evaluated there, leaving those table cells incomplete.
- AutoImpute.py expects Python 2.7 with numpy, scikit-learn, tensorflow, and matplotlib, and its R pre-processing requires many packages (R.matlab, Matrix, ggplot2, Rtsne, svd, plyr, dplyr, data.table, mclust, flexclust, reshape2, irlba, dynamicTreeCut, RColorBrewer, GenSA, gplots).
- The K-means result depends on the initialization parameter K; the paper fixes K to the number of unique cell-types, so performance may vary under different K choices.

## Evidence

- [abstract] AutoImpute performed competitively against existing single-cell imputation methods on expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and cell-type separability: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
- [intro] Overcomplete autoencoder formulation of the imputation method: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [readme] Expected input format of the AutoImpute model: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] R pre-processing command for new datasets: "Rscript pre-process.R <input-file-name> <dataset-name>"
