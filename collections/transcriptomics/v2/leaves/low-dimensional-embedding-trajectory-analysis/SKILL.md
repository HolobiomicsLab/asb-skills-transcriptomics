---
name: low-dimensional-embedding-trajectory-analysis
description: Use when scRNA-seq data span multiple processing batches and you need
  pseudotime/trajectory inference on an embedding that suppresses technical batch
  differences (e.g. DESC X_Embeded_z feeding Monocle3).
when_to_use_negative:
- When technical batch differences are larger than true biological variation — DESC
  only removes batch effects when technical differences are smaller than biological
  ones
- When the goal is clustering accuracy assessment alone (use ARI against reference
  labels) rather than pseudotemporal/trajectory reconstruction
- When no per-cell batch labels are available, since batch-mixing KL divergence cannot
  be computed to validate the trajectory
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Produces the batch-corrected low-dimensional embedding (X_Embeded_z in adata.obsm)
    used as Monocle3 input
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (log1p, filter_genes_dispersion, scale) and loading/annotating
    the adata object with batch labels and expression values
  repo: https://github.com/theislab/scanpy
- name: Monocle3
  role: Trajectory construction and pseudotime assignment on the DESC embedding
- name: Keras
  role: Backend used by DESC to train the deep embedding model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:08fef157d87da4ef1a923af58865d6a4f61255a705d9587fb5be078d2de5f250
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Monocle3
  - Scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# low-dimensional-embedding-trajectory-analysis

## Summary

Builds trajectory/pseudotime reconstructions (e.g. Monocle3) on a deep-learning low-dimensional embedding such as DESC's X_Embeded_z representation, so that pseudotemporal structure is revealed while batch effects are removed.

## When to use

Use when scRNA-seq data span multiple processing batches and you need pseudotime/trajectory inference on an embedding that suppresses technical batch differences (e.g. DESC X_Embeded_z feeding Monocle3).

## When NOT to use

- When technical batch differences are larger than true biological variation — DESC only removes batch effects when technical differences are smaller than biological ones
- When the goal is clustering accuracy assessment alone (use ARI against reference labels) rather than pseudotemporal/trajectory reconstruction
- When no per-cell batch labels are available, since batch-mixing KL divergence cannot be computed to validate the trajectory

## Inputs

- Raw scRNA-seq UMI count matrix for 10,878 human monocyte cells with per-cell batch labels (three processing batches)
- DESC low-dimensional embedding (X_Embeded_z representation stored in adata.obsm)
- Raw per-cell expression values for a trajectory marker gene (e.g. S100A8)

## Outputs

- Per-cell pseudotime values assigned from a Monocle3 trajectory built on the DESC embedding
- Box plot of regional batch-mixing KL divergence values across 200 iterations for all batch pairs
- Smoothed marker-gene (S100A8) expression curve as a function of pseudotime, plotted separately for each batch

## How to apply

Preprocess counts with Scanpy per the DESC protocol (log1p, filter_genes_dispersion with n_top_genes=1000, scale with max_value=6), train DESC, and take the resulting low-dimensional embedding (X_Embeded_z stored in adata.obsm) as the Monocle3 input. Build a Monocle3 trajectory on the embedding (for the human monocyte dataset: 10,878 cells across three processing batches) and assign a pseudotime value to every cell. Quantify batch consistency of pseudotime using the regional batch-mixing criterion: KL divergence between pseudotime distributions of each pair of batches, with K nearest neighbors where K = 5 × number of batches, repeated over 200 iterations with randomly chosen cells, summarized as a box plot. As biological validation, plot the smoothed expression of a known pseudotemporal marker (e.g. S100A8) against pseudotime per batch and confirm a monotone increase in each batch.

## Related tools

- **DESC** (Produces the batch-corrected low-dimensional embedding (X_Embeded_z in adata.obsm) used as Monocle3 input) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (log1p, filter_genes_dispersion, scale) and loading/annotating the adata object with batch labels and expression values) — https://github.com/theislab/scanpy
- **Monocle3** (Trajectory construction and pseudotime assignment on the DESC embedding)
- **Keras** (Backend used by DESC to train the deep embedding model)

## Examples

```
import scanpy as sc; sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)
```

## Evaluation signals

- Batch-mixing KL divergence of pseudotime distributions (K = 5 × number of batches, 200 iterations) is small across all three batches
- Smoothed S100A8 expression curve shows a monotone increase over pseudotime within each of the three batches
- Pseudotime distributions are visually similar across the three processing batches (box plots of per-iteration KL values overlap)
- The DESC embedding X_Embeded_z is present in adata.obsm for all 10,878 cells and every cell receives a finite pseudotime value

## Limitations

- DESC batch-effect removal works only as long as technical differences across batches are smaller than true biological variations
- Pairwise-based correctors (CCA/MNN/Seurat 3.0/Scanorama) make results order-dependent; DESC was chosen to avoid this, but the KL criterion itself is evaluated pairwise
- Software version numbers for Keras/Scanpy are unstated in the article; desc requires Python 3.5.x/3.6.x and TensorFlow <2.0 (desc 2.0.3) to reproduce paper results

## Evidence

- [abstract] pseudotemporal structure via DESC soft clustering: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [discussion] KL divergence batch-mixing evaluation: "We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence."
- [abstract] DESC removes batch effects through iterative self-learning: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [methods] Scanpy filter steps from methods: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [readme] DESC soft clustering capabilities from README: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence"
