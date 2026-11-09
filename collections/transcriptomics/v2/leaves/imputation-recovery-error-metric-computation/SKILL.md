---
name: imputation-recovery-error-metric-computation
description: 'Use when benchmarking an imputation method on scRNA-seq count matrices:
  you need controlled dropout recovery curves (error vs masking percentage) comparable
  to Fig. 3 of the AutoImpute paper.'
when_to_use_negative:
- The imputed matrix has different gene dimensions than the original (e.g. drImpute's
  gene-filtering preprocessing), making position-wise masked-recovery error undefined
- You need biological downstream evaluation (clustering accuracy, cell-type separability)
  rather than pure expression-recovery error
- The expression matrix is not pre-processed/log-transformed and cells/genes orientation
  does not match AutoImpute.py's expected input
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation model; generates masked matrices (--masked_matrix_test,
    --masking_percentage) and imputed outputs for the recovery test
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: MAGIC
  role: Baseline imputer run on the same masked matrices for comparison
- name: scImpute
  role: Baseline imputer run on the same masked matrices for comparison
- name: numpy
  role: Masking of non-zero values and metric computation (NMSE, RMSE, MAE) at masked
    locations
- name: TensorFlow
  role: Backend for training the AutoImpute autoencoder with RMSProp
- name: matplotlib
  role: Plotting RMSE-vs-masking-percentage curves (Fig. 3 reproduction)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:420001c248ce06f10b1d6fda97b5185b03e58e1dee6c3333253524ba95413cb3
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
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# imputation-recovery-error-metric-computation

## Summary

Quantifies scRNA-seq imputation quality by artificially masking non-zero expression values (10–50%), running imputers on the incomplete matrices, and computing NMSE, RMSE, and MAE between recovered and original values at masked locations.

## When to use

Use when benchmarking an imputation method on scRNA-seq count matrices: you need controlled dropout recovery curves (error vs masking percentage) comparable to Fig. 3 of the AutoImpute paper.

## When NOT to use

- The imputed matrix has different gene dimensions than the original (e.g. drImpute's gene-filtering preprocessing), making position-wise masked-recovery error undefined
- You need biological downstream evaluation (clustering accuracy, cell-type separability) rather than pure expression-recovery error
- The expression matrix is not pre-processed/log-transformed and cells/genes orientation does not match AutoImpute.py's expected input

## Inputs

- Pre-processed, log-transformed scRNA-seq count matrices (.mat or .csv, genes as columns, cells as rows) for seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel)
- Artificially masked matrices at 10–50% masking of non-zero values
- Imputed matrices from AutoImpute, MAGIC, and scImpute on the same masked inputs

## Outputs

- recovery_metrics.csv with NMSE/RMSE/MAE (mean and variance over 10 runs) per method per masking level per dataset
- RMSE-vs-masking-percentage trend curves reproducing Fig. 3

## How to apply

Load pre-processed, log-transformed count matrices (columns=genes, rows=cells) and, per dataset, randomly mute non-zero expression values at 10/20/30/40/50% using the --masked_matrix_test and --masking_percentage options of AutoImpute.py. Run the overcomplete-autoencoder (e.g. Blakeley: lambda=1, hidden units=2000, initial learning rate 1e-4, threshold 1e-4, RMSProp, 7000 iterations) plus baselines MAGIC and scImpute on each masked matrix, repeating 10 times per condition to average over random weight initialization. Compute NMSE, RMSE, and MAE between recovered and original full matrices restricted to masked positions; skip drImpute error computation because its gene-filtering preprocessing changes matrix dimensions. Aggregate mean and variance of each metric per condition (as in Supplementary Table S2) and plot RMSE vs masking percentage. Acceptance: AutoImpute attains the lowest RMSE across the seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel) as in Fig. 3.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation model; generates masked matrices (--masked_matrix_test, --masking_percentage) and imputed outputs for the recovery test) — https://github.com/divyanshu-talwar/AutoImpute
- **MAGIC** (Baseline imputer run on the same masked matrices for comparison)
- **scImpute** (Baseline imputer run on the same masked matrices for comparison)
- **numpy** (Masking of non-zero values and metric computation (NMSE, RMSE, MAE) at masked locations)
- **TensorFlow** (Backend for training the AutoImpute autoencoder with RMSProp)
- **matplotlib** (Plotting RMSE-vs-masking-percentage curves (Fig. 3 reproduction))

## Examples

```
python AutoImpute.py --data blakeley.csv --masked_matrix_test True --masking_percentage 10 --lambda_val 1 --hidden_units 2000 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- recovery_metrics.csv contains NMSE, RMSE, and MAE (mean and variance) for AutoImpute, MAGIC, and scImpute at all five masking levels (10–50%) on all seven datasets
- Each metric is aggregated over 10 repeats per condition to account for random weight initialization
- Metrics are computed only at masked locations between the recovered and original full expression matrix
- Qualitative check: AutoImpute attains the lowest RMSE across datasets, matching the trends of Fig. 3

## Limitations

- drImpute results cannot be included in the error comparison because its gene-filtering preprocessing changes matrix dimensions
- PBMC is excluded from the seven-dataset recovery experiment due to its size
- Results depend on random weight initialization, hence the requirement of 10 repeated runs per condition

## Evidence

- [readme] masked matrix recovery test: "--masked_matrix_test False	# Run the masked matrix recovery test?"
- [readme] masking percentage option: "--masking_percentage 10	# Percentage of masking required. Like 10, 20, 12.5 etc."
- [readme] input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [intro] ten independent datasets evaluated: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
- [abstract] competitive performance finding: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
