---
name: autoencoder-matrix-completion
description: Use when a log2-transformed, library-size-normalized scRNA-seq count
  matrix (e.g. top 1000 high-dispersion genes, .csv/.mat) contains dropout zeros and
  needs imputation before clustering or expression-recovery analysis.
when_to_use_negative:
- The matrix has not been R-preprocessed (raw .csv not yet log2-transformed, library-size
  normalized, and reduced to high-dispersion genes) — run Rscript pre-process.R first
- The goal is masked-matrix recovery benchmarking rather than genuine dropout imputation
  — set --masked_matrix_test True only for that evaluation mode
- The input is already dense/imputed, so training an autoencoder to complete it adds
  no information
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Autoencoder-based sparse gene expression matrix imputation model executed
    via AutoImpute.py
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Framework for the three-layer autoencoder trained by gradient descent with
    back-propagation and the RMSProp optimizer
- name: numpy
  role: Array handling for loading the pre-processed count matrix and writing the
    imputed matrix
- name: scikit-learn
  role: Downstream clustering evaluation (e.g. K-means ARI) of imputed versus non-imputed
    data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:5367827bd9804b366e264992f3d01dbebb46288b162bafb93597071a1aa1f910
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# autoencoder-matrix-completion

## Summary

Imputes dropout events in sparse single-cell RNA-seq count matrices using AutoImpute, a three-layer overcomplete autoencoder (TensorFlow) trained by gradient descent, framing the task as a collaborative-filtering-style matrix completion problem.

## When to use

Use when a log2-transformed, library-size-normalized scRNA-seq count matrix (e.g. top 1000 high-dispersion genes, .csv/.mat) contains dropout zeros and needs imputation before clustering or expression-recovery analysis.

## When NOT to use

- The matrix has not been R-preprocessed (raw .csv not yet log2-transformed, library-size normalized, and reduced to high-dispersion genes) — run Rscript pre-process.R first
- The goal is masked-matrix recovery benchmarking rather than genuine dropout imputation — set --masked_matrix_test True only for that evaluation mode
- The input is already dense/imputed, so training an autoencoder to complete it adds no information

## Inputs

- Pre-processed single-cell count matrix (.csv or .mat), rows = cells, columns = genes, log2-transformed and library-size normalized, e.g. top 1000 high-dispersion genes from the 10x Genomics PBMC dataset (~68,000 cells)

## Outputs

- Imputed count matrix (imputed_matrix.csv) with the same number of cells and genes as the input
- Training-loss log (log.txt) when run with --debug True
- Model checkpoint (checkpoints/model*.ckpt) of the learned autoencoder

## How to apply

Pre-process the raw count matrix in R (log2-transform after library-size normalization, select top 1000 high-dispersion genes) so rows are cells and columns are genes. Run AutoImpute.py with the Table 2 hyperparameters — lambda_val=1 (regularization coefficient), hidden_units=2000 (latent space size), initial_learning_rate=1e-5, threshold=1e-3 as the convergence criterion — training up to 7000 iterations with the RMSProp optimizer. Back-propagate error only on the non-zero counts of the sparse matrix, so training loss and updates are driven by observed values while zeros are treated as missing dropouts. Reconstruct the imputed expression matrix as X = D*sigma(E*R) and save it with the same cell-by-gene dimensions as the input via --imputed_save. Because the autoencoder generalizes without overfitting on this data-restricted problem, this method scales to ~68,000-cell datasets (e.g. the 10x PBMC matrix) where other imputation methods exhaust system memory.

## Related tools

- **AutoImpute** (Autoencoder-based sparse gene expression matrix imputation model executed via AutoImpute.py) — https://github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Framework for the three-layer autoencoder trained by gradient descent with back-propagation and the RMSProp optimizer)
- **numpy** (Array handling for loading the pre-processed count matrix and writing the imputed matrix)
- **scikit-learn** (Downstream clustering evaluation (e.g. K-means ARI) of imputed versus non-imputed data)

## Examples

```
python AutoImpute.py --data pbmc.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.00001 --iterations 7000 --threshold 0.001 --imputed_save imputed_matrix
```

## Evaluation signals

- imputed_matrix.csv exists with the same cell (~68,000 for PBMC) and gene (1000) counts as the input matrix
- Training loss in log.txt decreases and the run converges (change in loss below threshold=1e-3) or stops at the 7000-iteration cap
- The run completes without a memory-exhaustion error, matching the paper's scalability claim for PBMC where other imputation methods failed
- Downstream clustering improves relative to no imputation (K-means ARI 0.2850 imputed vs 0.2687 non-imputed on PBMC)
- Biologically silent genes show minimal modification after imputation, per the paper's design goal

## Limitations

- Python 2.7 is required per the README dependencies (numpy, scikit-learn, tensorflow, matplotlib), which may conflict with modern environments
- Other imputation methods exhaust system memory on the ~68,000-cell PBMC dataset; even AutoImpute's memory behavior should be monitored during training
- Non-default datasets must be pre-processed with the R script (Rscript pre-process.R <input-file-name> <dataset-name>) before AutoImpute can consume them
- The paper's extracted content lacks discussion-section material, so caveats beyond those stated in abstract/intro/README are not documented

## Evidence

- [readme] input is pre-processed count matrix: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [intro] overcomplete autoencoder proposal: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [intro] collaborative filtering framing: "The problem is akin to that of collaborative filtering; therein the ratings matrix is partially observed and the goal is to estimate the complete matrix given the partially observed ones."
- [abstract] minimal modification to silent genes: "imputes the missing values accordingly with minimal modification to the biologically silent genes"
- [readme] hidden units parameter: "--hidden_units 2000	# Size of hidden layer or latent space dimensions."
