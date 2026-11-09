---
name: autoencoder-pretraining
description: Use when starting a DESC clustering workflow on preprocessed scRNA-seq
  data (log1p-transformed, top-1000 HVGs, scaled) and a low-dimensional embedding
  is needed to initialize cluster centers.
when_to_use_negative:
- Input is raw UMI counts that have not undergone cell-level normalization (divide
  by total UMIs, multiply by 10,000, natural log transform) and gene-level standardization
- The analysis explicitly requires batch information to be incorporated, as in scVI
  or BERMUDA, rather than implicit batch-effect removal via iterative self-learning
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: desc
  role: Provides the DESC implementation whose parameter initialization step is the
    stacked autoencoder pretraining; versions 2.0.3 (tensorflow 1*) and 2.1.1 (tensorflow
    2*)
  repo: https://github.com/eleozzr/desc
- name: SCANPY
  role: 'Preprocessing before pretraining: sc.pp.log1p, sc.pp.filter_genes_dispersion
    for HVG selection, sc.pp.scale with max_value=6'
  repo: https://github.com/theislab/scanpy
- name: TensorFlow
  role: Backend used to train the stacked autoencoder neural network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:dd7a43ed437953a62561badc8e86c55d2dff4e469d23177ccb94c62ccc4ac410
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# autoencoder-pretraining

## Summary

Pretrain a stacked autoencoder to obtain a low-dimensional representation of the gene expression matrix, serving as parameter initialization for DESC's iterative KL-divergence self-training clustering of scRNA-seq data.

## When to use

Use when starting a DESC clustering workflow on preprocessed scRNA-seq data (log1p-transformed, top-1000 HVGs, scaled) and a low-dimensional embedding is needed to initialize cluster centers.

## When NOT to use

- Input is raw UMI counts that have not undergone cell-level normalization (divide by total UMIs, multiply by 10,000, natural log transform) and gene-level standardization
- The analysis explicitly requires batch information to be incorporated, as in scVI or BERMUDA, rather than implicit batch-effect removal via iterative self-learning

## Inputs

- AnnData object of gene expression counts (e.g. sc.datasets.paul15())
- Preprocessed matrix: log1p-transformed, top 1000 highly variable genes (filter_genes_dispersion), scaled with max_value=6

## Outputs

- Low-dimensional bottleneck embedding (X_Embeded_z) stored in adata.obsm
- Soft-clustering assignment probabilities stored in adata.obs
- Trained autoencoder weights used as initialization for iterative KL-divergence clustering
- t-SNE coordinates (learning_rate=200) saved to paul_result/adata_desc.h5ad

## How to apply

After Scanpy preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale with max_value=6), pretrain a stacked autoencoder with encoder dims [n_genes, 64, 32] on the standardized expression matrix to learn the low-dimensional bottleneck representation. This pretraining provides the parameter initialization from which the DESC procedure starts. Next, run Louvain modularity clustering (n_neighbors=10, resolutions 0.8 and 1.0) on the bottleneck-layer features to initialize cluster centers, then compute Student's t-kernel soft assignments Q and iteratively minimize KL(P||Q) with batch_size=256 until the proportion of cells changing cluster assignment between consecutive iterations drops below tol=0.005. The pretrained encoder weights constrain the self-training phase so DESC learns cluster-specific information while ignoring irrelevant signals such as batch effect.

## Related tools

- **desc** (Provides the DESC implementation whose parameter initialization step is the stacked autoencoder pretraining; versions 2.0.3 (tensorflow 1*) and 2.1.1 (tensorflow 2*)) — https://github.com/eleozzr/desc
- **SCANPY** (Preprocessing before pretraining: sc.pp.log1p, sc.pp.filter_genes_dispersion for HVG selection, sc.pp.scale with max_value=6) — https://github.com/theislab/scanpy
- **TensorFlow** (Backend used to train the stacked autoencoder neural network)

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); desc.train(adata, dims=[adata.n_vars, 64, 32], tol=0.005, batch_size=256, resolution=[0.8, 1.0], n_neighbors=10, tsne_learning_rate=200, save_encoder_weights=True)
```

## Evaluation signals

- Louvain clustering on the bottleneck features (n_neighbors=10, resolutions 0.8 and 1.0) yields stable initial cluster centers
- Iterative self-training converges: the proportion of cells changing cluster assignment between consecutive steps falls below tol=0.005
- Clustering accuracy measured by ARI against reference cell-type labels (e.g. paul15_clusters) is comparable to the paper's reported performance
- Output AnnData contains soft-clustering probabilities in obs, embedding in X_Embeded_z, and t-SNE coordinates, and saves successfully to paul_result/adata_desc.h5ad
- Batch-effect removal assessed by regional KL divergence of batch mixing (K = 5 × number of batches, 200 iterations) shows mixing improvement

## Limitations

- To reproduce the paper's results, tensorflow must be lower than 2.0 (use desc 2.0.3); desc requires python 3.5.x or 3.6.x
- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations
- Gaussian kernel was found less robust than the Student's t distribution kernel (Supplementary Note 3)
- No software version numbers are stated for Keras or Scanpy in the provided text

## Evidence

- [discussion] The DESC procedure starts with parameter initialization, in which a stacked autoencoder is used for pretraining and learning low-dimensional representation of the input gene expression matrix.: "The DESC procedure starts with parameter initialization, in which a stacked autoencoder is used for pretraining and learning low-dimensional representation of the input gene expression matrix."
- [discussion] This iterative procedure stops when the proportion of cells that changes cluster assignment between two consecutive steps is less than tol.: [evidence span withheld — anchor status: not_found]
- [readme] Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper.: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper"
- [discussion] Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
