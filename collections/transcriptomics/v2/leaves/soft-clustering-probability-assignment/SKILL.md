---
name: soft-clustering-probability-assignment
description: Use when clustering scRNA-seq data with DESC and you need per-cell cluster
  assignment probabilities to identify high-confidence cells or reveal pseudotemporal
  structure.
when_to_use_negative:
- When hard cluster labels alone suffice and no confidence measure or pseudotemporal
  interpretation of assignments is needed
- When batch effects are larger than true biological variation — DESC's self-learning
  removes batch effects only 'as long as technical differences across batches are
  smaller than true biological variations'
- When a reference-based or supervised annotation is required instead of unsupervised
  clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: desc
  role: 'Implements the iterative self-training loop: computes Student''s t-kernel
    soft assignments Q, builds auxiliary target distribution P, and minimizes KL(P||Q)
    to produce cluster-specific probabilities'
  repo: https://github.com/eleozzr/desc
- name: SCANPY
  role: Provides sc.datasets.paul15() data loading and preprocessing functions (sc.pp.log1p,
    sc.pp.filter_genes_dispersion, sc.pp.scale) and AnnData storage of the soft-clustering
    results
  repo: https://github.com/theislab/scanpy
- name: TensorFlow
  role: Backend used to train the autoencoder and run the KL-divergence-based iterative
    clustering (desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*)
- name: Matplotlib
  role: Visualization of clustering results, soft assignment probabilities, and t-SNE
    coordinates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:2e5f912aab6fa2b7b6f62fc36c151a8a56f2881a231b8cb54539c773755f4478
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - Matplotlib
  - NumPy
  - SCANPY
  - TensorFlow
  - desc
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# soft-clustering-probability-assignment

## Summary

Assign each cell cluster-specific probabilities via DESC's soft clustering, in which soft assignments Q from a Student's t kernel are iteratively refined against an auxiliary target distribution P by minimizing KL divergence, yielding biologically interpretable confidence values for cluster membership.

## When to use

Use when clustering scRNA-seq data with DESC and you need per-cell cluster assignment probabilities to identify high-confidence cells or reveal pseudotemporal structure.

## When NOT to use

- When hard cluster labels alone suffice and no confidence measure or pseudotemporal interpretation of assignments is needed
- When batch effects are larger than true biological variation — DESC's self-learning removes batch effects only 'as long as technical differences across batches are smaller than true biological variations'
- When a reference-based or supervised annotation is required instead of unsupervised clustering

## Inputs

- Preprocessed scRNA-seq AnnData object (e.g. Paul et al. 2015 myeloid progenitor data from sc.datasets.paul15(), log1p-transformed, top 1000 highly variable genes via sc.pp.filter_genes_dispersion, scaled with sc.pp.scale(max_value=6))
- Low-dimensional embedding of cells from the autoencoder bottleneck layer
- Initialized cluster centroids from Louvain clustering on bottleneck features

## Outputs

- Per-cell soft-clustering cluster assignment probabilities
- Discrete cluster labels stabilized at tol=0.005 assignment-change threshold
- AnnData object with soft-clustering probabilities, X_Embeded_z representation, and t-SNE coordinates saved to .h5ad (e.g. paul_result/adata_desc.h5ad)

## How to apply

After pretraining a stacked autoencoder (e.g. encoder dims [n_genes, 64, 32]) and initializing cluster centers by Louvain clustering on the bottleneck-layer features (n_neighbors=10, resolutions 0.8 and 1.0), compute soft assignments Q of embedded cells to centroids using a Student's t distribution kernel. Construct the auxiliary target distribution P that assigns higher probabilities to easy-to-cluster cells near cluster centroids, and iteratively minimize the KL(P||Q) loss (batch_size=256) until the proportion of cells changing cluster assignment between consecutive iterations drops below tol=0.005. Store the resulting soft-clustering probabilities in the AnnData object (obs/uns/obsm slots) alongside the embedded representation X_Embeded_z and t-SNE coordinates (learning_rate=200), then save as .h5ad (e.g. paul_result/adata_desc.h5ad). Rationale: the t-distribution kernel was found more robust than a Gaussian kernel, and the soft probabilities are what make DESC's assignments biologically interpretable and capable of revealing discrete and pseudotemporal structure.

## Related tools

- **desc** (Implements the iterative self-training loop: computes Student's t-kernel soft assignments Q, builds auxiliary target distribution P, and minimizes KL(P||Q) to produce cluster-specific probabilities) — https://github.com/eleozzr/desc
- **SCANPY** (Provides sc.datasets.paul15() data loading and preprocessing functions (sc.pp.log1p, sc.pp.filter_genes_dispersion, sc.pp.scale) and AnnData storage of the soft-clustering results) — https://github.com/theislab/scanpy
- **TensorFlow** (Backend used to train the autoencoder and run the KL-divergence-based iterative clustering (desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*))
- **Matplotlib** (Visualization of clustering results, soft assignment probabilities, and t-SNE coordinates)

## Examples

```
import scanpy as sc; import desc; adata = sc.datasets.paul15(); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); adata = desc.train(adata, dims=[adata.shape[1], 64, 32], tol=0.005, batch_size=256); adata.write('paul_result/adata_desc.h5ad')
```

## Evaluation signals

- The iterative optimization terminates only when the proportion of cells changing cluster assignment between two consecutive steps is below tol=0.005
- Soft assignment probabilities should be biologically consistent: for the Paul et al. (2015) myeloid progenitor data, derived clusters should recover cell types from the paul15_clusters annotation, evaluable by ARI
- Cluster assignment probabilities should reveal both discrete and pseudotemporal structure when projected onto t-SNE coordinates (learning_rate=200)
- The AnnData output must contain soft-clustering probabilities in obs, X_Embeded_z and t-SNE coordinates in obsm/uns, and serialize to a valid .h5ad file
- Batch mixing should improve relative to raw space, since DESC's iterative self-learning gradually removes batch effects (measurable by regional KL divergence of batch mixing with K = 5 × number of batches over 200 iterations)

## Limitations

- Batch effect removal works only 'as long as technical differences across batches are smaller than true biological variations'
- Gaussian distribution as a kernel was found less robust than the t distribution (Supplementary Note 3)
- To reproduce paper results, tensorflow must be lower than 2.0 (desc 2.0.3); desc requires Python 3.5.x or 3.6.x
- Supplementary Table 2 (default hidden layers/nodes) and Table 3 (autoencoder hyperparameters) defaults were not stated in the main text

## Evidence

- [abstract] As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [discussion] we compute a soft assignment of each cell between the embedded points and the cluster centroids. Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel: "we compute a soft assignment of each cell between the embedded points and the cluster centroids. Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel"
- [discussion] we define the objective function as a KL divergence loss between the soft cell assignments q_i and the auxiliary distribution p_i: [evidence span withheld — anchor status: not_found]
- [discussion] This iterative procedure stops when the proportion of cells that changes cluster assignment between two consecutive steps is less than tol.: [evidence span withheld — anchor status: not_found]
- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
