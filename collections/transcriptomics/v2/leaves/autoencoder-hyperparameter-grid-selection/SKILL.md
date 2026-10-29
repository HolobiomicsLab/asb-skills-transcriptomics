---
name: autoencoder-hyperparameter-grid-selection
description: Use when running AutoImpute's autoencoder imputation on a scRNA-seq count
  matrix and per-dataset hyper-parameters (lambda, hidden units, initial learning
  rate, threshold) must be chosen from Table 2-style settings.
when_to_use_negative:
- The input is raw unpre-processed count data in CSV form — it must first be run through
  the R-script pre-process.R before AutoImpute.py can consume it.
- The target tool is a matrix-factorization or nuclear-norm imputer rather than the
  TensorFlow overcomplete autoencoder — its hyper-parameters (lambda_val, hidden_units)
  do not exist there.
- Only a single one-shot imputation without repeated initialization is needed — the
  10-repeat aggregation is designed to control for random weight initialization and
  would be unnecessary overhead.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation model whose CLI exposes the hyper-parameter
    grid (lambda_val, hidden_units, initial_learning_rate, threshold, masking_percentage)
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Backend training the autoencoder with RMSProp gradient descent and checkpoint
    saving (checkpoints/model.ckpt)
- name: numpy
  role: Python dependency for matrix handling of expression matrices and metric computation
- name: scikit-learn
  role: Python dependency supporting metric computation on recovered vs. original
    matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:dc7d73febbbaffb7f1dabde9246061095a389841e12b5546340f9ac32106810c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - MAGIC
  - TensorFlow
  - matplotlib
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# autoencoder-hyperparameter-grid-selection

## Summary

Select per-dataset hyper-parameters (regularization coefficient lambda, hidden/latent layer size, learning rate, convergence threshold) for the AutoImpute overcomplete-autoencoder used to impute sparse scRNA-seq expression matrices, so that imputation quality is tuned per dataset rather than fixed globally.

## When to use

Use when running AutoImpute's autoencoder imputation on a scRNA-seq count matrix and per-dataset hyper-parameters (lambda, hidden units, initial learning rate, threshold) must be chosen from Table 2-style settings.

## When NOT to use

- The input is raw unpre-processed count data in CSV form — it must first be run through the R-script pre-process.R before AutoImpute.py can consume it.
- The target tool is a matrix-factorization or nuclear-norm imputer rather than the TensorFlow overcomplete autoencoder — its hyper-parameters (lambda_val, hidden_units) do not exist there.
- Only a single one-shot imputation without repeated initialization is needed — the 10-repeat aggregation is designed to control for random weight initialization and would be unnecessary overhead.

## Inputs

- Pre-processed, log-transformed scRNA-seq count matrix (.csv or .mat) with rows = cells and columns = genes (e.g. AutoImpute Model/data/<dataset_name>.{mat, csv})
- Masked version of the matrix generated with --masked_matrix_test True and --masking_percentage in {10, 20, 30, 40, 50}
- Candidate hyper-parameter grid (lambda_val, hidden_units, initial_learning_rate, threshold, iterations)

## Outputs

- Selected per-dataset hyper-parameter setting (Table 2-style: lambda, hidden units, learning rate, threshold)
- Imputed matrix (--imputed_save) and masked matrix (--masked_save) per configuration
- Recovery metrics (NMSE, RMSE, MAE — mean and variance over 10 runs) for ranking hyper-parameter settings

## How to apply

Load the pre-processed count matrix (columns = genes, rows = cells) as a .csv or .mat file and train the TensorFlow overcomplete autoencoder with RMSProp, exposing the tunable knobs via AutoImpute.py CLI flags: --lambda_val (regularization coefficient, e.g. 1 for Blakeley), --hidden_units (latent dimension, e.g. 2000), --initial_learning_rate (e.g. 0.0001), and --threshold (convergence stop on consecutive loss change, e.g. 0.0001), typically over --iterations 7000. For masked-matrix recovery evaluation, set --masked_matrix_test True with --masking_percentage (10/20/30/40/50) and repeat each configuration 10 times to account for random weight initialization, then aggregate NMSE/RMSE/MAE mean and variance across runs. Choose the setting whose repeated runs give the lowest reconstruction error at masked locations for that dataset, mirroring the per-dataset Table 2 hyper-parameters used for Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, and Zeisel.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation model whose CLI exposes the hyper-parameter grid (lambda_val, hidden_units, initial_learning_rate, threshold, masking_percentage)) — https://github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Backend training the autoencoder with RMSProp gradient descent and checkpoint saving (checkpoints/model.ckpt))
- **numpy** (Python dependency for matrix handling of expression matrices and metric computation)
- **scikit-learn** (Python dependency supporting metric computation on recovered vs. original matrices)

## Examples

```
python AutoImpute.py --data blakeley.csv --lambda_val 1 --hidden_units 2000 --initial_learning_rate 0.0001 --threshold 0.0001 --masked_matrix_test True --masking_percentage 10
```

## Evaluation signals

- recovery_metrics.csv contains NMSE/RMSE/MAE (mean and variance) for each hyper-parameter setting at all five masking levels (10-50%) on all seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel).
- Each configuration is run 10 times and both mean and variance of each metric are recorded, matching the Supplementary Table S2 aggregation scheme.
- The selected setting yields AutoImpute the lowest RMSE across datasets relative to MAGIC and scImpute baselines, as reported in Fig. 3.
- Training loss decreases until the --threshold criterion (change in consecutive loss values < 0.0001) triggers convergence, verifiable via --debug True logs / log.txt.
- The imputed matrix (--imputed_save) has exactly the same dimensions as the input matrix, since the model output preserves the cell x gene shape.

## Limitations

- drImpute cannot be compared on masked-recovery error because its gene-filtering preprocessing changes matrix dimensions relative to the original matrix.
- Results depend on random weight initialization, so single runs are unreliable; 10 repeats per condition are required.
- PBMC was excluded from the masked-recovery experiment due to its size, indicating scalability limits for very large datasets.
- The README is written for Python 2.7; other environments may need adaptation.

## Evidence

- [abstract] AutoImpute, which learns the inherent distribution of the input scRNA-seq data: "we developed an autoencoder-based sparse gene expression matrix imputation method. AutoImpute, which learns the inherent distribution of the input scRNA-seq data"
- [readme] README options block: "--lambda_val 1	# Regularization coefficient, to control the contribution of regularization term in the cost function."
- [readme] README convergence threshold: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold"
- [readme] README masking test flag: "--masked_matrix_test False	# Run the masked matrix recovery test?"
- [intro] autoencoder generalization: "Autoencoder based methods generalize better and are less prone to overfitting for a data restricted problem like ours"
