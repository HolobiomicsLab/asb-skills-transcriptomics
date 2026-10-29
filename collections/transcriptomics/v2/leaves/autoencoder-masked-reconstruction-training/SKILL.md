---
name: autoencoder-masked-reconstruction-training
description: Use when a preprocessed sparse single-cell gene expression matrix (cells
  x top high-dispersion genes, .csv) contains dropout dropouts that must be imputed
  via overcomplete autoencoder reconstruction, e.g. blakeley.csv.
when_to_use_negative:
- The matrix is already dense / free of dropouts — no imputation is needed
- Only raw unprocessed counts are available and R pre-processing (top-1000 high-dispersion
  gene selection, log2 transform) via pre-process.R has not been done
- The goal is de novo gene discovery or differential expression rather than dropout
  imputation of an existing partially observed matrix
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Core TensorFlow-based overcomplete autoencoder imputation model (AutoImpute.py)
    invoked from the command line
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Loads the preprocessed .csv expression matrix used as network input
- name: TensorFlow
  role: Builds the three-layer MLP autoencoder, computes the equation-8 cost, and
    runs RMSProp optimization
- name: scikit-learn
  role: Python dependency listed for the AutoImpute pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:837a9e582f1547ea02eb861f99680b313360e554eb331711646b72636ec93586
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# autoencoder-masked-reconstruction-training

## Summary

Train AutoImpute, a three-layer fully-connected overcomplete autoencoder (TensorFlow MLP), on a sparse log2-transformed scRNA-seq matrix to reconstruct the complete gene expression matrix, treating dropout imputation as a matrix-completion problem.

## When to use

Use when a preprocessed sparse single-cell gene expression matrix (cells x top high-dispersion genes, .csv) contains dropout dropouts that must be imputed via overcomplete autoencoder reconstruction, e.g. blakeley.csv.

## When NOT to use

- The matrix is already dense / free of dropouts — no imputation is needed
- Only raw unprocessed counts are available and R pre-processing (top-1000 high-dispersion gene selection, log2 transform) via pre-process.R has not been done
- The goal is de novo gene discovery or differential expression rather than dropout imputation of an existing partially observed matrix

## Inputs

- Preprocessed log2-transformed sparse single-cell count matrix as .csv (e.g. blakeley.csv), rows = cells, columns = top-1000 high-dispersion genes
- Optionally a .mat preprocessed matrix produced by pre-process.R

## Outputs

- Imputed dense expression matrix with the same dimensions as the input (imputed_matrix.csv)
- Learned autoencoder weights checkpoint (checkpoints/model1.ckpt)
- Training loss log (log.txt) with per-iteration loss values

## How to apply

Load the preprocessed log2-transformed expression matrix (e.g. blakeley.csv) with numpy, rows as cells and columns as genes. Build a three-layer fully-connected MLP autoencoder (input/hidden/output) whose input and output node counts equal the number of genes, with hidden_units default 2000 so the latent layer is overcomplete. Train with back-propagation on the equation-8 cost — squared Frobenius loss computed only over observed non-zero entries plus (lambda/2)*(||E||_F^2 + ||D||_F^2) with lambda_val 1 — using the RMSProp optimizer with initial_learning_rate 0.0001, initializing E and D from random normal distributions. Stop when the change in loss between consecutive iterations falls below threshold 0.0001 (up to iterations 7000); optionally grid-search lambda, hidden size, learning rate and threshold per Table 2. Run the trained encoder-decoder forward pass to reconstruct the full matrix and save it as imputed_matrix.csv plus model weights to checkpoints/model1.ckpt.

## Related tools

- **AutoImpute** (Core TensorFlow-based overcomplete autoencoder imputation model (AutoImpute.py) invoked from the command line) — https://github.com/divyanshu-talwar/AutoImpute
- **numpy** (Loads the preprocessed .csv expression matrix used as network input)
- **TensorFlow** (Builds the three-layer MLP autoencoder, computes the equation-8 cost, and runs RMSProp optimization)
- **scikit-learn** (Python dependency listed for the AutoImpute pipeline)

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --iterations 7000 --threshold 0.0001 --imputed_save imputed_matrix --save_model_location checkpoints/model1.ckpt
```

## Evaluation signals

- Training loss decreases monotonically and training stops only when the change in loss between consecutive iterations falls below threshold 0.0001
- The saved imputed_matrix.csv has the same dimensionality (cells x genes) as the input sparse matrix
- With --masked_matrix_test True, expression recovery from the masked matrix at the chosen masking_percentage is validated against known held-out entries
- Imputed values on biologically silent genes remain minimally modified relative to the original near-zero values
- Model weights checkpoint (checkpoints/model1.ckpt) is written and can be reloaded via --load_model_location

## Limitations

- Hyperparameters (lambda_val, hidden_units, initial_learning_rate, threshold) default to specific values (1, 2000, 0.0001, 0.0001) tuned on the blakeley dataset; other datasets may require the Table 2 grid search
- Autoencoder-based methods generalize better than matrix factorization or nuclear norm minimization only for this data-restricted problem; overfitting risk depends on regularization
- AutoImpute is implemented for Python 2.7 with numpy, scikit-learn, tensorflow and matplotlib; R pre-processing is a required upstream step for datasets other than blakeley.csv

## Evidence

- [intro] Propose AutoImpute, a novel method for sparse gene expression matrix imputation using overcomplete autoencoders: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [readme] README options list with defaults: "--hidden_units 2000	# Size of hidden layer or latent space dimensions. --lambda_val 1	# Regularization coefficient"
- [readme] README convergence threshold option: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold"
- [readme] README learning rate and iterations options: "--initial_learning_rate 0.0001	# Initial value of learning rate. --iterations 7000	# Number of iterations to train the model for."
