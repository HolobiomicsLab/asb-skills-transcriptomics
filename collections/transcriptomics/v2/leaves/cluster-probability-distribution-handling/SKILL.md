---
name: cluster-probability-distribution-handling
description: Use when sample-level distances between single-cell or pathomics samples
  must be computed from cell-cluster compositions, and each sample needs a normalized
  cluster-distribution vector (marginal) for optimal transport.
when_to_use_negative:
- Input samples are already represented as cluster-proportion vectors or a precomputed
  sample-level feature table — go directly to distance computation
- Samples use distinct technologies or cell-isolation methods (single nucleus vs.
  single cell, droplet vs. well based); PILOT assumes samples were collected uniformly,
  and mixing them invalidates distribution comparison
- The goal is comparing raw expression profiles rather than cell-composition distributions
  — use sample-level expression methods instead
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Estimates robust sample probability distributions via a hierarchical Dirichlet-multinomial
    model with pseudo cell counts before optimal transport
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Consumes the cluster-distribution vectors as marginals in emd2 to compute
    pairwise Wasserstein distances
- name: Seurat
  role: Pre-processing pipeline for normalization and clustering of structures that
    produces the cell-cluster assignments underlying the distributions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:8545be18592bb3e27c8c93a073e8a767a846b8cdb330d87f985bd6d4f4dd6b51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - Seurat
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cluster-probability-distribution-handling

## Summary

Prepare per-sample cluster-probability (cell-composition) distributions — e.g. Dirichlet-multinomial MAP estimates with pseudo cell counts — so they can serve as the marginals of PILOT's optimal-transport Wasserstein distance computation between single-cell samples.

## When to use

Use when sample-level distances between single-cell or pathomics samples must be computed from cell-cluster compositions, and each sample needs a normalized cluster-distribution vector (marginal) for optimal transport.

## When NOT to use

- Input samples are already represented as cluster-proportion vectors or a precomputed sample-level feature table — go directly to distance computation
- Samples use distinct technologies or cell-isolation methods (single nucleus vs. single cell, droplet vs. well based); PILOT assumes samples were collected uniformly, and mixing them invalidates distribution comparison
- The goal is comparing raw expression profiles rather than cell-composition distributions — use sample-level expression methods instead

## Inputs

- Per-sample cell-to-cluster assignments from Leiden/Seurat clustering of single-cell or pathomics data (.h5ad AnnData objects)
- Cluster median representations in PCA space (for the matching cost matrix)
- Pseudo cell count prior weights derived from the aggregate cell distribution across all samples

## Outputs

- Per-sample cluster-distribution vectors (Dirichlet-multinomial MAP estimates, each summing to 1 over the shared cluster set)
- Sample-by-cluster distribution matrix consumed as marginals by POT's emd2 to build the Wasserstein distance matrix (wasserstein_distance_matrix.csv / adata.uns['EMD'])

## How to apply

For each sample, summarize its cells into cluster proportions over the shared cluster set (from Leiden/Seurat clustering in PCA space). Robustly estimate these probability distributions with a hierarchical Dirichlet-multinomial model whose prior adds 'pseudo cell counts' weighted by the cell distribution of all samples, which stabilizes estimates for small samples. Ensure every sample's distribution sums to one and uses the identical cluster ordering, since these vectors become the marginals (pl, pq) of the earth-mover-distance problem solved with POT's emd2 against the cosine-distance cost matrix between cluster medians. Keep distributions consistent with what PILOT stores for downstream validation against adata.uns['EMD'].

## Related tools

- **PILOT** (Estimates robust sample probability distributions via a hierarchical Dirichlet-multinomial model with pseudo cell counts before optimal transport) — https://github.com/CostaLab/PILOT
- **POT library** (Consumes the cluster-distribution vectors as marginals in emd2 to compute pairwise Wasserstein distances)
- **Seurat** (Pre-processing pipeline for normalization and clustering of structures that produces the cell-cluster assignments underlying the distributions)

## Examples

```
import scanpy as sc, pilotpy as pl; adata = sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad'); pl.tl.wasserstein_distance(adata)
```

## Evaluation signals

- Every sample's cluster-distribution vector sums to 1 (a valid probability distribution) over the identical, consistently ordered cluster set
- The resulting Wasserstein distance matrix is square (n_samples x n_samples), symmetric, non-negative, with zero diagonal
- Computed EMD values are consistent with the matrix PILOT stores in adata.uns['EMD']
- Small samples do not produce degenerate (zero or extreme) distributions thanks to the Dirichlet-multinomial pseudo cell count prior

## Limitations

- PILOT assumes tissue samples were collected uniformly (similar tissue areas and same technology); analysis across distinct technologies or isolation methods is an open challenge
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence on the distribution-based predictions

## Evidence

- [discussion] Robust estimation of sample probability distributions using a hierarchical Dirichlet-multinomial model with pseudo cell counts: "This prior adds “pseudo cell counts”, which are weighted by the cell distribution of all samples."
- [discussion] Distributions as OT marginals: [evidence span withheld — anchor status: not_found]
- [discussion] Cluster medians feed the cost side of the OT problem: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [discussion] Uniform-sampling assumption: "The analysis of data from distinct technologies (single nucleus vs. single cell) or distinct single cell isolation methods (droplet based vs. well based) is still an open challenge"
- [abstract] Wasserstein distance as the OT objective: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
