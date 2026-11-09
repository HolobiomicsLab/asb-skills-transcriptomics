---
name: tensorflow-neural-network-training
description: Use when a raw scRNA-Seq count matrix needs deep-learning-based denoising/imputation
  and you must reconstruct the AutoClass architecture (autoencoder + classifier branch
  with pre-clustering) in TensorFlow.
when_to_use_negative:
- When a strong distribution assumption is acceptable and a simpler method such as
  DCA (zero-inflated negative binomial), SAVER, or scImpute suffices for a single
  noise type like dropouts
- When no raw count matrix is available (e.g. input is already a normalized/feature
  table) and the take_norm + count-based pipeline cannot be applied as described
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoClass
  role: Composite deep neural network (autoencoder + classifier) trained in TensorFlow
    to denoise and impute scRNA-Seq data
  repo: https://github.com/datapplab/AutoClass
- name: TensorFlow >=2.0
  role: Deep learning framework used to implement and train the AutoClass network
- name: numpy
  role: Numerical operations on the count matrix (X = expr.values) during preprocessing
    and imputation
- name: pandas
  role: Loading the raw count matrix CSV (pd.read_csv('Dataset1_raw.csv', index_col=0))
    and writing the imputed matrix
- name: sklearn
  role: K-means pre-clustering used to generate virtual class labels for the classifier
    branch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:e46d239ff2ce05bc518fc25a24f2b658cf67d4069c86f44630d5c78847df3f02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - Python
  - TensorFlow
  - TensorFlow >=2.0
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# tensorflow-neural-network-training

## Summary

Build and train the AutoClass composite network in TensorFlow (>=2.0): an autoencoder with a classifier branch attached to the bottleneck that uses virtual class labels from K-means pre-clustering, combining reconstruction and classification losses to denoise scRNA-Seq count matrices.

## When to use

Use when a raw scRNA-Seq count matrix needs deep-learning-based denoising/imputation and you must reconstruct the AutoClass architecture (autoencoder + classifier branch with pre-clustering) in TensorFlow.

## When NOT to use

- When a strong distribution assumption is acceptable and a simpler method such as DCA (zero-inflated negative binomial), SAVER, or scImpute suffices for a single noise type like dropouts
- When no raw count matrix is available (e.g. input is already a normalized/feature table) and the take_norm + count-based pipeline cannot be applied as described

## Inputs

- Raw scRNA-Seq gene-by-cell count matrix (e.g. Dataset1_raw.csv, genes x cells CSV)
- Number of expected clusters K for pre-clustering (K-1, K, K+1)
- Reference true counts for validation (e.g. Dataset1_true.csv, optional)

## Outputs

- Averaged imputed/denoised expression matrix saved as imputed_matrix.csv with the same gene-by-cell dimensions as the raw input
- Trained TensorFlow composite network (autoencoder + classifier branch)

## How to apply

Load the raw count matrix (e.g. Dataset1_raw.csv with pandas, index_col=0) and normalize with take_norm to log2 scale. Run K-means pre-clustering at K-1, K, and K+1 clusters to generate virtual class labels for the classifier branch, since AutoClass replaces known cell classes with these unsupervised labels. Build the composite network in TensorFlow: an encoder -> bottleneck -> decoder autoencoder with a classifier branch off the bottleneck, trained with a weighted loss combining reconstruction error and classification loss via classifier weight w, using dropout_rate=0.3 in AutoClassImpute. Run AutoClassImpute three times (one per pre-clustering number) and average the imputed outputs to obtain the final matrix, then save it (e.g. imputed_matrix.csv). The rationale is that the classifier branch filters out noise while retaining signal, and AutoClass is distribution agnostic — it makes no assumption on specific data distributions.

## Related tools

- **AutoClass** (Composite deep neural network (autoencoder + classifier) trained in TensorFlow to denoise and impute scRNA-Seq data) — https://github.com/datapplab/AutoClass
- **TensorFlow >=2.0** (Deep learning framework used to implement and train the AutoClass network)
- **numpy** (Numerical operations on the count matrix (X = expr.values) during preprocessing and imputation)
- **pandas** (Loading the raw count matrix CSV (pd.read_csv('Dataset1_raw.csv', index_col=0)) and writing the imputed matrix)
- **sklearn** (K-means pre-clustering used to generate virtual class labels for the classifier branch)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); X = expr.values; res = AutoClassImpute(X, dropout_rate=0.3)
```

## Evaluation signals

- imputed_matrix.csv exists and has the same gene-by-cell dimensions as the raw input matrix
- All imputed values are non-negative
- Imputed values cross-checked against reference true counts in Dataset1_true.csv for reconstruction fidelity
- Model predictions averaged over the three pre-clustering numbers (K-1, K, K+1) reduce sensitivity to the pre-clustering choice
- Downstream clustering on the imputed data recovers cell type pattern, which the regular autoencoder alone could not

## Limitations

- The regular autoencoder alone cannot recover cell type pattern — the classifier branch is necessary
- No software version numbers are stated in the results section, so exact reproducibility of library versions must be pinned separately
- Results depend on hyperparameters (bottleneck size, pre-clustering number, classifier weight), though the paper reports robustness to these settings

## Evidence

- [results] AutoClass adds a classifier branch from the bottleneck layer of the autoencoder: [evidence span withheld — anchor status: not_found]
- [abstract] AutoClass integrates two deep neural network components: "Here we design and develop the AutoClass model, integrating two deep neural network components, an autoencoder, and a classifier"
- [methods] TensorFlow >=2.0 requirement: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed first"
- [results] Imputation call with dropout rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [abstract] Distribution agnostic property: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [intro] Classifier branch necessary: "AutoClass but not the regular autoencoder was able to recover cell type pattern, indicating the classifier component is necessary for reconstructing scRNA-Seq data."
