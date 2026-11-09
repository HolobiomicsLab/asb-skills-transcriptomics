---
name: single-cell-rnaseq-batch-effect-correction
description: Use when clustering scRNA-seq data with strong batch effects across many
  batches (e.g., 30 samples), when batch information is unavailable, or when scaling
  to tens of thousands of cells and needing runtime/memory comparisons of correction
  methods.
when_to_use_negative:
- The analysis goal is only quality control or differential expression without clustering
  — batch effect removal via DESC is coupled to the clustering objective, not a standalone
  correction step
- The dataset fits pairwise-order assumptions and the user requires a reference-batch-free
  pairwise method workflow already established in their pipeline (Scanorama/Seurat
  3.0 correction may suffice at much lower runtime than DESC's competitors)
- Technical batch differences exceed true biological variation — DESC's iterative
  self-learning only removes batch effects 'as long as technical differences across
  batches are smaller than true biological variations'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Primary deep embedding algorithm that jointly clusters and removes batch effects
    via iterative self-learning on a Keras-trained autoencoder
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: 'Preprocessing: UMI normalization, log1p transform, and highly variable gene
    selection via filter_genes_dispersion'
  repo: https://github.com/theislab/scanpy
- name: Seurat 3.0
  role: MNN+CCA batch correction comparison method for the runtime benchmark
- name: scVI
  role: Comparison method that explicitly incorporates batch information
- name: BERMUDA
  role: Comparison method that explicitly incorporates batch information; monitored
    for exceeding 32 GB peak memory
- name: Scanorama
  role: Pairwise batch correction comparison method that does not require a reference
    batch
- name: Keras
  role: Backend used to train the DESC autoencoder model with GPU support
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:a9f59bd503f6dac1e67f1d718a3680aded8647f95f101c0ae9dc5802eb3884e3
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
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-rnaseq-batch-effect-correction

## Summary

Benchmark and apply deep-learning-based batch effect correction and clustering for scRNA-seq using DESC, comparing runtimes, memory, and clustering accuracy (ARI) against MNN (Seurat 3.0), scVI, BERMUDA, and Scanorama on multi-batch datasets such as the macaque retina data.

## When to use

Use when clustering scRNA-seq data with strong batch effects across many batches (e.g., 30 samples), when batch information is unavailable, or when scaling to tens of thousands of cells and needing runtime/memory comparisons of correction methods.

## When NOT to use

- The analysis goal is only quality control or differential expression without clustering — batch effect removal via DESC is coupled to the clustering objective, not a standalone correction step
- The dataset fits pairwise-order assumptions and the user requires a reference-batch-free pairwise method workflow already established in their pipeline (Scanorama/Seurat 3.0 correction may suffice at much lower runtime than DESC's competitors)
- Technical batch differences exceed true biological variation — DESC's iterative self-learning only removes batch effects 'as long as technical differences across batches are smaller than true biological variations'

## Inputs

- Raw scRNA-seq UMI count matrix (cells x genes) from a multi-batch/multi-sample dataset (e.g., macaque retina scRNA-seq, 30,000 cells)
- Batch labels (e.g., sample id) for methods that explicitly require batch information (scVI, BERMUDA)
- Reference cell-type labels for ARI-based clustering accuracy evaluation (optional)

## Outputs

- Soft cluster assignments with per-cell cluster probability scores from DESC
- Low-dimensional feature embedding (e.g., 32-dim bottleneck layer)
- Per-method runtime and peak-memory benchmark table (e.g., runtime_benchmark_macaque.csv)
- ARI values comparing clustering accuracy across methods

## How to apply

Load the multi-batch scRNA-seq count matrix (e.g., macaque retina, 30 batches defined by sample id, 30,000 cells subsampled) and preprocess with Scanpy: divide UMI counts per cell by total UMIs, multiply by 10,000, natural-log transform (sc.pp.log1p), and select highly variable genes with sc.pp.filter_genes_dispersion (n_top_genes=1000), then scale with sc.pp.scale(max_value=6). Run DESC with a stacked autoencoder pretraining step (dims=[n_genes,64,32]), Louvain-initialized cluster centers, a Student's t distribution kernel for soft assignment, and iterative self-training that stops when the proportion of cells changing cluster assignment between consecutive steps is below tol=0.005, using GPU via Keras when available. Time MNN (Seurat 3.0), scVI, BERMUDA, and Scanorama end-to-end on the identical input and record peak memory. Rationale: DESC jointly removes batch effects during clustering and gradually removes technical differences as long as they are smaller than true biological variation, unlike pairwise methods whose results depend on batch order.

## Related tools

- **DESC** (Primary deep embedding algorithm that jointly clusters and removes batch effects via iterative self-learning on a Keras-trained autoencoder) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing: UMI normalization, log1p transform, and highly variable gene selection via filter_genes_dispersion) — https://github.com/theislab/scanpy
- **Seurat 3.0** (MNN+CCA batch correction comparison method for the runtime benchmark)
- **scVI** (Comparison method that explicitly incorporates batch information)
- **BERMUDA** (Comparison method that explicitly incorporates batch information; monitored for exceeding 32 GB peak memory)
- **Scanorama** (Pairwise batch correction comparison method that does not require a reference batch)
- **Keras** (Backend used to train the DESC autoencoder model with GPU support)

## Examples

```
import scanpy as sc; adata=sc.datasets.paul15(); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)
```

## Evaluation signals

- Measured runtimes for 30,000 cells across 30 batches fall within a reasonable tolerance of reported values: DESC ~0.3 h, MNN ~9.5 h, scVI ~10.4 h, BERMUDA ~11.2 h, Scanorama ~0.4 h
- BERMUDA's recorded peak memory exceeds 32 GB; DESC shows a small memory footprint
- Clustering accuracy measured by ARI against reference cell-type labels is competitive or better than pairwise methods (MNN, Seurat 3.0, Scanorama)
- DESC convergence reached: the proportion of cells changing cluster assignment between two consecutive iterative steps dropped below tol=0.005
- Batch mixing improved, assessable via regional KL divergence of batch mixing with K nearest neighbors (K = 5 x number of batches) over 200 random-cell iterations

## Limitations

- DESC gradually removes batch effects only as long as technical differences across batches are smaller than true biological variations
- Pairwise-based methods (CCA, MNN, Seurat 3.0, Scanorama) are order-dependent: 'the order in which batches are corrected will affect the final results'
- BERMUDA can exceed 32 GB of memory on large 30-batch datasets, limiting scalability
- To reproduce the paper's results exactly, desc(2.0.3) with tensorflow < 2.0 must be used due to tensorflow 1* vs 2* differences; the package requires python 3.5.x or 3.6.x
- No software version numbers stated for Keras or Scanpy; Gaussian kernel was evaluated but found less robust than the t distribution

## Evidence

- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [abstract] DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information for batch effect removal: "DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information for batch effect removal"
- [discussion] This iterative procedure stops when the proportion of cells that changes cluster assignment between two consecutive steps is less than tol.: [evidence span withheld — anchor status: not_found]
- [methods] Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [discussion] the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale.: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [readme] Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper.: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
