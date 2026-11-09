---
name: transfer-learning-pretraining-on-reference-scrna
description: Use when a sparse scRNA-seq count matrix (e.g., example.msk90.hd5 / PBMC_G949_10K)
  must be imputed and a labeled reference dataset from the same platform (e.g., ref_example.mdk50.hd5)
  is available to initialize the autoencoder.
when_to_use_negative:
- No labeled reference dataset from a comparable platform is available — use plain
  LATE ('late' mode, random initialization) instead
- The environment only supports TensorFlow 2 — LATE/TRANSLATE are incompatible with
  TensorFlow 2
- The data are not single-cell gene expression counts (e.g., already normalized feature
  tables) — the methods assume raw counts with excess zeros treated as dropouts
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TRANSLATE
  role: 'Two-step transfer learning pipeline: pre-training on the reference dataset
    (step1/) then translate/impute on the target dataset (step2/)'
  repo: audreyqyfu/LATE
- name: LATE
  role: Underlying autoencoder whose parameters are randomly initialized during pre-training
    and reused as initialization in the translate step
  repo: audreyqyfu/LATE
- name: TensorFlow (1.0-1.4 / 1.15)
  role: Backend that trains the autoencoder; version 1.x required in a virtual environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:dd1b88f04d0910de6d16840425ba1029ecb6a44584cc63ed2be3640dde65f163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - DCA
  - LATE
  - MulticoreTSNE
  - Python+
  - TRANSLATE
  - TensorFlow
  - TensorFlow (1.0-1.4 / 1.15)
  - matplotlib
  - numpy
  - pandas
  - scVI
  - scipy
  - seaborn
  - sklearn
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# transfer-learning-pretraining-on-reference-scrna

## Summary

TRANSLATE-style transfer learning for scRNA-seq imputation: pre-train the LATE autoencoder on a labeled reference expression dataset, then use the learned weights and biases as initialization when imputing the target masked dataset, instead of starting from random parameters.

## When to use

Use when a sparse scRNA-seq count matrix (e.g., example.msk90.hd5 / PBMC_G949_10K) must be imputed and a labeled reference dataset from the same platform (e.g., ref_example.mdk50.hd5) is available to initialize the autoencoder.

## When NOT to use

- No labeled reference dataset from a comparable platform is available — use plain LATE ('late' mode, random initialization) instead
- The environment only supports TensorFlow 2 — LATE/TRANSLATE are incompatible with TensorFlow 2
- The data are not single-cell gene expression counts (e.g., already normalized feature tables) — the methods assume raw counts with excess zeros treated as dropouts

## Inputs

- Reference scRNA-seq dataset with labeled cell types in hd5 format (ref_example.mdk50.hd5; 10,000 genes and 30,000 cells)
- Target masked scRNA-seq count matrix in hd5/csv/tsv format (example.msk90.hd5 / PBMC_G949_10K; 10,000 genes and 10,000 cells, 90% masked)
- Ground truth count matrix (example.hd5) for downstream analysis

## Outputs

- step1/ folder containing pre-trained autoencoder parameters from the reference dataset
- step2/imputation.step2.hd5 — imputed matrix on the log10(count+1) scale with the same layout as the input
- Eval/ folder from example_analysis.py with imputation summaries and t-SNE visualizations

## How to apply

Run TRANSLATE in two steps: Step 1 ('pre-training') trains the LATE autoencoder with random initialization on the reference dataset (ref_example.mdk50.hd5, 10,000 genes x 30,000 cells), writing outputs to a step1/ folder; Step 2 ('translate') trains on the masked input (example.msk90.hd5), using the pre-trained parameters as initialization, writing to step2/. The input count matrix is log10(count+1)-transformed for imputation, and the imputed matrix is stored as step2/imputation.step2.hd5 on the same scale with the same layout. Run LATE requires TensorFlow 1.x (e.g., 1.15) in a virtual environment because LATE is incompatible with TensorFlow 2. The rationale is transfer learning: parameter estimates learned on the labeled reference give the autoencoder a better starting point than random initialization on the sparse target data alone.

## Related tools

- **TRANSLATE** (Two-step transfer learning pipeline: pre-training on the reference dataset (step1/) then translate/impute on the target dataset (step2/)) — audreyqyfu/LATE
- **LATE** (Underlying autoencoder whose parameters are randomly initialized during pre-training and reused as initialization in the translate step) — audreyqyfu/LATE
- **TensorFlow (1.0-1.4 / 1.15)** (Backend that trains the autoencoder; version 1.x required in a virtual environment)

## Examples

```
python example_translate_step1.py -mode='pre-training' -infile='../data/ref_example.mdk50.hd5'; python example_translate_step2.py -mode='translate' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- Step 1 creates a step1/ folder with pre-trained parameters; step 2 creates step2/imputation.step2.hd5 with the same layout as the input
- Values in step2/imputation.step2.hd5 are on the log10(count+1) scale, matching the README output specification
- t-SNE embedding (PCA projection followed by MulticoreTSNE) of the imputed data colored by the 10 cell-type labels resembles the ground truth plot
- Cell-type separation metrics show a high between-cluster sum of squares (BCSS) and a low within-cluster sum of squares (WCSS), matching Figure 5's pattern for LATE/TRANSLATE (unlike SAVER, scImpute, MAGIC, ALRA)
- The translate step must use step1/ pre-trained parameters as initialization rather than random initialization

## Limitations

- LATE is not compatible with TensorFlow 2; a virtual environment with TensorFlow 1.x (e.g., 1.15) and pandas 0.25.1 is needed
- LATE/TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value, so biological zeros are not distinguished from technical zeros
- Hyperparameters were manually tuned; systematic hyperparameter tuning (as implemented in DCA) was not explored

## Evidence

- [abstract] TRANSLATE initializes LATE parameters using a reference gene expression data set (transfer learning): [evidence span withheld — anchor status: not_found]
