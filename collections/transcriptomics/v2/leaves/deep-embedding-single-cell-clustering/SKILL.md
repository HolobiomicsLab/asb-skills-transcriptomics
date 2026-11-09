---
name: deep-embedding-single-cell-clustering
description: Use when clustering scRNA-seq count data (e.g. UMI count matrices like
  GSE115746 macaque retina bipolar cells, 21,017 foveal + 9,285 peripheral cells)
  where batch effects may confound cell-type identification and batch labels may be
  unavailable.
when_to_use_negative:
- When explicit batch-corrected integration requiring batch labels is needed and technical
  differences across batches are larger than true biological variation — DESC's guarantee
  only holds 'as long as technical differences across batches are smaller than true
  biological variations'.
- When the input is already a feature/embedding table or non-scRNA-seq data; DESC
  expects a raw scRNA-seq gene expression count matrix that it maps to a low-dimensional
  feature space itself.
- When reproducing the paper's exact numbers is required and the environment runs
  TensorFlow 2.x — the README states TensorFlow must be lower than 2.0 (desc 2.0.3)
  to get the same results as the paper.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Unsupervised deep embedding algorithm trained via desc.train to cluster cells
    and remove batch effects iteratively; version 2.0.3 for TensorFlow 1.x
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion for highly variable
    gene selection, sc.pp.scale) and ARI-based evaluation of clustering
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC deep neural network model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:2d04ff063dfa2ed28d71d0bb7ea1b3751f1f01beffdf706a06c9b8cc59af3f25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# deep-embedding-single-cell-clustering

## Summary

Applies DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a clustering objective function while gradually removing batch effects, without explicitly requiring batch information. Accuracy is quantified with ARI against reference cell-type labels, e.g. ARI 0.920 on macaque retina bipolar cells.

## When to use

Use when clustering scRNA-seq count data (e.g. UMI count matrices like GSE115746 macaque retina bipolar cells, 21,017 foveal + 9,285 peripheral cells) where batch effects may confound cell-type identification and batch labels may be unavailable.

## When NOT to use

- When explicit batch-corrected integration requiring batch labels is needed and technical differences across batches are larger than true biological variation — DESC's guarantee only holds 'as long as technical differences across batches are smaller than true biological variations'.
- When the input is already a feature/embedding table or non-scRNA-seq data; DESC expects a raw scRNA-seq gene expression count matrix that it maps to a low-dimensional feature space itself.
- When reproducing the paper's exact numbers is required and the environment runs TensorFlow 2.x — the README states TensorFlow must be lower than 2.0 (desc 2.0.3) to get the same results as the paper.

## Inputs

- Single-cell scRNA-seq UMI count matrix (e.g. GEO accession GSE115746 macaque retina bipolar-cell counts, 21,017 foveal + 9,285 peripheral cells)
- Optional cell metadata containing batch definitions (macaque id, region id, sample id)
- Reference cell-type labels for ARI evaluation (e.g. bipolar-cell type labels)

## Outputs

- Cluster assignments and soft clustering probabilities per cell from desc.train
- Low-dimensional embedded feature representation (bottleneck layer, e.g. 32 dims)
- ARI values comparing DESC clusters to reference labels, written e.g. to ari_results.csv

## How to apply

Preprocess with Scanpy: cell-level normalization (UMI counts divided by total UMIs per cell, multiplied by 10,000, natural log via sc.pp.log1p), highly variable gene selection with sc.pp.filter_genes_dispersion (e.g. n_top_genes=1000), and per-gene standardization with sc.pp.scale(max_value=6). Train desc.train with a stacked autoencoder (dims=[n_genes,64,32]), tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], initializing cluster centers with Louvain clustering on the bottleneck-layer features. DESC then computes soft assignments of cells to centroids using a Student's t-distribution kernel and minimizes the KL divergence between soft assignments q_i and an auxiliary target distribution p_i, stopping when the proportion of cells changing cluster assignment is less than tol; batch effects are removed iteratively as long as technical differences across batches are smaller than true biological variation. Evaluate clustering accuracy by computing the adjusted Rand index (ARI) against reference cell-type labels with Scanpy/scikit-learn, optionally repeating training with different batch definitions (macaque id, region id, sample id, or none) to test robustness.

## Related tools

- **DESC** (Unsupervised deep embedding algorithm trained via desc.train to cluster cells and remove batch effects iteratively; version 2.0.3 for TensorFlow 1.x) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion for highly variable gene selection, sc.pp.scale) and ARI-based evaluation of clustering) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC deep neural network model)

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); desc.train(adata, dims=[adata.n_vars,64,32], tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], use_GPU=False, num_Cores=1, save_dir='results')
```

## Evaluation signals

- ARI against reference bipolar-cell type labels falls within 0.919–0.970 when any batch definition (macaque id, region id, sample id) is supplied, and is ~0.920 with no batch information, matching the paper's reported values
- Cells are well mixed across batches in the embedding (assessed by regional KL divergence of batch mixing over K nearest neighbors, K = 5 × number of batches, 200 iterations)
- Iterative training converges: the proportion of cells changing cluster assignment between two consecutive steps drops below tol = 0.005
- Preprocessing invariants: 1000 highly variable genes selected, gene values clipped at max_value=6 during sc.pp.scale
- A contrast check: unlike DESC, scVI's ARI on the same macaque retina data without batch information drops to 0.242, so DESC results should not degrade comparably

## Limitations

- Batch effect removal only succeeds 'as long as technical differences across batches are smaller than true biological variations'; with cell-type-specific batch effects (studies that deplete or enrich certain cell types), performance may suffer.
- To reproduce the paper's results, TensorFlow must be lower than 2.0 (desc 2.0.3); desc 2.1.1 targets TensorFlow 2.x, and Python 3.5.x or 3.6.x is required.
- DESC does not explicitly require batch information, but pairwise-based alternatives (CCA, MNN, Seurat 3.0, Scanorama) make correction order-dependent, so comparisons across methods may vary by batch ordering.
- No software version numbers are stated for Keras or Scanpy in the paper, and Supplementary Tables 2–3 (default hidden layers/nodes and autoencoder hyperparameters) are referenced but not provided in the text.

## Evidence

- [abstract] DESC abstract claim: "We present DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a clustering objective function"
- [discussion] DESC convergence criterion: [evidence span withheld — anchor status: not_found]
- [discussion] Louvain cluster-center initialization: "The cluster centers are initialized by the Louvain’s clustering algorithm6, which aims to optimize modularity for community detection."
- [methods] Scanpy preprocessing of highly variable genes: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [discussion] ARI as clustering accuracy metric: "we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering."
- [abstract] Batch-effect removal condition: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [readme] TensorFlow version constraint for reproducibility: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
