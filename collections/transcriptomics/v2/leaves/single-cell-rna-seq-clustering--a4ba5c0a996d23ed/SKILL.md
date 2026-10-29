---
name: single-cell-rna-seq-clustering
description: Use when scRNA-seq count matrices need unsupervised clustering into cell
  populations, especially when batch effects are present but batch labels are unavailable
  or should not be explicitly supplied.
when_to_use_negative:
- Batch differences are larger than true biological variation — DESC only gradually
  removes batch effects 'as long as technical differences across batches are smaller
  than true biological variations'
- The analysis requires an explicitly batch-aware variational model that incorporates
  batch labels (e.g., scVI or BERMUDA were designed for that role)
- Clustering must be order-independent across pairwise batch corrections on many batches
  — pairwise methods like MNN/CCA or Scanorama may be preferred when control over
  correction order matters differently
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: desc
  role: 'Core implementation of deep embedding for single-cell clustering: autoencoder
    pretraining, Louvain center initialization, and iterative KL-divergence self-training
    (desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*)'
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion for highly variable
    gene selection, sc.pp.scale) and ARI-based clustering accuracy evaluation
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC autoencoder model
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:a5498837a5fc6ab64500064e4385c93307ce141c9f7164fe16636a67f204c3ef
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
yes_rate: 0.46153846153846156
n_claims: 13
---

# single-cell-rna-seq-clustering

## Summary

Cluster scRNA-seq data with DESC, an unsupervised deep embedding algorithm that pretrains a stacked autoencoder, initializes cluster centers with Louvain clustering, and iteratively minimizes KL divergence between soft assignments Q and an auxiliary target distribution P to learn cluster-specific representations while reducing batch effect.

## When to use

Use when scRNA-seq count matrices need unsupervised clustering into cell populations, especially when batch effects are present but batch labels are unavailable or should not be explicitly supplied.

## When NOT to use

- Batch differences are larger than true biological variation — DESC only gradually removes batch effects 'as long as technical differences across batches are smaller than true biological variations'
- The analysis requires an explicitly batch-aware variational model that incorporates batch labels (e.g., scVI or BERMUDA were designed for that role)
- Clustering must be order-independent across pairwise batch corrections on many batches — pairwise methods like MNN/CCA or Scanorama may be preferred when control over correction order matters differently

## Inputs

- scRNA-seq count matrix (e.g., GSE115746 macaque retina bipolar-cell counts; 21,017 foveal + 9,285 peripheral cells)
- Cell metadata with batch covariates (macaque id, region id, sample id) — optional
- Reference cell-type labels for ARI evaluation (e.g., paul15_clusters annotation)
- AnnData object from scanpy (e.g., sc.datasets.paul15())

## Outputs

- Soft-clustering probabilities per cell (cluster assignment probabilities)
- Low-dimensional embedded representation (X_Embeded_z)
- Cluster assignments and t-SNE coordinates (learning_rate=200) stored in AnnData obs/uns/obsm slots
- Saved AnnData file (e.g., paul_result/adata_desc.h5ad)
- ARI results table (e.g., ari_results.csv)

## How to apply

Preprocess counts with Scanpy: cell-level normalization (divide by total UMIs, multiply by 10,000, log1p), select the top 1000 highly variable genes with sc.pp.filter_genes_dispersion, and standardize genes with sc.pp.scale(max_value=6). Pretrain a stacked autoencoder (encoder dims [n_genes, 64, 32]) to obtain a low-dimensional embedding, then initialize cluster centers by Louvain clustering on bottleneck-layer features with n_neighbors=10 at louvain_resolution=[0.8, 1.0]. Compute Student's t-kernel soft assignments Q to centroids, build the auxiliary target distribution P that emphasizes easy-to-cluster cells, and iteratively minimize KL(P||Q) with batch_size=256 until the fraction of cells changing cluster assignment between consecutive iterations falls below tol=0.005. Evaluate clustering accuracy with ARI against reference cell-type labels and assess batch mixing via regional KL divergence on K-nearest neighbors.

## Related tools

- **desc** (Core implementation of deep embedding for single-cell clustering: autoencoder pretraining, Louvain center initialization, and iterative KL-divergence self-training (desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*)) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion for highly variable gene selection, sc.pp.scale) and ARI-based clustering accuracy evaluation) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC autoencoder model)

## Examples

```
adata = sc.datasets.paul15(); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); desc.train(adata, dims=[adata.shape[1],64,32], tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], save_dir='paul_result')
```

## Evaluation signals

- ARI against reference cell-type labels within reported range (e.g., ~0.920 without batch information on macaque retina data; 0.919–0.970 with any batch definition)
- Training convergence: proportion of cells changing cluster assignment between two consecutive iterations drops below tol=0.005
- Batch mixing assessed via regional KL divergence on K nearest neighbors (K = 5 × number of batches, 200 iterations) shows cells well mixed across batches
- Resulting AnnData contains X_Embeded_z embedding, soft-clustering probabilities, and t-SNE coordinates, saved as a valid .h5ad file
- Reproducibility check: using tensorflow < 2.0 with desc 2.0.3 reproduces paper results on Paul et al. (2015) myeloid progenitor data

## Limitations

- Batch effects are removed only when technical differences across batches are smaller than true biological variations
- To reproduce the paper's results exactly, tensorflow must be lower than 2.0 (desc 2.0.3); tensorflow 2* requires desc 2.1.1
- Python 3.5.x or 3.6.x is required for installing the desc package
- Gaussian kernel was evaluated but found less robust than the Student's t distribution (Supplementary Note 3)
- No batch information needs to be explicitly provided, but performance depends on iterative self-learning quality of the initial Louvain clustering

## Evidence

- [discussion] Iterative clustering via KL divergence self-training: [evidence span withheld — anchor status: not_found]
- [discussion] Convergence criterion: [evidence span withheld — anchor status: not_found]
- [discussion] Soft assignment kernel: "we compute a soft assignment of each cell between the embedded points and the cluster centroids. Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel"
- [methods] Preprocessing filter step: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [abstract] Batch effect removal without batch labels: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [readme] Tensorflow version compatibility: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
- [results] Validation result on macaque retina data: [evidence span withheld — anchor status: not_found]
- [discussion] Autoencoder pretraining: "The DESC procedure starts with parameter initialization, in which a stacked autoencoder is used for pretraining and learning low-dimensional representation of the input gene expression matrix."
