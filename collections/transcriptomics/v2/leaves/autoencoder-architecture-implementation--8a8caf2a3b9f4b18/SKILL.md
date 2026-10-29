---
name: autoencoder-architecture-implementation
description: Use when highly sparse scRNA-seq count matrices (e.g., >80-95% zeros,
  masked .hd5 inputs like example.msk90.hd5) need imputation and you must build or
  reproduce the LATE autoencoder network in TensorFlow 1.x.
when_to_use_negative:
- 'You need to distinguish biological zeros from technical zeros: LATE assumes all
  zeros are missing values and always imputes them with nonzero values'
- 'You want systematic hyperparameter tuning (as in DCA): LATE hyperparameters were
  manually tuned only'
- 'Your environment only supports TensorFlow 2: the implementation requires TensorFlow
  1.0-1.4 (or 1.15 in a virtualenv) and is not compatible with TensorFlow 2'
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: LATE
  role: Imputation method whose autoencoder this skill implements; trained de novo
    on sparse scRNA-seq data with random initial parameter values
  repo: https://github.com/audreyqyfu/LATE
- name: TensorFlow (1.0-1.4)
  role: Framework used to build and train the fully connected autoencoder with ReLU
    activations and dropout
- name: Python 3.5+
  role: Language of the implementation, alongside numpy, pandas, scipy, and tables
    for I/O on hd5 matrices
- name: pandas
  role: Reads the masked .hd5 input matrix and preserves gene-by-cell layout
- name: numpy
  role: Array operations for the count matrix during transformation and training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:2e04b13e2eb391be7ad96e35bda97bc5c55070a2e297a9e8d42d80d7e00f0010
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - LATE
  - Python 3.5+
  - TensorFlow
  - TensorFlow (1.0-1.4)
  - numpy
  - pandas
  - scipy
  - tables
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# autoencoder-architecture-implementation

## Summary

Implements the LATE autoencoder architecture: an autoencoder with randomly initialized parameters, fully connected hidden layers (e.g., 400 nodes, ReLU), and dropout, trained on log10(count+1)-transformed sparse scRNA-seq data to reconstruct and impute missing (zero) expression values.

## When to use

Use when highly sparse scRNA-seq count matrices (e.g., >80-95% zeros, masked .hd5 inputs like example.msk90.hd5) need imputation and you must build or reproduce the LATE autoencoder network in TensorFlow 1.x.

## When NOT to use

- You need to distinguish biological zeros from technical zeros: LATE assumes all zeros are missing values and always imputes them with nonzero values
- You want systematic hyperparameter tuning (as in DCA): LATE hyperparameters were manually tuned only
- Your environment only supports TensorFlow 2: the implementation requires TensorFlow 1.0-1.4 (or 1.15 in a virtualenv) and is not compatible with TensorFlow 2

## Inputs

- Masked scRNA-seq count matrix in .hd5 format (e.g., example.msk90.hd5, 10,000 genes x 10,000 cells), csv, tsv, or 10x Genomics h5
- Input matrix with row names (cell IDs) and column names (gene IDs)

## Outputs

- step2/imputation.step2.hd5 — imputed data matrix in hd5 format with the same layout as the input, on the log10(count+1) scale
- Trained autoencoder weights in the step2/ folder (reusable for 'impute' mode without retraining)

## How to apply

Read the masked gene-by-cell count matrix (e.g., example.msk90.hd5, 10,000 genes x 10,000 cells) with pandas/tables and transform values with log10(count+1). Build the autoencoder in TensorFlow 1.x with randomly generated initial parameter values, stacking 1, 3, or 5 fully connected hidden layers (e.g., 400 nodes each) with ReLU activations between an input and output layer of equal dimension, applying dropout to nodes. Train with mini-batch backpropagation using the Adam optimizer and a mean-squared-error loss computed only on nonzero input values, taking default hyperparameters from global_params.py (short runs may be specified in example_late.py). Then run the trained network in 'impute' mode to calculate imputed values for the full input matrix without further training, and write the result to step2/imputation.step2.hd5 in hd5 format, preserving the input layout and log10(count+1) scale.

## Related tools

- **LATE** (Imputation method whose autoencoder this skill implements; trained de novo on sparse scRNA-seq data with random initial parameter values) — https://github.com/audreyqyfu/LATE
- **TensorFlow (1.0-1.4)** (Framework used to build and train the fully connected autoencoder with ReLU activations and dropout)
- **Python 3.5+** (Language of the implementation, alongside numpy, pandas, scipy, and tables for I/O on hd5 matrices)
- **pandas** (Reads the masked .hd5 input matrix and preserves gene-by-cell layout)
- **numpy** (Array operations for the count matrix during transformation and training)

## Examples

```
python example_late.py -mode='late' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Output file step2/imputation.step2.hd5 exists and matches the input gene-by-cell dimensions (e.g., 10,000 x 10,000)
- Output values are on the log10(count+1) scale with the same layout as the input
- MSE of the imputed matrix against the unmasked ground truth (example.hd5) is lower than baseline imputation methods, consistent with the paper's reported performance
- Imputed entries corresponding to masked (zero) values are nonzero, since LATE always imputes zeros with a nonzero value
- Training minimized the mean-squared-error loss restricted to nonzero input values, and the network converged with randomly initialized parameters

## Limitations

- Hyperparameters were manually tuned; systematic tuning as implemented in DCA was not explored
- All zeros are assumed to be missing values; biological zeros are always imputed with nonzero values
- Combining imputed data matrices from both dimensions (genes and cells) is ad hoc; joint optimization was not performed
- The code must be run from the folder where it is stored (e.g., cd scripts/) and is not compatible with TensorFlow 2; a virtualenv with tensorflow==1.15 and pandas==0.25.1 is recommended

## Evidence

- [intro] LATE trains an autoencoder de novo on the highly sparse scRNA-seq data, with the initial values of the parameters randomly generated: "Our LATE (Learning with AuToEncoder) method trains an autoencoder de novo on the highly sparse scRNA-seq data, with the initial values of the parameters randomly generated."
- [methods] This implementation is written in Python (3.5+) and builds on TensorFlow (1.0-1.4).: "This implementation is written in Python (3.5+) and builds on TensorFlow (1.0-1.4)."
- [readme] Imputed data matrix is in the hd5 format and stored in `step2/imputation.step2.hd5`.: "Imputed data matrix is in the hd5 format and stored in `step2/imputation.step2.hd5`."
- [readme] The input data will be transformed with log10(count+1) for imputation.: "The input data will be transformed with log10(count+1) for imputation."
- [abstract] LATE and TRANSLATE outperform existing scRNA-seq imputation methods, achieving lower mean squared error in most cases: "LATE and TRANSLATE outperform existing scRNA-seq imputation methods, achieving lower mean squared error in most cases"
