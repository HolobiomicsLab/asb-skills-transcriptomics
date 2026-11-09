---
name: posterior-distribution-distance-computation
description: Use when querying scRNA-seq cells against a reference embedding (e.g.,
  DIRECTi or scVI latent space) and you need a model-aware similarity metric that
  accounts for posterior uncertainty rather than Euclidean distance between embedding
  vectors.
when_to_use_negative:
- The embedding model does not provide a variational posterior (e.g., a plain deterministic
  dimensionality reduction), leaving no posterior distributions to compare — Euclidean
  distance in the latent space is then the only option.
- The goal is generic dimensionality reduction or visualization rather than cell-querying
  similarity, where posterior-based distances are not meaningful.
- The reference dataset is too small to train the model properly — the paper notes
  datasets with fewer than 1000 cells may suffer from under-training issues.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: Provides the DIRECTi generative model, embedding, and the NPD cell-to-cell
    similarity metric used for cell querying
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model whose adversarial variational posterior supplies the
    distributions compared by NPD
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Canonical Gaussian-diagonal variational autoencoder trained on the same reference
    data as a baseline embedding for the NPD vs Euclidean comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:349399471c4e72e1e1d93c60372970cb3b2f89039afd8a87cf390f37d2ef4576
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# posterior-distribution-distance-computation

## Summary

Computes cell-to-cell similarity in Cell BLAST's low-dimensional embedding using the posterior-based normalized projection distance (NPD) instead of plain Euclidean distance, leveraging the variational posterior of the DIRECTi generative model to capture uncertainty in cell-to-cell similarity estimation.

## When to use

Use when querying scRNA-seq cells against a reference embedding (e.g., DIRECTi or scVI latent space) and you need a model-aware similarity metric that accounts for posterior uncertainty rather than Euclidean distance between embedding vectors.

## When NOT to use

- The embedding model does not provide a variational posterior (e.g., a plain deterministic dimensionality reduction), leaving no posterior distributions to compare — Euclidean distance in the latent space is then the only option.
- The goal is generic dimensionality reduction or visualization rather than cell-querying similarity, where posterior-based distances are not meaningful.
- The reference dataset is too small to train the model properly — the paper notes datasets with fewer than 1000 cells may suffer from under-training issues.

## Inputs

- Query scRNA-seq dataset in h5ad format
- Reference scRNA-seq dataset in h5ad format (e.g., pancreatic benchmark data)
- Trained DIRECTi variational posterior (per-cell posterior distribution parameters in the low-dimensional embedding)

## Outputs

- Per-pair NPD (posterior-based normalized projection distance) values
- Per-pair Euclidean distances between embedding vectors
- ROC curves and AUC values comparing NPD vs Euclidean distance for cell-type querying

## How to apply

Train a DIRECTi model on the reference scRNA-seq data (h5ad format loaded via Cell_BLAST.data; fit via Cell_BLAST.directi.fit_DIRECTi, which uses the default RMSpropOptimizer), optionally training a scVI Gaussian-diagonal VAE as a baseline embedding. Retrieve nearest-neighbor pairs between query cells and reference cells in the low-dimensional embedding. For each query-hit pair, compute the distance between the two cells' posterior distributions (NPD) as well as the Euclidean distance between embedding vectors. Evaluate the two metrics by building ROC curves and computing AUC for the task of distinguishing whether nearest reference cells are of the same cell type as the query; the NPD-based ROC should dominate the Euclidean one, especially with Cell BLAST's adversarial (non-Gaussian-diagonal) variational posterior, reproducing Supplementary Figs. 6g-h and 8c in a ten-dimensional embedding with independent query data.

## Related tools

- **Cell BLAST** (Provides the DIRECTi generative model, embedding, and the NPD cell-to-cell similarity metric used for cell querying) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model whose adversarial variational posterior supplies the distributions compared by NPD) — github.com/gao-lab/Cell_BLAST
- **scVI** (Canonical Gaussian-diagonal variational autoencoder trained on the same reference data as a baseline embedding for the NPD vs Euclidean comparison)

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(cb.data.read_h5ad('reference.h5ad')); pred = cb.blast.blast(model, cb.data.read_h5ad('query.h5ad'), metric='npd')
```

## Evaluation signals

- The NPD-based ROC curve should dominate (have higher AUC than) the Euclidean-distance ROC curve for distinguishing same-type nearest reference cells, consistent with Supplementary Figs. 6g-h.
- The AUC improvement with Cell BLAST's adversarial variational posterior should be confirmed in a ten-dimensional embedding using independent query data (Supplementary Fig. 8c).
- Nearest-neighbor pairs retrieved in the DIRECTi and scVI embeddings should be identical sets regardless of which distance metric is used for retrieval bookkeeping, so metric comparison is apples-to-apples.
- The trained DIRECTi model should be trained on the reference data only (default RMSpropOptimizer), with query cells projected into the shared embedding.

## Limitations

- The NPD advantage depends on the variational posterior; the canonical Gaussian-diagonal posterior (as in scVI) yields less of the reported benefit than Cell BLAST's adversarial posterior.
- The DIRECTi model is over-parameterized and requires a reasonable number of samples to train properly; small datasets (<1000 cells) may suffer under-training, degrading the posterior estimates NPD relies on.

## Evidence

- [discussion] Cell BLAST is an accurate and robust querying algorithm: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [discussion] NPD advantage tied to adversarial posterior: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [methods] Model fitting entry point: "import Cell_BLAST as cb"
- [readme] Installation and import verification: "pip install Cell-BLAST"
