---
name: autoencoder-classifier-composite-network-implementation
description: Use when a raw scRNA-Seq count matrix (e.g., genes x cells CSV) contains
  dropouts and other noise, and a distribution-free deep learning imputation that
  preserves cell-type signal is needed.
when_to_use_negative:
- When the method's strong distribution assumptions match your data better via alternatives
  with negative binomial assumptions (SAVER, DCA) or Gaussian/Gamma mixtures (scImpute)
  — i.e., when you specifically require model-based, distribution-assuming imputation
  rather than a distribution-agnostic network
- When input is not a raw count matrix (e.g., already normalized, batch-corrected,
  or a feature table), since the workflow starts from raw counts normalized via take_norm
- When TensorFlow (>=2.0) cannot be installed or GPU/NN training is infeasible for
  the data size
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoClass
  role: Core composite autoencoder+classifier network implemented and run via AutoClassImpute
  repo: https://github.com/datapplab/AutoClass
- name: TensorFlow >=2.0
  role: Deep learning framework in which the composite network is built and trained
- name: sklearn
  role: KMeans pre-clustering to generate virtual class labels and PCA/TSNE for evaluation
- name: numpy
  role: Array manipulation of the count matrix (X = expr.values)
- name: pandas
  role: Loading raw count CSVs with index_col=0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:ad95641f43f34c5de83d950c2e6c0223c739b2a59ad5719d6364d5da46e61f56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - Python
  - TensorFlow >=2.0
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# autoencoder-classifier-composite-network-implementation

## Summary

Implements AutoClass, a composite deep neural network for scRNA-Seq denoising that couples an autoencoder's bottleneck layer to a classifier branch trained on virtual class labels from K-means pre-clustering, maximizing both noise removal and signal retention without distribution assumptions.

## When to use

Use when a raw scRNA-Seq count matrix (e.g., genes x cells CSV) contains dropouts and other noise, and a distribution-free deep learning imputation that preserves cell-type signal is needed.

## When NOT to use

- When the method's strong distribution assumptions match your data better via alternatives with negative binomial assumptions (SAVER, DCA) or Gaussian/Gamma mixtures (scImpute) — i.e., when you specifically require model-based, distribution-assuming imputation rather than a distribution-agnostic network
- When input is not a raw count matrix (e.g., already normalized, batch-corrected, or a feature table), since the workflow starts from raw counts normalized via take_norm
- When TensorFlow (>=2.0) cannot be installed or GPU/NN training is infeasible for the data size

## Inputs

- Raw scRNA-Seq count matrix as CSV (e.g., Dataset1_raw.csv, gene x cell, index_col=0)
- Optional reference true counts for validation (e.g., Dataset1_true.csv)

## Outputs

- Imputed/denoised expression matrix of the same gene-by-cell dimensions as the input (e.g., imputed_matrix.csv)
- Optionally PCA-50 plus t-SNE 2D embedding of normalized data for cluster quality assessment

## How to apply

Load the raw count matrix (e.g., Dataset1_raw.csv) with pandas (index_col=0) and log2-normalize it with take_norm. Run K-means pre-clustering at K-1, K, and K+1 clusters to generate virtual class labels that train the classifier branch attached to the autoencoder's bottleneck layer. Build the composite network in TensorFlow (>=2.0): encoder -> bottleneck -> decoder plus classifier, trained with a weighted loss combining reconstruction error and classification loss scaled by classifier weight w. Run AutoClassImpute with dropout_rate=0.3, average predictions over the three pre-clustering numbers, and save the averaged imputed matrix (e.g., imputed_matrix.csv). The classifier component is necessary: a regular autoencoder alone failed to recover cell type pattern in the paper's evaluation.

## Related tools

- **AutoClass** (Core composite autoencoder+classifier network implemented and run via AutoClassImpute) — https://github.com/datapplab/AutoClass
- **TensorFlow >=2.0** (Deep learning framework in which the composite network is built and trained)
- **sklearn** (KMeans pre-clustering to generate virtual class labels and PCA/TSNE for evaluation)
- **numpy** (Array manipulation of the count matrix (X = expr.values))
- **pandas** (Loading raw count CSVs with index_col=0)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); X = expr.values; res = AutoClassImpute(X, dropout_rate=0.3); res.to_csv('imputed_matrix.csv')
```

## Evaluation signals

- imputed_matrix.csv exists and has the same gene-by-cell dimensions as the raw input matrix
- All imputed values are non-negative
- Downstream fidelity cross-checked against reference true counts (Dataset1_true.csv)
- Cell type pattern recovery (e.g., KMeans purity on t-SNE embedding of top-200 highly variable genes from find_hv_genes) improves over the regular autoencoder baseline
- Results are stable across the pre-clustering numbers K-1, K, K+1 whose predictions are averaged

## Limitations

- The paper's discussion is entirely affirmative with no explicit limitations section; limitations must be inferred
- No software version numbers are stated beyond TensorFlow (>=2.0) in the provided text
- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced but not included, so exact hyperparameter sweeps cannot be reproduced from the provided context alone
- Key hyperparameters (bottleneck layer size, pre-clustering number, classifier weight w) require setting, though the paper reports robustness to these settings

## Evidence

- [intro] composite network design: "AutoClass integrates two neural network components: an autoencoder and a classifier"
- [intro] classifier necessity: "AutoClass but not the regular autoencoder was able to recover cell type pattern, indicating the classifier component is necessary for reconstructing scRNA-Seq data."
- [abstract] distribution agnostic: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [results] imputation call: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [methods] environment setup: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] pre-clustering labels from source card: [evidence span withheld — anchor status: not_found]
- [readme] open-source availability: "We have made AutoClass open source at: https://github.com/datapplab/AutoClass"
