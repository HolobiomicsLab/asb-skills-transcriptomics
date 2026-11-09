---
name: single-cell-pseudotime-trajectory-inference
description: Use when scRNA-seq data from multiple processing batches (e.g. the 10,878-cell
  human monocyte dataset) must be ordered along pseudotime without batch effects confounding
  the trajectory.
when_to_use_negative:
- Technical differences across batches are larger than true biological variations
  — DESC's iterative self-learning may then fail to remove batch effects
- The analysis goal is purely discrete cell-type clustering with no temporal ordering,
  where soft-clustering probabilities alone suffice
- The user wants batch correction order-dependent pairwise methods (CCA, MNN, Seurat
  3.0, Scanorama) rather than DESC's simultaneous all-batch embedding
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DESC
  role: Produces the batch-corrected low-dimensional embedding (X_Embeded_z) and soft
    cluster assignment probabilities that reveal pseudotemporal structure, used as
    Monocle3 input
  repo: https://github.com/eleozzr/desc
- name: Monocle3
  role: Builds the trajectory on the DESC embedding and assigns pseudotime to every
    cell
- name: Scanpy
  role: Preprocessing before DESC training (sc.pp.log1p, sc.pp.filter_genes_dispersion
    with n_top_genes=1000, sc.pp.scale with max_value=6) and holding the embedding
    in adata.obsm
  repo: https://github.com/scverse/scanpy
- name: Keras
  role: Backend used to train the DESC deep embedding model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:43bf310938df631f4477ef265d6b1d89c2453e2995985c7b123f892a1b708f8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Monocle3
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-pseudotime-trajectory-inference

## Summary

Reconstruct pseudotemporal cell trajectories by feeding the DESC low-dimensional embedding (X_Embeded_z in adata.obsm) into Monocle3, leveraging DESC's soft clustering probabilities that reveal pseudotemporal structure while removing batch effects.

## When to use

Use when scRNA-seq data from multiple processing batches (e.g. the 10,878-cell human monocyte dataset) must be ordered along pseudotime without batch effects confounding the trajectory.

## When NOT to use

- Technical differences across batches are larger than true biological variations — DESC's iterative self-learning may then fail to remove batch effects
- The analysis goal is purely discrete cell-type clustering with no temporal ordering, where soft-clustering probabilities alone suffice
- The user wants batch correction order-dependent pairwise methods (CCA, MNN, Seurat 3.0, Scanorama) rather than DESC's simultaneous all-batch embedding

## Inputs

- Raw scRNA-seq UMI count matrix (e.g. 10,878-cell human monocyte dataset) with per-cell batch labels
- DESC low-dimensional embedding (X_Embeded_z) stored in adata.obsm
- Raw expression values of a temporal marker gene (e.g. S100A8)

## Outputs

- Per-cell pseudotime values from a Monocle3 trajectory built on the DESC embedding
- Box plots of regional batch-mixing KL divergence of pseudotime distributions across 200 iterations
- Smoothed S100A8 expression curves as a function of pseudotime for each of the three batches

## How to apply

First preprocess the raw counts (log1p, select 1000 highly variable genes with sc.pp.filter_genes_dispersion, scale with max_value=6), then train DESC so that its bottleneck layer yields the low-dimensional embedding stored in adata.obsm['X_Embeded_z']. Build a Monocle3 trajectory on this DESC representation and assign a pseudotime value to every cell. Quantify batch consistency by computing the regional KL divergence between pseudotime distributions of each batch pair (K nearest neighbors with K = 5 × number of batches, 200 iterations with randomly chosen cells), and report box plots of the divergence values. Validate that batch effect was removed by confirming similar pseudotime distributions across the three batches and a monotone increase of a marker gene (S100A8) along pseudotime in each batch.

## Related tools

- **DESC** (Produces the batch-corrected low-dimensional embedding (X_Embeded_z) and soft cluster assignment probabilities that reveal pseudotemporal structure, used as Monocle3 input) — https://github.com/eleozzr/desc
- **Monocle3** (Builds the trajectory on the DESC embedding and assigns pseudotime to every cell)
- **Scanpy** (Preprocessing before DESC training (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale with max_value=6) and holding the embedding in adata.obsm) — https://github.com/scverse/scanpy
- **Keras** (Backend used to train the DESC deep embedding model)

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); desc.train(adata, dims=[512,128,32])
```

## Evaluation signals

- Regional batch-mixing KL divergence of pseudotime distributions is small across all three batches (K = 5 × number of batches, 200 random iterations)
- Box plots of per-iteration KL divergence values show low spread across the 200 iterations
- S100A8 expression curve increases monotonically over pseudotime separately within each of the three batches
- DESC iterative training converges: proportion of cells changing cluster assignment between consecutive steps drops below tol = 0.005
- Pseudotime distributions are statistically similar across batches after using the DESC representation as Monocle3 input

## Limitations

- DESC batch-effect removal works only as long as technical differences across batches are smaller than true biological variations
- The t distribution is used as the soft-assignment kernel because a Gaussian kernel was found to be less robust (Supplementary Note 3)
- Pairwise-based correction methods (CCA, MNN, Seurat 3.0, Scanorama) are order-dependent; the DESC embedding approach avoids this but requires DESC training
- DESC package versions are tied to TensorFlow releases: desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*; reproducing paper results requires tensorflow < 2.0 and python 3.5.x/3.6.x

## Evidence

- [abstract] As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [discussion] Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel: "Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel"
- [discussion] We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence.: "We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence."
- [readme] For  `tensorflow 1*`, we released `desc(2.0.3)`. Please see our jupyter notebook example [desc_2.0.3_paul.ipynb](./desc_2.0.3_paul.ipynb): "For  `tensorflow 1*`, we released `desc(2.0.3)`. Please see our jupyter notebook example [desc_2.0.3_paul.ipynb](./desc_2.0.3_paul.ipynb)"
- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
