---
name: kl-divergence-self-training-optimization
description: Use when clustering scRNA-seq data with a pretrained deep embedding (e.g.,
  stacked autoencoder bottleneck features) and you need to iteratively refine soft
  cluster assignments until the fraction of cells changing clusters falls below a
  tolerance.
when_to_use_negative:
- Technical batch differences are larger than true biological variations — DESC's
  gradual batch-effect removal only holds when 'technical differences across batches
  are smaller than true biological variations'.
- You need hard, non-iterative clustering without a pretrained embedding — alternative
  methods such as SC3 consensus clustering or pairwise batch-correction tools (Seurat
  3.0 CCA/MNN, Scanorama) address different setups.
- Input is raw UMI counts without the required cell-level (counts/total × 10,000,
  log1p) and gene-level standardization preprocessing.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: desc
  role: 'Implements the deep embedding algorithm: pretraining, Student''s t soft assignment,
    auxiliary target distribution P, and KL-divergence self-training loop; desc 2.0.3
    for tensorflow 1*, desc 2.1.1 for tensorflow 2*'
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing of Paul15 data (sc.pp.log1p, sc.pp.filter_genes_dispersion with
    n_top_genes=1000, sc.pp.scale max_value=6) before the self-training optimization
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC neural network model during KL-loss optimization
- name: TensorFlow
  role: Deep learning framework underlying desc; version must be <2.0 to reproduce
    paper results (desc 2.0.3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:beccbef3146a1a79c8cb490ddab64fe42574eb4cdcb957c4cf08208a96179484
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - Keras
  - Matplotlib
  - NumPy
  - Scanpy
  - TensorFlow
  - desc
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# KL-divergence self-training optimization

## Summary

Iterative self-training scheme from DESC that refines cluster assignments by minimizing the Kullback–Leibler divergence between a soft assignment distribution Q (Student's t-kernel over embedded cells and centroids) and an auxiliary target distribution P, learning cluster-specific representations while ignoring irrelevant signals such as batch effect.

## When to use

Use when clustering scRNA-seq data with a pretrained deep embedding (e.g., stacked autoencoder bottleneck features) and you need to iteratively refine soft cluster assignments until the fraction of cells changing clusters falls below a tolerance.

## When NOT to use

- Technical batch differences are larger than true biological variations — DESC's gradual batch-effect removal only holds when 'technical differences across batches are smaller than true biological variations'.
- You need hard, non-iterative clustering without a pretrained embedding — alternative methods such as SC3 consensus clustering or pairwise batch-correction tools (Seurat 3.0 CCA/MNN, Scanorama) address different setups.
- Input is raw UMI counts without the required cell-level (counts/total × 10,000, log1p) and gene-level standardization preprocessing.

## Inputs

- Pretrained stacked-autoencoder bottleneck embedding of the gene expression matrix (X_Embeded_z) after Scanpy preprocessing (sc.pp.log1p, filter_genes_dispersion n_top_genes=1000, sc.pp.scale max_value=6)
- Initial cluster centroids from Louvain clustering on bottleneck-layer features
- Paul et al. (2015) myeloid progenitor dataset (sc.datasets.paul15) with paul15_clusters labels for benchmarking

## Outputs

- Soft-clustering probabilities per cell (cluster assignment probabilities)
- Converged low-dimensional embedded representation (X_Embeded_z) stored in adata.obsm
- t-SNE coordinates (learning_rate=200) and final cluster assignments saved to paul_result/adata_desc.h5ad

## How to apply

After pretraining a stacked autoencoder (encoder dims [n_genes, 64, 32]) and initializing cluster centers with Louvain clustering (n_neighbors=10, resolutions 0.8 and 1.0) on the bottleneck features, compute soft assignments Q of each embedded cell to centroids using the Student's t distribution kernel, following van der Maaten and Hinton. Construct the auxiliary target distribution P, which assigns higher probabilities to easy-to-cluster cells near centroids, and iteratively minimize the KL(P||Q) loss with batch_size=256, updating P, cluster centers, and encoder parameters. Stop when the proportion of cells changing cluster assignment between two consecutive iterations is below tol=0.005. The rationale is that DESC thereby learns cluster-specific information while ignoring irrelevant signals such as batch effect, provided technical differences are smaller than true biological variation. Use the t distribution rather than a Gaussian kernel, which was found less robust (Supplementary Note 3).

## Related tools

- **desc** (Implements the deep embedding algorithm: pretraining, Student's t soft assignment, auxiliary target distribution P, and KL-divergence self-training loop; desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing of Paul15 data (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale max_value=6) before the self-training optimization) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC neural network model during KL-loss optimization)
- **TensorFlow** (Deep learning framework underlying desc; version must be <2.0 to reproduce paper results (desc 2.0.3))

## Examples

```
import desc; adata = desc.train(adata, dims=[adata.n_vars, 64, 32], tol=0.005, batch_size=256, louvain_resolution=[0.8, 1.0], n_neighbors=10, save_dir='paul_result'); adata.write('paul_result/adata_desc.h5ad')
```

## Evaluation signals

- Self-training convergence: the proportion of cells changing cluster assignment between two consecutive iterations drops below tol=0.005.
- Clustering accuracy: ARI computed against the reference paul15_clusters cell-type labels; larger ARI indicates higher clustering accuracy.
- Soft assignment sanity: cluster assignment probabilities are biologically interpretable and reveal both discrete and pseudotemporal structure of cells.
- Batch mixing: regional KL divergence of batch mixing evaluated with K = 5 × number of batches nearest neighbors over 200 iterations.
- Reproducibility: with tensorflow <2.0 and desc 2.0.3, results should match the paper's Paul dataset example (paul_result/adata_desc.h5ad).

## Limitations

- DESC only gradually removes batch effects when technical differences across batches are smaller than true biological variations.
- The Gaussian kernel alternative was evaluated and found less robust than the Student's t distribution (Supplementary Note 3).
- Reproducing paper results requires tensorflow <2.0 and python 3.5.x/3.6.x; desc 2.0.3 targets tensorflow 1*, desc 2.1.1 targets tensorflow 2*.
- Default hidden layers/nodes and autoencoder hyperparameters are only in Supplementary Tables 2 and 3, not in the main text.

## Evidence

- [methods] iterative KL self-training with auxiliary target P: [evidence span withheld — anchor status: not_found]
- [discussion] Student's t kernel soft assignments: "we compute a soft assignment of each cell between the embedded points and the cluster centroids. Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel"
- [discussion] convergence tolerance: [evidence span withheld — anchor status: not_found]
- [abstract] batch-effect removal condition: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [supplementary] Gaussian kernel less robust than t kernel: "We also evaluated the performance of Gaussian distribution as a kernel, but found it to be less robust than the t distribution (Supplementary Note 3)."
- [discussion] Louvain centroid initialization on bottleneck features: "The cluster centers are initialized by the Louvain’s clustering algorithm6, which aims to optimize modularity for community detection."
