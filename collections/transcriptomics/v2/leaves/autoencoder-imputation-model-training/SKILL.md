---
name: autoencoder-imputation-model-training
description: Use when a pre-processed, log-transformed single-cell count matrix (cells
  x genes, CSV or .mat) has dropouts or artificially masked non-zero values that need
  imputation with an overcomplete autoencoder, or when benchmarking imputation recovery
  error.
when_to_use_negative:
- The matrix has not been pre-processed/log-transformed — run pre-process.R first
  rather than feeding raw counts to AutoImpute.py
- The baseline imputer under comparison is drImpute in a masked-recovery benchmark,
  since its gene-filtering preprocessing changes matrix dimensions and makes error
  computation invalid
- The dataset is very large (e.g. PBMC was excluded from the masked-recovery test
  due to its size)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation model executed via AutoImpute.py; performs
    training, masking, and imputed-matrix output
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Backend used to train the autoencoder with the RMSProp optimizer
- name: numpy
  role: Python dependency for matrix handling during training and metric computation
- name: scikit-learn
  role: Python dependency for the AutoImpute pipeline
- name: matplotlib
  role: Python dependency used for plotting (e.g. RMSE-vs-masking-percentage curves)
- name: MAGIC
  role: Baseline imputer run on the same masked matrices for recovery-error comparison
- name: scImpute
  role: Baseline imputer run on the same masked matrices for recovery-error comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:6fb2976091c4d71fadd78e658addedf8bb54083bcddbab836c3c4ffe6bc85889
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# autoencoder-imputation-model-training

## Summary

Train the AutoImpute overcomplete-autoencoder on sparse scRNA-seq count matrices (CSV/MAT, cells as rows, genes as columns) with TensorFlow/RMSProp, tuning lambda, hidden units, learning rate, and convergence threshold per dataset. Used to impute dropouts or to run masked-matrix recovery benchmarks (NMSE/RMSE/MAE).

## When to use

Use when a pre-processed, log-transformed single-cell count matrix (cells x genes, CSV or .mat) has dropouts or artificially masked non-zero values that need imputation with an overcomplete autoencoder, or when benchmarking imputation recovery error.

## When NOT to use

- The matrix has not been pre-processed/log-transformed — run pre-process.R first rather than feeding raw counts to AutoImpute.py
- The baseline imputer under comparison is drImpute in a masked-recovery benchmark, since its gene-filtering preprocessing changes matrix dimensions and makes error computation invalid
- The dataset is very large (e.g. PBMC was excluded from the masked-recovery test due to its size)

## Inputs

- Pre-processed count matrix as <dataset_name>.csv or <dataset_name>.mat (rows=cells, columns=genes), e.g. blakeley.csv
- Raw dataset CSV placed in AutoImpute Model/data/raw/ when preprocessing is needed

## Outputs

- Imputed count matrix with the same dimensions as the input (--imputed_save imputed_matrix)
- Masked matrix saved alongside imputation during masked-matrix tests (--masked_save masked_matrix)
- Training log (log.txt) and debug loss values; optional saved TensorFlow checkpoint (checkpoints/model.ckpt)

## How to apply

First pre-process raw data with the R script pre-process.R (Rscript pre-process.R <input-file-name> <dataset-name>) so columns are genes and rows are cells. Then run AutoImpute.py with per-dataset hyper-parameters, e.g. Blakeley: lambda=1, hidden_units=2000, initial_learning_rate=0.0001, threshold=0.0001 (stop gradient descent when the loss change between consecutive iterations falls below the threshold), iterations up to 7000. For the masked-recovery experiment, set --masked_matrix_test True and --masking_percentage to one of 10/20/30/40/50 to mute non-zero expression values, then compute NMSE, RMSE, and MAE at masked locations between the imputed and original matrices; repeat each run 10 times to average over random weight initialization. The rationale is that the overcomplete autoencoder learns the inherent distribution of the input scRNA-seq data and imputes missing values with minimal modification to biologically silent genes.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation model executed via AutoImpute.py; performs training, masking, and imputed-matrix output) — https://github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Backend used to train the autoencoder with the RMSProp optimizer)
- **numpy** (Python dependency for matrix handling during training and metric computation)
- **scikit-learn** (Python dependency for the AutoImpute pipeline)
- **matplotlib** (Python dependency used for plotting (e.g. RMSE-vs-masking-percentage curves))
- **MAGIC** (Baseline imputer run on the same masked matrices for recovery-error comparison)
- **scImpute** (Baseline imputer run on the same masked matrices for recovery-error comparison)

## Examples

```
python AutoImpute.py --data blakeley.csv --masked_matrix_test True --masking_percentage 10 --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001 --threshold 0.0001
```

## Evaluation signals

- recovery_metrics.csv contains NMSE, RMSE, and MAE (mean and variance over 10 runs) for AutoImpute, MAGIC, and scImpute at all five masking levels (10–50%) on all seven datasets (Blakeley, Jurkat-293T, Kolodziejczyk, Preimplantation, Quake, Usoskin, Zeisel)
- AutoImpute attains the lowest RMSE across the majority of the seven datasets, reproducing the Fig. 3 RMSE-vs-masking-percentage trend
- Training converges: gradient descent stops when the change in loss between consecutive iterations is below threshold=0.0001 (debug log shows loss values every debug_display_step)
- Imputed matrix has exactly the same dimensions as the input matrix
- PBMC is excluded from the masked-recovery benchmark due to size, and drImpute errors are skipped because gene filtering changes matrix dimensions

## Limitations

- Convergence depends on the threshold option (default 0.0001) stopping gradient descent; results vary with random weight initialization, so 10 repeated runs are needed for stable mean/variance
- Hyper-parameters are dataset-specific (e.g. Blakeley lambda=1, 2000 hidden units, lr 1e-4); reusing one setting across datasets is not validated
- Very large datasets such as PBMC are impractical for the masked-matrix test due to size
- AutoImpute.py assumes Python 2.7 with numpy, scikit-learn, tensorflow, matplotlib, and pre-processing requires an R toolchain

## Evidence

- [intro] AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values.: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [readme] The input to AutoImpute.py is a pre-processed count single cell matrix, with columns representing genes and rows representing cells.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] --threshold 0.0001 # To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence."
- [readme] --masking_percentage 10 # Percentage of masking required. Like 10, 20, 12.5 etc.: "--masking_percentage 10	# Percentage of masking required. Like 10, 20, 12.5 etc."
