---
name: roc-curve-auc-evaluation
description: Use when you must decide whether a model-specific cell-to-cell similarity
  metric (e.g. NPD from a variational posterior) discriminates same-type query/reference
  neighbor pairs better than plain Euclidean distance in a learned embedding.
when_to_use_negative:
- The embedding provides only point estimates with no variational posterior (NPD is
  undefined, so only the Euclidean-distance ROC/AUC arm can be computed)
- The goal is cell querying itself rather than evaluating the similarity metric —
  use Cell BLAST's querying workflow instead of the metric benchmarking ROC analysis
- The reference dataset has fewer than ~1000 cells, where the over-parameterized DIRECTi
  model suffers from under-training and the metric comparison would be confounded
  by a poorly fit embedding
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Cell BLAST
  role: Provides the customized cell-to-cell similarity metric (NPD) and data module
    for loading h5ad query/reference datasets
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Neural network-based generative model whose adversarial variational posterior
    supplies the posterior distributions used to compute NPD and the low-dimensional
    embedding
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Canonical Gaussian-diagonal variational autoencoder baseline embedding against
    which the NPD-vs-Euclidean AUC improvement is contrasted
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:f66a7483907815bc831c3b3053f96f2dd63e00f77087048b25ea181466368d2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# roc-curve-auc-evaluation

## Summary

Benchmark whether Cell BLAST's posterior-based normalized projection distance (NPD) outperforms Euclidean distance for judging whether nearest reference cells match a query cell's type, by building ROC curves and comparing AUCs for both metrics in the DIRECTi and scVI embedding spaces.

## When to use

Use when you must decide whether a model-specific cell-to-cell similarity metric (e.g. NPD from a variational posterior) discriminates same-type query/reference neighbor pairs better than plain Euclidean distance in a learned embedding.

## When NOT to use

- The embedding provides only point estimates with no variational posterior (NPD is undefined, so only the Euclidean-distance ROC/AUC arm can be computed)
- The goal is cell querying itself rather than evaluating the similarity metric — use Cell BLAST's querying workflow instead of the metric benchmarking ROC analysis
- The reference dataset has fewer than ~1000 cells, where the over-parameterized DIRECTi model suffers from under-training and the metric comparison would be confounded by a poorly fit embedding

## Inputs

- Query scRNA-seq dataset in h5ad format (pancreatic benchmark)
- Reference scRNA-seq dataset in h5ad format with cell type annotations
- DIRECTi embedding of the reference data (e.g. ten-dimensional latent space) with variational posterior
- scVI embedding of the same reference data (Gaussian-diagonal VAE baseline)
- Query-reference nearest-neighbor pairs with cell type labels

## Outputs

- ROC curves for cell-type querying under NPD versus Euclidean distance
- AUC values for each metric-model combination
- Comparison figures reproducing Supplementary Figs. 6g-h and 8c

## How to apply

Load the pancreatic benchmark query and reference scRNA-seq datasets in h5ad format via the Cell_BLAST.data module, then train a DIRECTi model on the reference with Cell_BLAST.directi.fit_DIRECTi and a scVI model on the same reference as the canonical Gaussian-diagonal variational autoencoder baseline. Retrieve nearest-neighbor pairs between query cells and reference cells in both the DIRECTi and scVI low-dimensional embeddings. For each pair, compute the posterior-based normalized projection distance (NPD) as well as the Euclidean distance between embedding vectors. Build ROC curves treating same-type nearest-neighbor hits as positives and compute AUC for NPD versus Euclidean distance under each model, mirroring the Supplementary Figs. 6g-h and 8c comparison. The analysis is validated when the NPD-based ROC curve dominates the Euclidean-distance curve (higher AUC), particularly for the adversarial variational posterior of DIRECTi.

## Related tools

- **Cell BLAST** (Provides the customized cell-to-cell similarity metric (NPD) and data module for loading h5ad query/reference datasets) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Neural network-based generative model whose adversarial variational posterior supplies the posterior distributions used to compute NPD and the low-dimensional embedding) — github.com/gao-lab/Cell_BLAST
- **scVI** (Canonical Gaussian-diagonal variational autoencoder baseline embedding against which the NPD-vs-Euclidean AUC improvement is contrasted) — github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb"
```

## Evaluation signals

- The NPD-based ROC curve dominates the Euclidean-distance ROC curve (higher AUC) under the DIRECTi adversarial variational posterior
- The AUC improvement of NPD over Euclidean distance is confirmed in a ten-dimensional embedding with independent query data
- The comparison reproduces Supplementary Figs. 6g-h (DIRECTi) and 8c (scVI baseline)
- All computed AUC values fall within the valid [0, 1] range and nearest-neighbor pairs are one-to-one between query and reference cells
- The scVI baseline behaves as the canonical Gaussian-diagonal VAE (posterior covariance restricted to diagonal), so its NPD-vs-Euclidean gap is smaller than for DIRECTi

## Limitations

- scVI's variational posterior enforces a Gaussian distribution with diagonal-covariance matrices, which weakens the posterior-based distance and bounds the expected NPD advantage on that baseline
- The Cell BLAST model is over-parameterized like all neural network-based models and requires a reasonable number of samples to train properly; ~3000 cells is generally sufficient, while datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition, which could distort neighbor-pair ground truth

## Evidence

- [other] NPD superiority with the adversarial variational posterior is confirmed in a ten-dimensional embedding with independent query data: [evidence span withheld — anchor status: not_found]
- [discussion] scVI Gaussian-diagonal posterior baseline: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [discussion] Under-training threshold for reference training: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
- [readme] Installation and package import from README: [evidence span withheld — anchor status: external_doc]
- [readme] Cell-to-cell similarity purpose from README: "For each query cell, it searches for most similar cells in the reference database."
