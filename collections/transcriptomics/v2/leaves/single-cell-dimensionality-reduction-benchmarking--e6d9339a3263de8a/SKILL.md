---
name: single-cell-dimensionality-reduction-benchmarking
description: Use when assessing whether a dimensionality-reduction method can handle
  a million-plus-cell scRNA-seq dataset within practical runtime, by timing ivis against
  Barnes-Hut t-SNE with a 24-hour timeout.
when_to_use_negative:
- The dataset is small or moderate in size (e.g. the 4,645 x 23,686 Melanoma SCP11
  dataset or the 70,000-observation MNIST dataset), where t-SNE finishes well within
  the timeout and a runtime-scaling benchmark is uninformative
- The goal is accuracy/structure-preservation comparison (e.g. Mantel-test PCC or
  SVM-classifier accuracy on subsamples) rather than runtime scalability
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ivis
  role: Dimensionality-reduction method being benchmarked; run on the full dataset
    with large-dataset hyperparameters (k=15-150, n_epochs_without_progress=2-3, model='szubert')
  repo: https://github.com/beringresearch/ivis
- name: scikit-learn
  role: Provides the Barnes-Hut t-SNE baseline implementation under a 24-hour timeout,
    and MinMaxScaler for [0, 1] preprocessing
- name: t-SNE
  role: Comparison algorithm whose runtime is benchmarked against ivis on the 1.3M-cell
    matrix
- name: Python
  role: Execution environment for the benchmark pipeline (ivis, scikit-learn interfaces)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:f622b50b1e81caa16401905bbd143174c22e4f68e1659c54be2795d884e0a83c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-dimensionality-reduction-benchmarking

## Summary

A benchmarking skill for comparing the runtime and scalability of the ivis siamese-network dimensionality-reduction algorithm against scikit-learn's Barnes-Hut t-SNE on very large single-cell datasets (e.g. the 1.3-million-cell 10X Genomics mouse brain scRNA-seq matrix), demonstrating ivis's linear scaling to millions of observations.

## When to use

Use when assessing whether a dimensionality-reduction method can handle a million-plus-cell scRNA-seq dataset within practical runtime, by timing ivis against Barnes-Hut t-SNE with a 24-hour timeout.

## When NOT to use

- The dataset is small or moderate in size (e.g. the 4,645 x 23,686 Melanoma SCP11 dataset or the 70,000-observation MNIST dataset), where t-SNE finishes well within the timeout and a runtime-scaling benchmark is uninformative
- The goal is accuracy/structure-preservation comparison (e.g. Mantel-test PCC or SVM-classifier accuracy on subsamples) rather than runtime scalability

## Inputs

- 1.3-million-cell 10X Genomics mouse brain scRNA-seq expression matrix (downloaded public dataset)
- Preprocessed matrix scaled to [0, 1] via sklearn MinMaxScaler

## Outputs

- runtime_benchmark_results.csv containing a runtime row for both ivis and Barnes-Hut t-SNE
- ivis embedding of the full dataset produced without subsampling in under 30 minutes
- recorded t-SNE non-completion outcome (>24 hours)

## How to apply

Download the public 1.3-million-cell 10X Genomics mouse brain scRNA-seq dataset and scale all expression values to the [0, 1] range with sklearn's MinMaxScaler, following the paper's preprocessing convention. Run ivis on the full unsampled matrix using the large-dataset hyperparameters recommended for >1M observations (k=15-150, n_epochs_without_progress=2-3, model='szubert') and record total training time. Launch scikit-learn's Barnes-Hut t-SNE on the same preprocessed matrix with a 24-hour timeout, recording non-completion if the limit is exceeded. Write both runtimes to runtime_benchmark_results.csv; validation requires a row for each method showing ivis completing in under 30 minutes while t-SNE exceeds 24 hours, consistent with the paper's claim that ivis scales linearly to millions of observations.

## Related tools

- **ivis** (Dimensionality-reduction method being benchmarked; run on the full dataset with large-dataset hyperparameters (k=15-150, n_epochs_without_progress=2-3, model='szubert')) — https://github.com/beringresearch/ivis
- **scikit-learn** (Provides the Barnes-Hut t-SNE baseline implementation under a 24-hour timeout, and MinMaxScaler for [0, 1] preprocessing)
- **t-SNE** (Comparison algorithm whose runtime is benchmarked against ivis on the 1.3M-cell matrix)
- **Python** (Execution environment for the benchmark pipeline (ivis, scikit-learn interfaces))

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- runtime_benchmark_results.csv exists and contains a runtime row for both ivis and Barnes-Hut t-SNE
- Recorded ivis runtime is under 30 minutes on the full (unsubsampled) 1.3M-cell matrix
- Recorded t-SNE outcome exceeds the 24-hour timeout (recorded as non-completion)
- Expression matrix was scaled to the [0, 1] range before both runs (MinMaxScaler applied identically)
- ivis hyperparameters match the >1M observation recommendation (k=15-150, n_epochs_without_progress=2-3, model='szubert')

## Limitations

- The abstract's scalability claim is stated as linear scaling to hundreds of thousands of cells; the million-cell runtime claim relies on the benchmark outcome with t-SNE truncated at a 24-hour timeout rather than a completed t-SNE run
- t-SNE's runtime is censored at the timeout, so the comparison quantifies a lower bound on t-SNE runtime rather than exact runtimes for both methods
- t-SNE tends to yield spurious clusters with clean boundaries in the presence of random noise, so runtime comparisons should not be conflated with embedding-quality comparisons

## Evidence

- [abstract] ivis scales linearly to hundreds of thousands of cells: "and scales linearly to hundreds of thousands of cells"
- [methods] MinMaxScaler preprocessing convention: "X = MinMaxScaler().fit_transform(X)"
- [readme] ivis designed for very large datasets via siamese triplet training: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets."
- [discussion] t-SNE spurious clusters under noise, caution on interpreting embeddings: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries"
- [other] Runtime benchmark finding on the 1.3M-cell 10X dataset: [evidence span withheld — anchor status: not_found]
