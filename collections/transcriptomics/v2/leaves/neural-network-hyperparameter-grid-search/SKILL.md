---
name: neural-network-hyperparameter-grid-search
description: Use when training an autoencoder-based imputation model (e.g., AutoImpute)
  on a sparse single-cell expression matrix and the cost-function regularization coefficient,
  hidden layer size, learning rate, or convergence threshold must be selected before
  final training.
when_to_use_negative:
- The data are not a sparse gene expression matrix (e.g., a dense bulk-RNA-seq or
  feature table), since AutoImpute's cost is computed only over observed non-zero
  dropout entries
- A hyperparameter sweep is not needed because a previously saved model checkpoint
  (checkpoints/model0.ckpt) with known-good settings is being loaded (--load_saved
  True)
- The task is raw-data preprocessing (quality filtering, gene dispersion selection)
  rather than model training, which is handled by the R pre-process.R pipeline instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoImpute
  role: Three-layer overcomplete-autoencoder imputation model whose CLI exposes the
    hyperparameter flags being grid-searched
  repo: github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Backend used to train the autoencoder with RMSProp and adjust the learning
    rate during the sweep
- name: numpy
  role: Loads and handles the preprocessed expression matrix passed to each grid candidate
- name: scikit-learn
  role: Python dependency of the AutoImpute pipeline used during model training/evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:640d521e1e108817e2c4d1fb31ae5942b262f007c56674a52b7b90110fa9827d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# neural-network-hyperparameter-grid-search

## Summary

Grid-search protocol for tuning AutoImpute's overcomplete-autoencoder hyperparameters (regularization coefficient lambda, hidden-layer size, learning rate, and convergence threshold) to impute sparse scRNA-seq expression matrices.

## When to use

Use when training an autoencoder-based imputation model (e.g., AutoImpute) on a sparse single-cell expression matrix and the cost-function regularization coefficient, hidden layer size, learning rate, or convergence threshold must be selected before final training.

## When NOT to use

- The data are not a sparse gene expression matrix (e.g., a dense bulk-RNA-seq or feature table), since AutoImpute's cost is computed only over observed non-zero dropout entries
- A hyperparameter sweep is not needed because a previously saved model checkpoint (checkpoints/model0.ckpt) with known-good settings is being loaded (--load_saved True)
- The task is raw-data preprocessing (quality filtering, gene dispersion selection) rather than model training, which is handled by the R pre-process.R pipeline instead

## Inputs

- Preprocessed log2-transformed scRNA-seq count matrix (e.g., blakeley.csv) with rows as cells and columns as top-1000 high-dispersion genes
- Grid specification of lambda_val, hidden_units, initial_learning_rate, and threshold values (per Table 2 of the paper)

## Outputs

- Selected hyperparameter configuration (lambda, hidden size, learning rate, threshold)
- Per-iteration training loss trace in log.txt used to compare grid candidates
- Trained encoder/decoder weights saved to checkpoints/model1.ckpt for the winning configuration

## How to apply

Build the three-layer fully-connected MLP autoencoder whose input/output node counts equal the number of top-1000 high-dispersion genes and whose hidden layer is overcomplete (default hidden_units=2000). Sweep the grid parameters named in the paper's Table 2 via the CLI flags: lambda_val (default 1) controlling the (lambda/2)*(||E||_F^2 + ||D||_F^2) regularizer, hidden_units, initial_learning_rate (default 0.0001) consumed by the RMSProp optimizer, and threshold (default 0.0001) for the stopping rule on the change in equation-8 cost between consecutive iterations (up to iterations=7000). Train each configuration by back-propagation on the squared Frobenius loss computed only over observed non-zero entries of the sparse matrix, logging per-iteration loss to log.txt. Select the configuration whose training loss decreases monotonically and converges below the threshold, then re-run the full forward pass to emit the imputed matrix.

## Related tools

- **AutoImpute** (Three-layer overcomplete-autoencoder imputation model whose CLI exposes the hyperparameter flags being grid-searched) — github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Backend used to train the autoencoder with RMSProp and adjust the learning rate during the sweep)
- **numpy** (Loads and handles the preprocessed expression matrix passed to each grid candidate)
- **scikit-learn** (Python dependency of the AutoImpute pipeline used during model training/evaluation)

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --iterations 7000 --threshold 0.0001 --debug True --log_file log.txt
```

## Evaluation signals

- For each grid candidate, the equation-8 cost decreases monotonically and stops only when the change in loss between consecutive iterations falls below threshold (default 0.0001) before iterations=7000
- Loss values logged to log.txt at debug_display_step intervals can be compared across lambda_val, hidden_units, and initial_learning_rate settings to rank candidates
- The winning configuration's imputed_matrix output must have the same dimensionality (cells x genes) as the input sparse matrix
- Regularization contribution scales as (lambda/2)*(||E||_F^2 + ||D||_F^2) with lambda_val=1 by default; changing lambda must visibly alter the cost trace
- Masked-matrix recovery test (--masked_matrix_test True with --masking_percentage 10) can validate imputation quality of a chosen configuration

## Limitations

- Defaults (hidden_units=2000, lambda_val=1, initial_learning_rate=0.0001, iterations=7000, threshold=0.0001) were tuned for the paper's datasets and may not transfer to matrices with different cell/gene counts
- Grid search requires retraining per candidate (up to 7000 RMSProp iterations each), which is compute-intensive
- Convergence is judged solely by the change in loss between consecutive iterations; a plateau above the threshold still triggers the stopping rule

## Evidence

- [intro] Propose AutoImpute, a novel method for sparse gene expression matrix imputation using overcomplete autoencoders: [evidence span withheld — anchor status: not_found]
- [readme] README: hyperparameter flags: "--hidden_units 2000	# Size of hidden layer or latent space dimensions.
		--lambda_val 1	# Regularization coefficient"
- [readme] README: convergence threshold: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold"
- [readme] README: learning rate and iterations: "--initial_learning_rate 0.0001	# Initial value of learning rate.
		--iterations 7000	# Number of iterations to train the model for."
