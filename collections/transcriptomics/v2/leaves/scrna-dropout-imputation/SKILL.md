---
name: scrna-dropout-imputation
description: Use when a preprocessed scRNA-seq count matrix contains excessive zero
  dropouts and the goal is expression-matrix completion that retains true (biologically
  silent) zeros rather than overwriting them.
when_to_use_negative:
- Input is a bulk RNA-seq matrix or any fully observed (non-sparse) expression matrix
  — dropout imputation is only meaningful for sparse single-cell data
- The matrix has not been pre-processed (gene filtering, normalization, top-1000 high-dispersion
  selection); AutoImpute.py expects the pre-processed count matrix as input
- The goal is differential-expression testing rather than matrix completion prior
  to downstream clustering or variance analysis
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder model that imputes dropout zeros in the preprocessed
    scRNA-seq matrix
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scImpute
  role: Comparison imputation method used to benchmark true-zero retention conservativeness
- name: drImpute
  role: Comparison imputation method used to benchmark true-zero retention conservativeness
- name: MAGIC
  role: Comparison imputation method used to benchmark true-zero retention conservativeness
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:9dc616d0cd65f35c98c7caca43172615382ca9eb15af1def5427bea36750ce58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - DCA
  - DeepImpute
  - MAGIC
  - SAUCIE
  - SAVER
  - SERM
  - SciPy
  - drImpute
  - mcImpute
  - scImpute
  - scScope
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- scrna-seq-dropout-imputation
- scrna-seq-dropout-imputation-benchmarking
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# scrna-dropout-imputation

## Summary

Impute dropout events in sparse single-cell RNA-seq count matrices using AutoImpute, an overcomplete-autoencoder approach that preserves biologically silent true zeros. Evaluated against scImpute, drImpute, and MAGIC using matched bulk/single-cell Trapnell (GSE52529) data.

## When to use

Use when a preprocessed scRNA-seq count matrix contains excessive zero dropouts and the goal is expression-matrix completion that retains true (biologically silent) zeros rather than overwriting them.

## When NOT to use

- Input is a bulk RNA-seq matrix or any fully observed (non-sparse) expression matrix — dropout imputation is only meaningful for sparse single-cell data
- The matrix has not been pre-processed (gene filtering, normalization, top-1000 high-dispersion selection); AutoImpute.py expects the pre-processed count matrix as input
- The goal is differential-expression testing rather than matrix completion prior to downstream clustering or variance analysis

## Inputs

- Preprocessed scRNA-seq count matrix (.csv or .mat), rows = cells, columns = genes
- Raw scRNA-seq count matrix (.csv) for R pre-processing via pre-process.R
- Matched bulk RNA-seq data (e.g., Trapnell GSE52529) for true-zero validation binning

## Outputs

- Imputed count matrix of the same dimensions as the input
- Per-bulk-bin zero-fraction table (e.g., true_zero_fractions.csv) for true-zero retention analysis

## How to apply

Preprocess the count matrix in R (gene filter: expressed in >=3 cells with >=3 reads; median library-size normalization; select top-1000 high-dispersion genes; log2 transform with pseudo-count 1), with columns as genes and rows as cells. Then run AutoImpute, an overcomplete autoencoder (hidden layer 2000 units, RMSProp optimizer, lambda=1 regularization, initial learning rate 1e-4, convergence threshold 1e-4), which learns the inherent distribution of the input data and imputes dropouts with minimal modification to biologically silent genes. To validate true-zero retention, bin genes by matched bulk expression (Trapnell GSE52529 myoblast replicates, TMM-normalized log2 bulk medians): the zero-expression bin is biologically silent, and per-bin zero fractions in the imputed matrix should stay high for silent genes while decreasing as bulk median expression increases. Compare against scImpute, drImpute, and MAGIC for conservativeness.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder model that imputes dropout zeros in the preprocessed scRNA-seq matrix) — https://github.com/divyanshu-talwar/AutoImpute
- **scImpute** (Comparison imputation method used to benchmark true-zero retention conservativeness)
- **drImpute** (Comparison imputation method used to benchmark true-zero retention conservativeness)
- **MAGIC** (Comparison imputation method used to benchmark true-zero retention conservativeness)

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- Biologically-silent bin (bulk median = 0) retains the highest fraction of zeros among all bins in the imputed matrix
- Imputed-zero fraction decreases monotonically (trend) as bulk median expression bin increases
- Imputed matrix has the same dimensions as the input preprocessed matrix (cells x genes)
- Convergence reached: change in loss between consecutive iterations < threshold 1e-4 (default max 7000 iterations)
- Masked-matrix recovery test (e.g., --masking_percentage 10) shows recovered values close to the masked ground truth

## Limitations

- Model is TensorFlow-based and written for Python 2.7; preprocessing R scripts require many packages (Matrix, irlba, mclust, dynamicTreeCut, etc.)
- Convergence relies on a loss-change threshold (1e-4) with a capped 7000 iterations; very large or noisy matrices may stop before full convergence
- Imputation modifies zeros only as dropouts if learned as such; overly aggressive methods (unlike AutoImpute) may overwrite true biologically silent zeros — verify against matched bulk data when available

## Evidence

- [abstract] overcomplete autoencoder design: "we developed an autoencoder-based sparse gene expression matrix imputation method. AutoImpute, which learns the inherent distribution of the input scRNA-seq data"
- [abstract] true-zero preservation claim: "imputes the missing values accordingly with minimal modification to the biologically silent genes"
- [readme] README execution command: "AutoImpute` model can be run using the following command and command line arguments :
bash
python AutoImpute.py"
- [readme] README preprocessing step: "To pre-process any dataset, place the raw data (in `.csv` format) into `AutoImpute\ Model/data/raw/` and change your directory to `AutoImpute\ Model/Pre-processing/`"
- [readme] README default parameters: "--hidden_units 2000	# Size of hidden layer or latent space dimensions."
