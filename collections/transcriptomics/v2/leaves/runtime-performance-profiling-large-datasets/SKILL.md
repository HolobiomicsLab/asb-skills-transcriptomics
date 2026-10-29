---
name: runtime-performance-profiling-large-datasets
description: Use when you must verify that a dimensionality-reduction method scales
  to a million-plus-observation dataset by timing it against a baseline algorithm
  with a fixed timeout (e.g., 24 hours for Barnes-Hut t-SNE).
when_to_use_negative:
- The dataset is small enough (e.g., tens of thousands of observations like MNIST
  70,000 or Levine-32-dim 265,627 cells) that the baseline algorithm completes quickly,
  making the scaling comparison uninformative
- The goal is embedding quality rather than runtime — use accuracy-based evaluations
  such as Mantel-test Pearson correlation or SVM classification on subsamples instead
- The matrix requires subsampling to fit in memory — ivis's linear-scaling claim was
  demonstrated without subsampling
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Scalable dimensionality-reduction method whose training time on the full dataset
    is measured; installed via pip (ivis[cpu] or ivis[gpu]) and runs on TensorFlow
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Baseline Barnes-Hut implementation from scikit-learn run on the same preprocessed
    matrix with a 24-hour timeout
- name: scikit-learn
  role: Provides the Barnes-Hut t-SNE baseline and the MinMaxScaler used to scale
    values to the [0, 1] range before timing
- name: Python
  role: Language in which both benchmark runs and result recording are implemented
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:cebf71d583dd8505f418816697e651da119d81447f924c2cedba38d0ccfd18fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# runtime-performance-profiling-large-datasets

## Summary

Benchmarks the wall-clock runtime of a scalable dimensionality-reduction algorithm (ivis, a siamese neural network trained with triplet loss) against a reference implementation (scikit-learn Barnes-Hut t-SNE) on a very large single-cell expression matrix, recording completion times to verify linear-scaling claims.

## When to use

Use when you must verify that a dimensionality-reduction method scales to a million-plus-observation dataset by timing it against a baseline algorithm with a fixed timeout (e.g., 24 hours for Barnes-Hut t-SNE).

## When NOT to use

- The dataset is small enough (e.g., tens of thousands of observations like MNIST 70,000 or Levine-32-dim 265,627 cells) that the baseline algorithm completes quickly, making the scaling comparison uninformative
- The goal is embedding quality rather than runtime — use accuracy-based evaluations such as Mantel-test Pearson correlation or SVM classification on subsamples instead
- The matrix requires subsampling to fit in memory — ivis's linear-scaling claim was demonstrated without subsampling

## Inputs

- Full scRNA-seq expression matrix (e.g., 1.3M-cell 10X Genomics mouse brain dataset), unsampled
- Preprocessed matrix scaled to [0, 1] via sklearn MinMaxScaler
- Hyperparameter table for large datasets (k=15-150, n_epochs_without_progress=2-3, model='szubert' for >1M observations)
- Timeout budget for the baseline algorithm (24 hours)

## Outputs

- runtime_benchmark_results.csv containing a runtime row for both ivis and Barnes-Hut t-SNE
- Recorded ivis training time (<30 minutes) and t-SNE outcome (>24 hours / non-completion)

## How to apply

Download the full scRNA-seq expression matrix (e.g., the 1.3-million-cell 10X Genomics mouse brain dataset) without subsampling. Preprocess by scaling all values to the [0, 1] range with sklearn's MinMaxScaler, matching the paper's convention that 'values in both datasets were scaled to [0, 1] range'. Run ivis on the full matrix using large-dataset hyperparameters from the paper's hyperparameter table (k=15-150, n_epochs_without_progress=2-3, model='szubert' for >1M observations) and record total training time. Launch scikit-learn's Barnes-Hut t-SNE on the identical preprocessed matrix with a 24-hour timeout, recording non-completion if the limit is exceeded. The decision point is the runtime comparison: ivis should finish in under 30 minutes on 1.3M cells while t-SNE exceeds 24 hours, consistent with the claim that ivis scales linearly to millions of observations.

## Related tools

- **ivis** (Scalable dimensionality-reduction method whose training time on the full dataset is measured; installed via pip (ivis[cpu] or ivis[gpu]) and runs on TensorFlow) — https://github.com/beringresearch/ivis
- **t-SNE** (Baseline Barnes-Hut implementation from scikit-learn run on the same preprocessed matrix with a 24-hour timeout)
- **scikit-learn** (Provides the Barnes-Hut t-SNE baseline and the MinMaxScaler used to scale values to the [0, 1] range before timing)
- **Python** (Language in which both benchmark runs and result recording are implemented) — https://github.com/beringresearch/ivis

## Examples

```
from sklearn.preprocessing import MinMaxScaler; from ivis import Ivis; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- runtime_benchmark_results.csv exists and contains a runtime row for both ivis and Barnes-Hut t-SNE
- ivis runtime on the 1.3M-cell matrix is under 30 minutes
- Barnes-Hut t-SNE runtime exceeds the 24-hour timeout (recorded as non-completion)
- Both algorithms were run on the identical MinMaxScaler-scaled matrix, and ivis used the recommended >1M-observation hyperparameters (k=15-150, n_epochs_without_progress=2-3, model='szubert')
- The full dataset was used without subsampling, matching the paper's claim of linear scaling to millions of observations

## Limitations

- The paper's abstract states linear scaling only 'to hundreds of thousands of cells'; the 1.3M-cell 10X benchmark extends this via README claims that ivis 'easily extends to millions of observations'
- Baseline t-SNE may not complete within a 24-hour timeout at this scale, so no t-SNE embedding quality result is obtainable in the comparison
- Runtimes are hardware-dependent; the <30-minute ivis result assumes the recommended large-dataset hyperparameters and a CPU or GPU TensorFlow installation

## Evidence

- [abstract] ivis scales linearly to hundreds of thousands of cells: "and scales linearly to hundreds of thousands of cells"
- [methods] MinMaxScaler scaling to [0, 1]: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] ivis scalability claim in README: [evidence span withheld — anchor status: external_doc]
- [readme] ivis installation via pip on TensorFlow: [evidence span withheld — anchor status: external_doc]
