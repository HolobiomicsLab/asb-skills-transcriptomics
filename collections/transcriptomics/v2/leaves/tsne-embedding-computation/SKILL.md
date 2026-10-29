---
name: tsne-embedding-computation
description: Use when benchmarking dimensionality-reduction scalability on a large
  scRNA-seq or CyTOF matrix (e.g. the 1.3M-cell 10X Genomics mouse brain dataset),
  or when a t-SNE baseline embedding is required for comparison against ivis.
when_to_use_negative:
- When the goal is to embed millions of observations within practical time — the paper
  shows Barnes-Hut t-SNE did not finish within 24 hours on 1.3M cells; use ivis instead
- When embedding quality under random noise matters — t-SNE was shown to yield spurious
  clusters with clean boundaries in the presence of random noise
- When the downstream task requires adding new data points to an existing embedding
  — t-SNE lacks a transform method, unlike ivis's parametric mapping
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: t-SNE
  role: Barnes-Hut t-SNE baseline whose runtime is compared against ivis on the same
    preprocessed matrix
- name: scikit-learn
  role: Provides the Barnes-Hut t-SNE implementation and MinMaxScaler used to scale
    inputs to [0, 1]
- name: ivis
  role: Comparison algorithm; produced meaningful embeddings without subsampling in
    under 30 minutes where t-SNE exceeded 24 hours
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Execution environment for the benchmark scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:c1b2588c1b8e5ecb7d6e1e4f636331ff243f9b60ff3894ff34f814b564825588
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tsne-embedding-computation

## Summary

Compute a Barnes-Hut t-SNE embedding of a high-dimensional single-cell (or general) matrix with scikit-learn, primarily as the runtime/scale baseline against which the ivis algorithm's scalability is measured.

## When to use

Use when benchmarking dimensionality-reduction scalability on a large scRNA-seq or CyTOF matrix (e.g. the 1.3M-cell 10X Genomics mouse brain dataset), or when a t-SNE baseline embedding is required for comparison against ivis.

## When NOT to use

- When the goal is to embed millions of observations within practical time — the paper shows Barnes-Hut t-SNE did not finish within 24 hours on 1.3M cells; use ivis instead
- When embedding quality under random noise matters — t-SNE was shown to yield spurious clusters with clean boundaries in the presence of random noise
- When the downstream task requires adding new data points to an existing embedding — t-SNE lacks a transform method, unlike ivis's parametric mapping

## Inputs

- High-dimensional expression matrix (e.g. 1.3M-cell 10X Genomics mouse brain scRNA-seq dataset)
- Min-max scaled version of the matrix (values in [0, 1], via sklearn MinMaxScaler)
- Wall-clock timeout budget (e.g. 24 hours)

## Outputs

- Low-dimensional (2D) t-SNE embedding if the run completes
- Runtime benchmark record per algorithm (e.g. runtime_benchmark_results.csv with ivis and Barnes-Hut t-SNE rows)
- Timeout / non-completion status for runs exceeding the time limit

## How to apply

Preprocess the expression matrix by scaling all values to the [0, 1] range with sklearn's MinMaxScaler, following the paper's preprocessing convention. Run scikit-learn's Barnes-Hut t-SNE (sklearn.manifold.TSNE) on the same preprocessed matrix used for ivis, imposing a hard wall-clock timeout (24 hours in the 10X Genomics mouse brain benchmark) and recording non-completion if the limit is exceeded. In the article's benchmark, Barnes-Hut t-SNE did not finish within 24 hours on 1.3 million cells while ivis completed in under 30 minutes without subsampling, demonstrating linear scaling for ivis versus t-SNE's practical limit at millions of observations. Record both runtimes (e.g. to runtime_benchmark_results.csv) with one row per algorithm so the outcome can be validated.

## Related tools

- **t-SNE** (Barnes-Hut t-SNE baseline whose runtime is compared against ivis on the same preprocessed matrix)
- **scikit-learn** (Provides the Barnes-Hut t-SNE implementation and MinMaxScaler used to scale inputs to [0, 1])
- **ivis** (Comparison algorithm; produced meaningful embeddings without subsampling in under 30 minutes where t-SNE exceeded 24 hours) — https://github.com/beringresearch/ivis
- **Python** (Execution environment for the benchmark scripts)

## Examples

```
from sklearn.manifold import TSNE; from sklearn.preprocessing import MinMaxScaler; X = MinMaxScaler().fit_transform(X_raw); emb = TSNE().fit_transform(X)
```

## Evaluation signals

- runtime_benchmark_results.csv exists and contains a runtime row for both ivis and Barnes-Hut t-SNE
- Recorded outcome shows ivis completing in under 30 minutes while t-SNE exceeds the 24-hour limit
- Both algorithms were run on the identical MinMaxScaler-scaled matrix (values in [0, 1])
- Non-completion is recorded explicitly rather than as a missing result when the timeout is hit
- Runtime behaviour is consistent with the paper's claim that ivis scales linearly to millions of observations

## Limitations

- Barnes-Hut t-SNE did not finish within 24 hours on the 1.3-million-cell 10X dataset, so no embedding or runtime value beyond the timeout exists at that scale
- In the presence of random noise, t-SNE tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation
- No version numbers were stated for the compared software (t-SNE, SIMLR, scvis, DeepCyTOF), limiting reproducibility of the baseline

## Evidence

- [results] t-SNE timeout at 1.3M cells: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing convention: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [discussion] t-SNE spurious clusters under noise: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery"
- [abstract] ivis scalability claim: "and scales linearly to hundreds of thousands of cells"
