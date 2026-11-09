---
name: single-cell-rna-seq-imputation
description: Use when a scRNA-seq count matrix is highly sparse (>80% or even 95%
  zeros) and true expression levels behind excess zeros must be recovered computationally,
  treating all zeros as missing values.
when_to_use_negative:
- 'When zeros may be biological (true zero expression): LATE assumes all zeros are
  missing values and always imputes zeros with a nonzero value, without distinguishing
  biological from technical zeros'
- 'When TensorFlow 2 is the required/available framework: LATE is not compatible with
  TensorFlow 2 and needs a TF 1.x (1.0–1.4) virtual environment'
- When the input is not raw read counts but already imputed, normalized, or a feature/summary
  table
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LATE
  role: Core autoencoder method (mode='late') that trains de novo with randomly generated
    initial parameters to impute zeros in the sparse input matrix
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: 'Transfer-learning extension: ''pre-training'' trains the autoencoder on a
    reference dataset, and ''translate'' uses those weights/biases as initial values
    for the input dataset'
  repo: https://github.com/audreyqyfu/LATE
- name: TensorFlow (1.0-1.4)
  role: Backend framework in which the autoencoder is built and trained; requires
    a TF 1.x virtual environment (e.g. tensorflow==1.15), not TF2
- name: pandas
  role: Used (with tables) to load the masked hd5 input matrices and read the log-transformed
    count data; version pinned to 0.25.1 in the virtual environment
- name: DCA
  role: Comparison imputation method with an explicit model for read counts; benchmarked
    against LATE with rather unsatisfactory performance in the paper's comparison
- name: scVI
  role: Comparison imputation method with an explicit count model; benchmarked against
    LATE with rather unsatisfactory performance
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:0092da9200794774ad1ef661ffbe43b302438f136441595a5ec4ef1d61f1520b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - DCA
  - LATE
  - Python+
  - TRANSLATE
  - TensorFlow
  - TensorFlow (1.0-1.4)
  - numpy
  - pandas
  - scVI
  - scipy
  - tables
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# single-cell-rna-seq-imputation

## Summary

Impute dropout zeros in highly sparse scRNA-seq count matrices (e.g. 10x Genomics h5/csv/tsv inputs) by training an autoencoder (LATE) with random initialization on nonzero values, producing a log10(count+1)-scale imputed hd5 matrix.

## When to use

Use when a scRNA-seq count matrix is highly sparse (>80% or even 95% zeros) and true expression levels behind excess zeros must be recovered computationally, treating all zeros as missing values.

## When NOT to use

- When zeros may be biological (true zero expression): LATE assumes all zeros are missing values and always imputes zeros with a nonzero value, without distinguishing biological from technical zeros
- When TensorFlow 2 is the required/available framework: LATE is not compatible with TensorFlow 2 and needs a TF 1.x (1.0–1.4) virtual environment
- When the input is not raw read counts but already imputed, normalized, or a feature/summary table

## Inputs

- scRNA-seq read-count matrix in csv, tsv, or 10x Genomics h5 sparse matrix format (e.g. example.msk90.hd5 with 10,000 genes and 10,000 cells)
- Optional reference gene expression dataset for TRANSLATE pre-training (e.g. ref_example.mdk50.hd5)
- Optional ground-truth count matrix in the same format as the input (set to the input itself when unavailable, as in Example 4)
- Orientation specification (genes vs cells in rows/columns) for 10x h5 input files

## Outputs

- Imputed data matrix stored as step2/imputation.step2.hd5, same layout as input, on the log10(count+1) scale
- step1/ folder with pre-trained parameters when running TRANSLATE pre-training
- Eval/ folder with summary and visualization of imputation results from mode='analysis'

## How to apply

Load a gene-by-cell or cell-by-gene read-count matrix in csv, tsv, or 10x h5 format (explicitly specify orientation for h5), and note that the input is transformed with log10(count+1) before imputation. Build the autoencoder in Python 3.5+ with TensorFlow 1.x (1.0–1.4) in a virtual environment: randomly initialize parameters, stack fully connected hidden layers (e.g. 400 nodes with ReLU) between input and output of equal dimension, and train with mini-batch backpropagation using the Adam optimizer and mean-squared-error loss computed only on nonzero input values. Run with mode='late' (random initialization, no reference data) or use TRANSLATE transfer learning: 'pre-training' on a reference dataset such as ref_example.mdk50.hd5 (10,000 genes x 30,000 cells), then 'translate' on the input. Optionally run mode='impute' to calculate imputed values for the full matrix without further training. The imputed matrix is written to step2/imputation.step2.hd5 in hd5 format, preserving the input layout on the log10(count+1) scale; run mode='analysis' to summarize/visualize against ground truth in the Eval/ folder.

## Related tools

- **LATE** (Core autoencoder method (mode='late') that trains de novo with randomly generated initial parameters to impute zeros in the sparse input matrix) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning extension: 'pre-training' trains the autoencoder on a reference dataset, and 'translate' uses those weights/biases as initial values for the input dataset) — https://github.com/audreyqyfu/LATE
- **TensorFlow (1.0-1.4)** (Backend framework in which the autoencoder is built and trained; requires a TF 1.x virtual environment (e.g. tensorflow==1.15), not TF2)
- **pandas** (Used (with tables) to load the masked hd5 input matrices and read the log-transformed count data; version pinned to 0.25.1 in the virtual environment)
- **DCA** (Comparison imputation method with an explicit model for read counts; benchmarked against LATE with rather unsatisfactory performance in the paper's comparison)
- **scVI** (Comparison imputation method with an explicit count model; benchmarked against LATE with rather unsatisfactory performance)

## Examples

```
python example_late.py -mode='late' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Output file step2/imputation.step2.hd5 exists, matches the input gene-by-cell dimensions, and is on the log10(count+1) scale
- Mean squared error of the imputed matrix against the unmasked ground truth (e.g. example.hd5 vs the 90%-masked example.msk90.hd5) is lower than baseline imputation methods
- Imputed data better separates cell types and recovers nonlinear gene-gene relationships relative to baselines
- Scalability check: LATE should process over 1 million cells in a few hours on a GPU (e.g. 1.3M-cell mouse brain data in under 11 hours on GPU vs 53.2 hours on CPU)
- Mode='analysis' successfully generates the Eval/ folder with summary and visualization using the ground truth specified in example_analysis.py

## Limitations

- Hyperparameters were manually tuned; systematic hyperparameter tuning (as implemented in DCA) was not explored
- The current approach for combining imputed data matrices from both dimensions is ad hoc; joint optimization for both dimensions simultaneously was not performed
- All zeros are assumed to be missing values and are always imputed with a nonzero value; biological vs technical zeros are not distinguished
- Since the true dropout process is unknown, the choice of sampling distribution in simulation may introduce additional uncertainty in performance assessment
- LATE is not compatible with TensorFlow 2; a virtual environment with TensorFlow 1.x (e.g. 1.15) and pandas 0.25.1 is needed
- In the current version, code needs to run where it is stored on your computer (from the scripts/ directory)

## Evidence
