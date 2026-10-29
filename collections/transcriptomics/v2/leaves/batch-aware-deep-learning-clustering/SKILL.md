---
name: batch-aware-deep-learning-clustering
description: Use when clustering single-cell RNA-seq data spanning multiple batches
  where batch effect must be removed jointly with clustering, and no reliable batch
  annotation is available (DESC does not explicitly require batch information).
when_to_use_negative:
- When batch information is explicit and a batch-supervised method like scVI or BERMUDA
  is required by the design
- When technical differences across batches are larger than true biological variations
  — DESC removes batch effects only as long as technical differences are smaller than
  biological variation
- When reproduction of published results requires tensorflow >= 2.0 with the desc
  2.0.3 release — the README states tensorflow must be lower than 2.0 for paper-equivalent
  results
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Deep embedding algorithm performing the iterative self-learning clustering
    and batch effect removal
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: 'Preprocessing: cell-level UMI normalization, log1p transform, filter_genes_dispersion
    HVG selection, sc.pp.scale'
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC autoencoder model on GPU
- name: desc 2.0.3
  role: Package release compatible with tensorflow 1* for reproducing paper results
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:ce5ebaa102b55bbfdcbb5c928f54c93762fe4e970f21e8afc1e11e665a1948b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - BERMUDA
  - DESC
  - Keras
  - Scanorama
  - Scanpy
  - Seurat 3.0
  - desc 2.0.3
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# batch-aware-deep-learning-clustering

## Summary

Apply DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a KL-divergence clustering objective, so that batch effects are gradually removed during clustering without explicitly supplying batch labels.

## When to use

Use when clustering single-cell RNA-seq data spanning multiple batches where batch effect must be removed jointly with clustering, and no reliable batch annotation is available (DESC does not explicitly require batch information).

## When NOT to use

- When batch information is explicit and a batch-supervised method like scVI or BERMUDA is required by the design
- When technical differences across batches are larger than true biological variations — DESC removes batch effects only as long as technical differences are smaller than biological variation
- When reproduction of published results requires tensorflow >= 2.0 with the desc 2.0.3 release — the README states tensorflow must be lower than 2.0 for paper-equivalent results

## Inputs

- scRNA-seq UMI count matrix (cells x genes), e.g. macaque retina data subsampled to 30,000 cells across 30 sample-id batches
- Optional batch/sample id definitions (not required by DESC itself)
- Number of clusters for Louvain-based cluster-center initialization

## Outputs

- Soft cluster assignment probabilities per cell
- Low-dimensional embedded feature representation (bottleneck layer)
- Cluster labels after convergence (tol=0.005)
- Benchmark artifacts such as per-method runtime and peak-memory tables (runtime_benchmark_macaque.csv)

## How to apply

Preprocess the UMI count matrix with Scanpy: divide each gene's counts by total UMIs per cell, multiply by 10,000, natural-log transform (sc.pp.log1p), then select highly variable genes with sc.pp.filter_genes_dispersion (e.g. n_top_genes=1000) and scale with sc.pp.scale (max_value=6), standardizing each gene by its mean and standard deviation across cells. Initialize the stacked autoencoder (e.g. dims=[n_genes,64,32]) to learn a low-dimensional representation, then initialize cluster centers by Louvain clustering in the bottleneck-layer feature space. Iteratively compute soft assignments using a Student's t distribution kernel and refine clusters by minimizing KL divergence between soft assignments q_i and an auxiliary target distribution p_i, updating P, cluster centers, and encoder parameters until the proportion of cells changing cluster assignment between consecutive steps falls below tol=0.005. DESC can use the GPU when available (implemented in Keras/tensorflow, desc 2.0.3 for tensorflow 1*, desc 2.1.1 for tensorflow 2*), and was benchmarked far faster than MNN, scVI, and BERMUDA (about 0.3 h for 30,000 cells across 30 macaque retina batches vs 9.5–11.2 h for the others).

## Related tools

- **DESC** (Deep embedding algorithm performing the iterative self-learning clustering and batch effect removal) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing: cell-level UMI normalization, log1p transform, filter_genes_dispersion HVG selection, sc.pp.scale) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC autoencoder model on GPU)
- **desc 2.0.3** (Package release compatible with tensorflow 1* for reproducing paper results) — https://github.com/eleozzr/desc

## Examples

```
import scanpy as sc; adata=sc.datasets.paul15(); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)
```

## Evaluation signals

- Iterative training stops only when the proportion of cells changing cluster assignment between two consecutive steps is below tol=0.005
- Clustering accuracy validated against reference cell-type labels using ARI, with larger ARI indicating higher accuracy
- Batch effect removal quantified by regional KL divergence of batch mixing with K = 5 x number of batches nearest neighbors over 200 iterations
- Benchmark reproduction should yield runtimes within tolerance of reported values (DESC ~0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, scanorama 0.4 h for 30,000 cells / 30 batches)
- DESC's recorded peak memory should stay small (small memory footprint; BERMUDA by contrast exceeds 32 GB)

## Limitations

- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations
- Reproducing the paper's results requires tensorflow < 2.0 (desc 2.0.3); desc 2.1.1 targets tensorflow 2* and the desc package requires Python 3.5.x or 3.6.x
- Pairwise batch-correction alternatives (CCA, MNN, Seurat 3.0, Scanorama) are order-dependent, a limitation DESC avoids but that affects comparison benchmarks
- Cluster centers are initialized by Louvain clustering, so initialization quality can affect the iterative refinement

## Evidence

- [abstract] DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information: "DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information"
- [methods] Preprocessing chain quoted from methods: "sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)"
- [discussion] Iteration stopping criterion: [evidence span withheld — anchor status: not_found]
- [readme] README version constraint: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
- [results] Benchmark runtime finding: [evidence span withheld — anchor status: not_found]
