---
name: scrnaseq-dropout-masking-simulation
description: Use when benchmarking an scRNA-seq imputation method on controlled dropout
  recovery — i.e., you have complete pre-processed count matrices and need to quantify
  NMSE/RMSE/MAE recovery at artificially masked non-zero values.
when_to_use_negative:
- When the goal is biological downstream analysis (clustering, cell-type separability)
  rather than controlled reconstruction-error benchmarking
- When comparing against drImpute with error metrics — its gene-filtering preprocessing
  changes matrix dimensions, making RMSE/MAE/NMSE at masked locations incomparable
- When input matrices are not pre-processed/log-transformed or lack a complete reference
  matrix to score masked entries against
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation model; its --masked_matrix_test and --masking_percentage
    options create the masked matrices and produce the imputed/recovered matrices
  repo: github.com/divyanshu-talwar/AutoImpute
- name: MAGIC
  role: Baseline imputation method run on the same masked matrices for recovery-error
    comparison
- name: scImpute
  role: Baseline imputation method run on the same masked matrices for recovery-error
    comparison
- name: TensorFlow
  role: Backend used to train the AutoImpute autoencoder with the RMSProp optimizer
- name: numpy
  role: Python dependency for masked-matrix construction and metric computation (NMSE,
    RMSE, MAE)
- name: matplotlib
  role: Python dependency used to plot RMSE-vs-masking-percentage recovery curves
    (Fig. 3)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:446e004056905c940618154a01f0827cd9f3ee73a4b570fcc33409f6c322ac2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - MAGIC
  - TensorFlow
  - matplotlib
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrnaseq-dropout-masking-simulation

## Summary

A masked-matrix recovery benchmark for scRNA-seq imputation: artificially mute 10–50% of non-zero expression values in pre-processed log-transformed count matrices, impute with AutoImpute and baselines (MAGIC, scImpute), and score recovery with NMSE, RMSE, and MAE at the masked locations.

## When to use

Use when benchmarking an scRNA-seq imputation method on controlled dropout recovery — i.e., you have complete pre-processed count matrices and need to quantify NMSE/RMSE/MAE recovery at artificially masked non-zero values.

## When NOT to use

- When the goal is biological downstream analysis (clustering, cell-type separability) rather than controlled reconstruction-error benchmarking
- When comparing against drImpute with error metrics — its gene-filtering preprocessing changes matrix dimensions, making RMSE/MAE/NMSE at masked locations incomparable
- When input matrices are not pre-processed/log-transformed or lack a complete reference matrix to score masked entries against

## Inputs

- Pre-processed, log-transformed scRNA-seq count matrices (columns = genes, rows = cells) in .mat/.csv format for seven datasets: Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel
- Masking percentage levels (10, 20, 30, 40, 50) applied to non-zero expression values
- Per-dataset AutoImpute hyper-parameters from Table 2 (lambda, hidden units, initial learning rate, convergence threshold)

## Outputs

- Masked (incomplete) expression matrices per dataset and masking level
- Imputed/recovered expression matrices from AutoImpute, MAGIC, and scImpute
- recovery_metrics.csv containing NMSE/RMSE/MAE (mean and variance) per method, dataset, and masking level (Supplementary Table S2 analogue; Fig. 3 RMSE curves)

## How to apply

Load the pre-processed, log-transformed count matrices for seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel; PBMC excluded due to size) from the package data directory. For each dataset, randomly mute non-zero expression values at five levels (10%, 20%, 30%, 40%, 50%) using the --masked_matrix_test and --masking_percentage options of AutoImpute.py. Run the AutoImpute overcomplete-autoencoder per masked matrix with per-dataset hyper-parameters from Table 2 (e.g., Blakeley: lambda=1, hidden_units=2000, initial_learning_rate=1e-4, threshold=1e-4), trained with RMSProp, repeating each run 10 times to account for random weight initialization. Run MAGIC and scImpute on the same masked matrices; skip drImpute error computation since its gene-filtering preprocessing changes matrix dimensions. Compute NMSE, RMSE, and MAE between recovered and original matrices at each masked location, aggregating mean and variance over the 10 runs as in Supplementary Table S2 and the RMSE-vs-masking-percentage curves of Fig. 3.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation model; its --masked_matrix_test and --masking_percentage options create the masked matrices and produce the imputed/recovered matrices) — github.com/divyanshu-talwar/AutoImpute
- **MAGIC** (Baseline imputation method run on the same masked matrices for recovery-error comparison)
- **scImpute** (Baseline imputation method run on the same masked matrices for recovery-error comparison)
- **TensorFlow** (Backend used to train the AutoImpute autoencoder with the RMSProp optimizer)
- **numpy** (Python dependency for masked-matrix construction and metric computation (NMSE, RMSE, MAE))
- **matplotlib** (Python dependency used to plot RMSE-vs-masking-percentage recovery curves (Fig. 3))

## Examples

```
python AutoImpute.py --data blakeley.csv --masked_matrix_test True --masking_percentage 10 --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- recovery_metrics.csv contains NMSE/RMSE/MAE (mean and variance) for AutoImpute, MAGIC, and scImpute at all five masking levels (10–50%) on all seven datasets
- AutoImpute attains the lowest RMSE across the majority of the seven datasets, matching the qualitative trend of Fig. 3
- Each (dataset, masking level, method) condition is aggregated over 10 repeated runs to account for random weight initialization, with both mean and variance recorded
- Masked matrices preserve original dimensions (same genes × cells) so recovery error is computed only at the masked locations against the original full matrix
- Training converges (loss change between consecutive iterations below threshold=1e-4) or stops at the 7000-iteration cap

## Limitations

- PBMC dataset is excluded from the masked-recovery experiment due to its size
- drImpute cannot be scored in this experiment because its gene-filtering preprocessing changes matrix dimensions, breaking position-wise error comparison
- Results depend on random weight initialization, hence the need for 10 repeated runs per condition

## Evidence

- [full_text] For each dataset, randomly mute (mask) non-zero expression values at five levels (10%, 20%, 30%, 40%, 50%): [evidence span withheld — anchor status: not_found]
- [readme] --masked_matrix_test and --masking_percentage options exist in AutoImpute.py: "--masked_matrix_test False	# Run the masked matrix recovery test?
		--masking_percentage 10	# Percentage of masking required."
- [readme] Overcomplete autoencoder with configurable hidden units, lambda, learning rate, and threshold: "--hidden_units 2000	# Size of hidden layer or latent space dimensions.
		--lambda_val 1	# Regularization coefficient"
- [abstract] AutoImpute learns the inherent distribution and imputes with minimal modification to silent genes: "AutoImpute, which learns the inherent distribution of the input scRNA-seq data"
- [full_text] Ten repeats account for random weight initialization: [evidence span withheld — anchor status: not_found]
