---
name: cross-method-imputation-benchmarking
description: Use when a new or existing scRNA-seq imputation method must be quantitatively
  compared against competing imputers by masking known non-zero values and measuring
  recovery error (RMSE/MAE/NMSE).
when_to_use_negative:
- The goal is downstream biological evaluation (clustering accuracy, variance stabilization,
  cell-type separability) rather than masked-value recovery error
- The baseline method's preprocessing changes matrix dimensions (e.g. drImpute gene
  filtering), making masked-location RMSE comparison invalid
- The input matrix is already incomplete with unknown ground truth — masked recovery
  requires a full matrix to mask and score against
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Imputation model under test; also generates the masked matrices via --masked_matrix_test
    and --masking_percentage
  repo: github.com/divyanshu-talwar/AutoImpute
- name: MAGIC
  role: Baseline imputer run on the same masked matrices for RMSE/MAE/NMSE comparison
- name: scImpute
  role: Baseline imputer run on the same masked matrices for RMSE/MAE/NMSE comparison
- name: TensorFlow
  role: Backend training the overcomplete autoencoder with RMSProp optimizer
- name: numpy
  role: Computing NMSE, RMSE, and MAE between recovered and original matrices at masked
    locations
- name: matplotlib
  role: Plotting RMSE-vs-masking-percentage curves (Fig. 3)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:a67619867d23347b073e5305a569d476867b199cc19b3cc366a2f1d2839e8354
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-method-imputation-benchmarking

## Summary

Benchmark an imputation method (AutoImpute overcomplete autoencoder) against baselines (MAGIC, scImpute) using an artificially masked-expression-value recovery experiment scored with NMSE, RMSE, and MAE on seven scRNA-seq datasets.

## When to use

Use when a new or existing scRNA-seq imputation method must be quantitatively compared against competing imputers by masking known non-zero values and measuring recovery error (RMSE/MAE/NMSE).

## When NOT to use

- The goal is downstream biological evaluation (clustering accuracy, variance stabilization, cell-type separability) rather than masked-value recovery error
- The baseline method's preprocessing changes matrix dimensions (e.g. drImpute gene filtering), making masked-location RMSE comparison invalid
- The input matrix is already incomplete with unknown ground truth — masked recovery requires a full matrix to mask and score against

## Inputs

- Pre-processed, log-transformed scRNA-seq count matrices (.mat/.csv) for Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel
- Per-dataset hyper-parameters from Table 2 (lambda, hidden units, learning rate, convergence threshold)
- Masking percentages: 10%, 20%, 30%, 40%, 50%

## Outputs

- recovery_metrics.csv with NMSE/RMSE/MAE (mean and variance over 10 runs) for AutoImpute, MAGIC, and scImpute at all five masking levels on all seven datasets
- RMSE-vs-masking-percentage curves per dataset (as in Fig. 3)

## How to apply

Load pre-processed, log-transformed count matrices for seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel; PBMC excluded due to size). For each dataset, randomly mask non-zero expression values at five levels (10-50%) using AutoImpute.py's --masked_matrix_test and --masking_percentage options, then run the overcomplete-autoencoder per dataset with Table 2 hyper-parameters (e.g. Blakeley: lambda=1, hidden_units=2000, initial_learning_rate=1e-4, threshold=1e-4, RMSProp, 7000 iterations max), repeating each run 10 times to average out random weight initialization. Run MAGIC and scImpute on the identical masked matrices; skip drImpute error computation because its gene filtering changes matrix dimensions. Compute NMSE, RMSE, and MAE between recovered and original values at the masked locations, aggregating mean and variance over the 10 runs (Supplementary Table S2, Fig. 3 RMSE-vs-masking curves).

## Related tools

- **AutoImpute** (Imputation model under test; also generates the masked matrices via --masked_matrix_test and --masking_percentage) — github.com/divyanshu-talwar/AutoImpute
- **MAGIC** (Baseline imputer run on the same masked matrices for RMSE/MAE/NMSE comparison)
- **scImpute** (Baseline imputer run on the same masked matrices for RMSE/MAE/NMSE comparison)
- **TensorFlow** (Backend training the overcomplete autoencoder with RMSProp optimizer)
- **numpy** (Computing NMSE, RMSE, and MAE between recovered and original matrices at masked locations)
- **matplotlib** (Plotting RMSE-vs-masking-percentage curves (Fig. 3))

## Examples

```
python AutoImpute.py --data blakeley.csv --masked_matrix_test True --masking_percentage 10 --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001 --iterations 7000
```

## Evaluation signals

- recovery_metrics.csv contains NMSE/RMSE/MAE (mean AND variance) for AutoImpute, MAGIC, and scImpute at all five masking levels on all seven datasets
- Qualitative acceptance: AutoImpute attains the lowest RMSE across the majority of the seven datasets, as reported in Fig. 3
- Each condition is aggregated over 10 independent runs to account for random weight initialization; per-run variance must be finite and recorded
- drImpute must be excluded from the error table (its gene filtering changes matrix dimensions, so metrics are not comparable)
- Imputed matrix dimensions match the input masked matrix (columns = genes, rows = cells), per the README pipeline

## Limitations

- drImpute cannot be scored in this benchmark because its gene-filtering preprocessing changes matrix dimensions relative to the masked ground truth
- PBMC dataset is excluded from this experiment due to its size
- Results depend on random weight initialization, hence the 10-repeat averaging; single runs are not stable estimates
- Masked-recovery RMSE only measures point-wise reconstruction of artificially masked non-zero values, not preservation of biological signal (e.g. biologically silent genes)

## Evidence

- [readme] CLI options for masked test and masking percentage: "--masked_matrix_test False	# Run the masked matrix recovery test?
		--masking_percentage 10	# Percentage of masking required."
- [readme] AutoImpute input/output format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [intro] Overcomplete autoencoder method statement: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [intro] Benchmark scope across datasets: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
