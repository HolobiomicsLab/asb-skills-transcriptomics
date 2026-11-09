---
name: single-cell-rnaseq-dropout-imputation
description: Use when a log2-transformed, library-size-normalized scRNA-seq count
  matrix (cells x top-1000 high-dispersion genes) contains dropout zeros that impair
  downstream clustering, expression recovery, or variance stabilization analyses.
when_to_use_negative:
- The expression matrix is already dense/imputed — further autoencoder imputation
  would distort true zeros in biologically silent genes
- Data have not been pre-processed (log2 transform, library-size normalization, top-1000
  high-dispersion gene selection) — AutoImpute.py expects a pre-processed count matrix
- The analysis goal is a very small dataset where overfitting by an overcomplete (2000-unit)
  hidden layer is a concern and the paper's matrix-factorization or nuclear-norm baselines
  may suffice
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: AutoImpute
  role: The three-layer overcomplete-autoencoder imputation model (AutoImpute.py)
    that learns the inherent distribution of the input scRNA-seq data and reconstructs
    the missing values
  repo: github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Implements the autoencoder's gradient descent training with back-propagation
    and the RMSProp optimizer
- name: numpy
  role: Loads and handles the preprocessed log2-transformed expression matrix (e.g.
    blakeley.csv) as NumPy arrays
- name: scikit-learn
  role: Supports downstream evaluation of imputation quality such as cell-clustering
    accuracy
- name: pre-process.R
  role: R pre-processing script (log2 transform, library-size normalization, top-1000
    high-dispersion gene selection) producing the input matrix format AutoImpute.py
    requires
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:c48e88f46b7c7dfdff14075cce1122039fc7c6c7d84fb08045b48b463ee9310f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - pre-process.R
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-rnaseq-dropout-imputation

## Summary

Impute dropouts in sparse single-cell RNA-seq expression matrices using AutoImpute, a three-layer fully-connected overcomplete autoencoder trained on observed (non-zero) entries only. Treats imputation as a collaborative-filtering/matrix-completion problem so missing values are filled with minimal modification to biologically silent genes.

## When to use

Use when a log2-transformed, library-size-normalized scRNA-seq count matrix (cells x top-1000 high-dispersion genes) contains dropout zeros that impair downstream clustering, expression recovery, or variance stabilization analyses.

## When NOT to use

- The expression matrix is already dense/imputed — further autoencoder imputation would distort true zeros in biologically silent genes
- Data have not been pre-processed (log2 transform, library-size normalization, top-1000 high-dispersion gene selection) — AutoImpute.py expects a pre-processed count matrix
- The analysis goal is a very small dataset where overfitting by an overcomplete (2000-unit) hidden layer is a concern and the paper's matrix-factorization or nuclear-norm baselines may suffice

## Inputs

- Pre-processed scRNA-seq count matrix CSV (.csv or .mat), rows = cells, columns = genes, log2-transformed and library-size normalized with top-1000 high-dispersion genes (e.g. blakeley.csv)
- Raw count matrix in .csv format placed in AutoImpute\ Model/data/raw/ for R pre-processing

## Outputs

- Imputed count matrix (imputed_matrix.csv) with identical dimensions (cells x genes) to the input matrix
- Trained model weights saved as checkpoints/model1.ckpt
- Training-loss log file (log.txt)

## How to apply

Pre-process the raw count matrix with the R script pre-process.R (log2 transform, library-size normalization, retain the top-1000 high-dispersion genes) and place the result in AutoImpute\ Model/data/. Run AutoImpute.py on the CSV (e.g. blakeley.csv), building a three-layer MLP autoencoder whose input/output node counts equal the number of genes and whose overcomplete hidden layer defaults to hidden_units=2000. Train with RMSProp (initial_learning_rate 0.0001 for Blakely; 1e-5 for the ~68,000-cell PBMC dataset) by back-propagating error only over the non-zero observed entries, minimizing the equation-8 cost: squared Frobenius loss on observed entries plus (lambda/2)*(||E||_F^2 + ||D||_F^2) with lambda_val=1. Stop when the change in consecutive loss values falls below threshold=0.0001 (up to iterations=7000), logging losses to log.txt and saving weights to checkpoints/model1.ckpt. Reconstruct the full matrix as X = D*sigma(E*R) and save it with the same cells x genes dimensions via --imputed_save.

## Related tools

- **AutoImpute** (The three-layer overcomplete-autoencoder imputation model (AutoImpute.py) that learns the inherent distribution of the input scRNA-seq data and reconstructs the missing values) — github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Implements the autoencoder's gradient descent training with back-propagation and the RMSProp optimizer)
- **numpy** (Loads and handles the preprocessed log2-transformed expression matrix (e.g. blakeley.csv) as NumPy arrays)
- **scikit-learn** (Supports downstream evaluation of imputation quality such as cell-clustering accuracy)
- **pre-process.R** (R pre-processing script (log2 transform, library-size normalization, top-1000 high-dispersion gene selection) producing the input matrix format AutoImpute.py requires) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --iterations 7000 --threshold 0.0001 --imputed_save imputed_matrix
```

## Evaluation signals

- Training loss decreases monotonically and stops when the change in loss between consecutive iterations falls below threshold=0.0001 (within iterations=7000)
- imputed_matrix.csv exists with the same dimensionality (cells x genes) as the input matrix — e.g. ~68,000 cells x 1000 genes for PBMC
- Masked-matrix recovery test (--masked_matrix_test True, --masking_percentage 10) shows observed entries are recovered accurately
- For the ~68,000-cell PBMC run, training completes without a memory-exhaustion error (AutoImpute's scalability advantage over baselines)
- Downstream K-means clustering ARI on the imputed matrix is not degraded versus no imputation (e.g. ARI 0.2850 vs 0.2687 for PBMC)

## Limitations

- Hyperparameters are dataset-dependent and were grid-searched per Table 2 (lambda_val, hidden_units, initial_learning_rate, threshold); defaults (lambda=1, 2000 hidden units, lr=0.0001) are tuned for blakeley.csv and must be adjusted for other datasets
- Convergence can take up to 7000 iterations; the run stops only when consecutive loss change < threshold=0.0001
- Other existing imputation methods exhausted system memory on the ~68,000-cell PBMC dataset, so results may not be comparable across all methods at scale
- Training back-propagates only on non-zero (observed) entries, so imputed values in very sparse regions rely entirely on the learned distribution

## Evidence

- [intro] In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders.: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [intro] AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values.: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [readme] The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [readme] --threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence.: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence."
- [readme] --lambda_val 1	# Regularization coefficient, to control the contribution of regularization term in the cost function.: "--lambda_val 1	# Regularization coefficient, to control the contribution of regularization term in the cost function."
