---
name: deep-embedding-soft-clustering
description: Use when clustering scRNA-seq count data (e.g. PBMCs, thousands of cells)
  where batch effects confound biological variation and batch labels are unavailable
  or should not be used, and soft cluster probabilities are needed.
when_to_use_negative:
- Input is already a dimensionally reduced embedding or feature table rather than
  a raw UMI count matrix requiring cell-level normalization (CP10K + log1p) and gene
  standardization.
- Technical batch differences are larger than true biological variations — DESC only
  removes batch effect when technical differences across batches are smaller than
  true biological variations.
- Pairwise batch correction with known batch labels and reference batches (e.g. Seurat
  3.0 CCA/MNN or Scanorama workflows) is specifically required.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Unsupervised deep embedding algorithm performing iterative soft clustering
    and batch effect removal via KL-divergence self-training; invoked through the
    desc package's train function.
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: 'Preprocessing of count data: cell-level normalization, sc.pp.log1p, highly
    variable gene selection with sc.pp.filter_genes_dispersion (n_top_genes=1000),
    and sc.pp.scale(max_value=6) before DESC training.'
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the stacked autoencoder and deep neural network model
    of DESC.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:4ea9fd00a7fd8497db1373d502de63d60ddd5eca74d98c0ec7162bd4dfd634e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - desc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# deep-embedding-soft-clustering

## Summary

Apply DESC, an unsupervised deep embedding algorithm, to cluster single-cell RNA-seq data and remove batch effects without explicit batch labels by iteratively optimizing a KL-divergence clustering objective over an autoencoder-derived low-dimensional embedding. The soft cluster assignment probabilities (max.prob) also flag high-confidence cells.

## When to use

Use when clustering scRNA-seq count data (e.g. PBMCs, thousands of cells) where batch effects confound biological variation and batch labels are unavailable or should not be used, and soft cluster probabilities are needed.

## When NOT to use

- Input is already a dimensionally reduced embedding or feature table rather than a raw UMI count matrix requiring cell-level normalization (CP10K + log1p) and gene standardization.
- Technical batch differences are larger than true biological variations — DESC only removes batch effect when technical differences across batches are smaller than true biological variations.
- Pairwise batch correction with known batch labels and reference batches (e.g. Seurat 3.0 CCA/MNN or Scanorama workflows) is specifically required.

## Inputs

- UMI count matrix (e.g. Kang et al. PBMC counts from GEO GSE96583; 24,679 cells, control vs interferon-beta stimulated) as a Scanpy AnnData object or count matrix

## Outputs

- Cluster assignments per cell with soft assignment probabilities (max.prob) from the DESC probability matrix
- Annotated AnnData (e.g. adata_desc.h5ad) with DESC cluster labels and bottleneck embedding
- Cell-type annotations obtained by mapping DESC clusters to canonical PBMC types via marker genes
- Per-cell-type differential expression summary (e.g. kang_desc_de_results.csv) quantifying condition effects per cluster

## How to apply

Preprocess the UMI count matrix in Scanpy: normalize each cell (divide by total UMIs, multiply by 10,000, sc.pp.log1p), select highly variable genes with sc.pp.filter_genes_dispersion (e.g. n_top_genes=1000), and standardize with sc.pp.scale(max_value=6). Train DESC via desc.train without providing batch labels: initialize with a stacked autoencoder (bottleneck such as dims=[n_genes, 64, 32]), initialize cluster centers with Louvain clustering over candidate resolutions (e.g. [0.8, 1.0]), compute soft assignments with a Student's t distribution kernel, and minimize KL divergence between soft assignments q_i and the auxiliary target distribution p_i, stopping when the proportion of cells changing cluster is less than tol=0.005. The iterative self-training gradually removes batch effects as long as technical differences across batches are smaller than true biological variations. Downstream, compute per-cell maximum cluster probability from the soft-assignment matrix to identify high-confidence cells and map clusters to known cell types using marker genes.

## Related tools

- **DESC** (Unsupervised deep embedding algorithm performing iterative soft clustering and batch effect removal via KL-divergence self-training; invoked through the desc package's train function.) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing of count data: cell-level normalization, sc.pp.log1p, highly variable gene selection with sc.pp.filter_genes_dispersion (n_top_genes=1000), and sc.pp.scale(max_value=6) before DESC training.) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the stacked autoencoder and deep neural network model of DESC.)

## Examples

```
adata_res = desc.train(adata, dims=[adata.n_vars, 64, 32], tol=0.005, louvain_resolution=[0.8, 1.0], save_dir='.', save_encoder=True)
```

## Evaluation signals

- DESC soft-assignment max.prob values are high for major PBMC populations, indicating high-confidence cluster assignments.
- Convergence is reached: the proportion of cells changing cluster assignment between two consecutive iterations falls below tol=0.005.
- On the Kang et al. lupus PBMC data, cells from control and interferon-beta conditions are randomly mixed between DESC clusters for all cell types except CD14+ monocytes, indicating batch effect removal.
- Within-cell-type DE between control and stimulated cells shows DE gene counts and magnitudes for CD14+ monocytes several orders higher than other cell types, matching the expected INF-β biological signal.
- Clustering accuracy can be benchmarked via ARI against reference cell-type labels where such labels exist.

## Limitations

- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations.
- Reproducing the paper's results requires tensorflow < 2.0 with desc 2.0.3; desc 2.1.1 is needed for tensorflow 2*, and the desc package requires python 3.5.x or 3.6.x.
- The Gaussian kernel was evaluated and found less robust than the Student's t distribution kernel (Supplementary Note 3).
- No batch labels are provided explicitly; DESC is not a supervised batch-balancing method like scVI or BERMUDA, which explicitly incorporate batch information.

## Evidence

- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [discussion] Student's t soft assignment and KL self-training: [evidence span withheld — anchor status: not_found]
- [discussion] Convergence tolerance: [evidence span withheld — anchor status: not_found]
- [methods] Scanpy HVG selection: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [readme] desc versions tied to tensorflow release: "For  `tensorflow 1*`, we released `desc(2.0.3)`"
- [readme] DESC soft clustering interpretable probabilities: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence"
