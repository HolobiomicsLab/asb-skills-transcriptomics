---
name: gene-expression-trajectory-visualization
description: Use when a DESC-embedded single-cell dataset (e.g. 10,878 human monocytes
  across three processing batches) needs pseudotime ordering and per-batch visualization
  of a marker gene such as S100A8 along that trajectory.
when_to_use_negative:
- Technical batch differences are larger than true biological variations — DESC is
  only expected to remove batch effects when technical differences across batches
  are smaller than biological ones
- The goal is discrete cell-type clustering accuracy (ARI) rather than pseudotemporal
  ordering and marker-gene trajectory visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Produces the batch-corrected low-dimensional embedding (X_Embeded_z) used
    as Monocle3 trajectory input
  repo: https://github.com/eleozzr/desc
- name: Monocle3
  role: Builds the trajectory on the DESC embedding and assigns pseudotime to every
    cell
- name: Scanpy
  role: Handles the AnnData object, preprocessing (e.g. sc.pp.log1p, filter_genes_dispersion),
    and storage of the embedding in adata.obsm
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC deep embedding model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:46361e0bc79242b5fabc00d32812e8a56953a40f22ab0cdd0b902e1efe8de833
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-expression-trajectory-visualization

## Summary

Build a Monocle3 pseudotime trajectory on a DESC low-dimensional embedding (X_Embeded_z in adata.obsm) and visualize a marker gene's smoothed expression over pseudotime per batch, to show pseudotemporal structure while confirming batch-effect removal in scRNA-seq data.

## When to use

Use when a DESC-embedded single-cell dataset (e.g. 10,878 human monocytes across three processing batches) needs pseudotime ordering and per-batch visualization of a marker gene such as S100A8 along that trajectory.

## When NOT to use

- Technical batch differences are larger than true biological variations — DESC is only expected to remove batch effects when technical differences across batches are smaller than biological ones
- The goal is discrete cell-type clustering accuracy (ARI) rather than pseudotemporal ordering and marker-gene trajectory visualization

## Inputs

- DESC low-dimensional embedding (X_Embeded_z stored in adata.obsm) for 10,878 human monocyte cells
- Per-cell batch labels for the three processing batches
- Raw S100A8 expression values per cell

## Outputs

- Monocle3 trajectory with a pseudotime value assigned to every cell
- Box plot of batch-mixing KL divergence values across 200 iterations for each pair of batches
- Smoothed S100A8 expression vs pseudotime curve plotted separately for each of the three batches

## How to apply

Load the DESC low-dimensional embedding (the X_Embeded_z representation stored in adata.obsm) together with per-cell batch labels and raw marker gene expression values (e.g. S100A8). Build a Monocle3 trajectory on the DESC embedding and assign a pseudotime value to every cell, so that the trajectory inherits DESC's batch-effect-removal property. Quantify pseudotime consistency across batches with the regional batch-mixing KL divergence criterion: for each pair of batches use K nearest neighbors with K = 5 × number of batches, repeat over 200 iterations with randomly chosen cells, and box-plot the resulting KL divergence values. Finally plot the smoothed expression curve of the marker gene as a function of pseudotime separately for each processing batch. Accept the result if the pseudotime KL divergences are small across all batches and the S100A8 curve increases monotonically over pseudotime in each batch.

## Related tools

- **DESC** (Produces the batch-corrected low-dimensional embedding (X_Embeded_z) used as Monocle3 trajectory input) — https://github.com/eleozzr/desc
- **Monocle3** (Builds the trajectory on the DESC embedding and assigns pseudotime to every cell)
- **Scanpy** (Handles the AnnData object, preprocessing (e.g. sc.pp.log1p, filter_genes_dispersion), and storage of the embedding in adata.obsm) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC deep embedding model)

## Examples

```
import desc; adata = desc.train(adata, dims=[512,128,32]); adata.obsm['X_Embeded_z']  # then build Monocle3 trajectory on this embedding and plot smoothed S100A8 vs pseudotime per batch
```

## Evaluation signals

- Batch-mixing KL divergence of pseudotime distributions is small across all three pairs of batches (K = 5 × number of batches, 200 random iterations)
- Box plots of the KL divergence across iterations show consistently low values, indicating similar pseudotime distributions across batches
- Smoothed S100A8 expression shows a monotone increase over pseudotime in each of the three batches
- Every cell receives a finite pseudotime value from the Monocle3 trajectory built on X_Embeded_z
- Trajectory input is the DESC embedding (adata.obsm['X_Embeded_z']), not raw or PCA-reduced expression

## Limitations

- DESC removes batch effects only when technical differences across batches are smaller than true biological variations
- Batch-effect removal by DESC is gradual through iterative self-learning, so convergence of cluster assignments (stopping tolerance 0.005) is a prerequisite for a reliable embedding
- To reproduce paper results, tensorflow must be below 2.0 with desc 2.0.3, and python 3.5.x/3.6.x is required by the desc package

## Evidence

- [results] Using the DESC representation as Monocle3 input, the estimated pseudotime for the three batches has similar distributions, indicating batch effect was successfully removed: [evidence span withheld — anchor status: not_found]
- [discussion] Batch effect removal evaluation via regional KL divergence of batch mixing: "We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence."
- [abstract] DESC reveals pseudotemporal structure: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [readme] DESC installs with tensorflow version constraint: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
