---
name: memory-constrained-large-dataset-training
description: Use when an scRNA-seq count matrix is too large (~68,000 cells) for standard
  imputation tools and the goal is to run autoencoder-based imputation within available
  RAM, per the AutoImpute PBMC scalability protocol.
when_to_use_negative:
- When the dataset is small enough that any existing imputation method fits in memory
  — the scalability claim only distinguishes AutoImpute on the ~68,000-cell PBMC matrix
  where other methods exhausted memory
- When the input has not been pre-processed (raw counts, all genes) — README requires
  running Rscript pre-process.R on raw .csv data first
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Autoencoder imputation model trained on the large PBMC matrix via AutoImpute.py
  repo: github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Backend for the three-layer autoencoder trained with RMSProp and sparse back-propagation
    on non-zero counts
- name: numpy
  role: Numerical handling of the pre-processed expression matrix and imputed output
- name: scikit-learn
  role: Downstream K-means clustering used to check imputation quality (ARI 0.2850
    vs 0.2687 without imputation)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:652d1a1914e36b17b5bb9537aa047cf95812bfab9f0a9b9a670f04760f348f19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# memory-constrained-large-dataset-training

## Summary

Trains the AutoImpute three-layer TensorFlow autoencoder on the largest tested dataset (~68,000-cell 10x Genomics PBMC matrix, top 1000 high-dispersion genes) without exhausting system memory, exploiting sparse-matrix back-propagation on non-zero counts only.

## When to use

Use when an scRNA-seq count matrix is too large (~68,000 cells) for standard imputation tools and the goal is to run autoencoder-based imputation within available RAM, per the AutoImpute PBMC scalability protocol.

## When NOT to use

- When the dataset is small enough that any existing imputation method fits in memory — the scalability claim only distinguishes AutoImpute on the ~68,000-cell PBMC matrix where other methods exhausted memory
- When the input has not been pre-processed (raw counts, all genes) — README requires running Rscript pre-process.R on raw .csv data first

## Inputs

- Pre-processed PBMC count matrix in .csv or .mat format (~68,000 cells x 1000 genes), log2-transformed and library-size normalized with the top 1000 high-dispersion genes
- PBMC hyperparameters: lambda_val=1, hidden_units=2000, initial_learning_rate=1e-5, threshold=1e-3, 7000 iterations

## Outputs

- imputed_matrix (via --imputed_save) with the same dimensions as the input (~68,000 cells x 1000 genes)
- Training-loss log (log.txt) and TensorFlow model checkpoints (checkpoints/*.ckpt)

## How to apply

Start from the pre-processed PBMC matrix (log2-transformed, library-size normalized, top 1000 high-dispersion genes) produced by the R pre-process.R script, with columns as genes and rows as cells. Launch AutoImpute.py with the Table 2 PBMC hyperparameters — lambda_val=1, hidden_units=2000, initial_learning_rate=1e-5, threshold=1e-3 — training up to 7000 iterations with the RMSProp optimizer. Back-propagate error only on the non-zero counts of the sparse matrix so the memory footprint stays bounded, while writing loss logs to log.txt and checkpoints to checkpoints/. Monitor memory throughout the run; success means the run completes without memory exhaustion, unlike the competing imputation methods that failed on PBMC. Reconstruct the imputed matrix as X = D*sigma(E*R) and save it via --imputed_save with the same dimensions as the input.

## Related tools

- **AutoImpute** (Autoencoder imputation model trained on the large PBMC matrix via AutoImpute.py) — github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Backend for the three-layer autoencoder trained with RMSProp and sparse back-propagation on non-zero counts)
- **numpy** (Numerical handling of the pre-processed expression matrix and imputed output)
- **scikit-learn** (Downstream K-means clustering used to check imputation quality (ARI 0.2850 vs 0.2687 without imputation))

## Examples

```
python AutoImpute.py --data pbmc.csv --lambda_val 1 --hidden_units 2000 --initial_learning_rate 0.00001 --threshold 0.001 --iterations 7000 --imputed_save imputed_matrix
```

## Evaluation signals

- Run completes without a memory-exhaustion error on the ~68,000-cell PBMC matrix, matching the paper's scalability claim
- imputed_matrix.csv exists with the same dimensions as the input: ~68,000 cells and 1000 genes
- log.txt shows training-loss values converging, with gradient descent stopping once consecutive-iteration loss change falls below the threshold
- K-means clustering on the imputed matrix gives ARI 0.2850, above the 0.2687 obtained without imputation
- Model checkpoints written to the checkpoints directory, confirming training completed and was saved

## Limitations

- AutoImpute was written for Python 2.7 with the TensorFlow of that era; the README's options and defaults (e.g. initial_learning_rate 0.0001) must be overridden with the PBMC Table 2 values explicitly
- Non-AutoImpute imputation methods exhaust system memory on PBMC, so cross-method comparisons at this scale are not feasible on the same machine
- Input must be pre-processed (log2-transformed, normalized, top 1000 high-dispersion genes); running on raw counts is outside the validated pipeline

## Evidence

- [other] AutoImpute successfully imputed the PBMC dataset (K-means ARI 0.2850 vs 0.2687 without imputation), whereas each of the other existing imputation methods exhausted the system memory when applied to PBMC.: [evidence span withheld — anchor status: not_found]
- [readme] The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [readme] --threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence.: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence."
- [intro] In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders.: "we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders"
