---
name: variational-generative-model-training-scrna-seq
description: Use when scRNA-seq query data must be projected into a batch-corrected
  low-dimensional embedding trained on a reference dataset (e.g. h5ad inputs via Cell_BLAST.data),
  and cell-to-cell similarity must be scored for cell-type querying.
when_to_use_negative:
- The reference dataset has fewer than ~1000 cells — the article notes datasets with
  fewer than 1000 cells may suffer from under-training issues (3000 cells generally
  sufficient)
- The task is simple dimensionality reduction without cell querying — the embedding
  is purpose-built for the NPD cell-to-cell similarity metric and query-based annotation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Main package providing Cell_BLAST.data for h5ad loading and the DIRECTi variational
    generative model with the NPD similarity metric
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained via Cell_BLAST.directi.fit_DIRECTi (default
    RMSpropOptimizer) to produce the adversarial variational posterior embedding
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Canonical Gaussian-diagonal variational autoencoder baseline trained on the
    same reference data for posterior comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:c372d35a79e5a7b49c122e036c68b1097187de51249129367c0e26c4c5e6fd2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# variational-generative-model-training-scrna-seq

## Summary

Train a neural network-based variational generative model (DIRECTi, or scVI as the canonical Gaussian-diagonal VAE baseline) on scRNA-seq reference data to obtain unbiased low-dimensional cell embeddings, and use posterior-based distance metrics to quantify cell-to-cell similarity for querying.

## When to use

Use when scRNA-seq query data must be projected into a batch-corrected low-dimensional embedding trained on a reference dataset (e.g. h5ad inputs via Cell_BLAST.data), and cell-to-cell similarity must be scored for cell-type querying.

## When NOT to use

- The reference dataset has fewer than ~1000 cells — the article notes datasets with fewer than 1000 cells may suffer from under-training issues (3000 cells generally sufficient)
- The task is simple dimensionality reduction without cell querying — the embedding is purpose-built for the NPD cell-to-cell similarity metric and query-based annotation

## Inputs

- Reference scRNA-seq dataset in h5ad format
- Query scRNA-seq dataset in h5ad format (e.g. independent pancreatic benchmark query data)
- Cell-type annotations on the reference cells for ROC/AUC evaluation

## Outputs

- Trained DIRECTi model on the reference data
- Trained scVI baseline model on the same reference data
- Low-dimensional embeddings for query and reference cells
- Nearest-neighbor query-reference cell pairs with NPD and Euclidean distances
- ROC curves and AUC values comparing NPD vs Euclidean distance

## How to apply

Load query and reference scRNA-seq datasets in h5ad format with the Cell_BLAST.data module, then fit a DIRECTi model on the reference data via Cell_BLAST.directi.fit_DIRECTi (using the default RMSpropOptimizer per the migration guide) in a pip-installed Cell-BLAST conda environment (conda create -n cb python=3.9). Train scVI on the same reference data as the canonical Gaussian-diagonal variational autoencoder baseline. Retrieve nearest-neighbor pairs between query and reference cells in both the DIRECTi and scVI low-dimensional embeddings, and score each pair with the posterior-based normalized projection distance (NPD) alongside the plain Euclidean distance between embedding vectors. Build ROC curves and compute AUC for cell-type querying under NPD versus Euclidean distance to validate that the variational posterior improves similarity estimation. Rationale: DIRECTi explicitly models multilevel batch effect and uncertainty, whereas the canonical scVI-style posterior enforces a diagonal-covariance Gaussian.

## Related tools

- **Cell BLAST** (Main package providing Cell_BLAST.data for h5ad loading and the DIRECTi variational generative model with the NPD similarity metric) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained via Cell_BLAST.directi.fit_DIRECTi (default RMSpropOptimizer) to produce the adversarial variational posterior embedding) — github.com/gao-lab/Cell_BLAST
- **scVI** (Canonical Gaussian-diagonal variational autoencoder baseline trained on the same reference data for posterior comparison)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- The NPD-based ROC curve dominates the Euclidean-distance ROC curve (higher AUC), consistent with the reported NPD superiority with the adversarial variational posterior (Supplementary Figs. 6g-h, 8c)
- The AUC improvement of the DIRECTi variational posterior over the scVI canonical Gaussian-diagonal posterior is confirmed in a ten-dimensional embedding with independent query data
- import Cell_BLAST as cb succeeds and cb.__version__ prints in the conda environment
- Nearest-neighbor pairs are retrieved for query cells against reference cells in both DIRECTi and scVI embedding spaces before distance computation

## Limitations

- The model, like all neural network-based models, is over-parameterized and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [abstract] Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric.: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [discussion] the canonical variational autoencoder model used by scVI and several other tools enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior.: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [methods] pip install Cell-BLAST: "pip install Cell-BLAST"
- [methods] cb.data.h5_to_h5ad(inputfilename, outputfilename): "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [readme] To reproduce results, please check out the `rep` branch.: [evidence span withheld — anchor status: external_doc]
