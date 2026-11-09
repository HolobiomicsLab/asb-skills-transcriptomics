---
name: single-cell-rna-seq-preprocessing-normalization
description: Use when raw UMI count matrices from scRNA-seq (e.g. Fluidigm C1, SMART-seq2,
  CEL-seq/2 or PBMC datasets) must be normalized, HVG-filtered, and standardized before
  clustering with DESC or similar deep-embedding methods.
when_to_use_negative:
- Input is already normalized/standardized expression data (e.g. a saved adata_desc.h5ad
  from a prior DESC run) — re-scaling would distort values
- Data types not based on UMI counts where library-size normalization ×10,000 and
  log1p are inappropriate
- Downstream methods that expect raw counts (e.g. probabilistic models like scVI that
  model count distributions directly) rather than log-normalized scaled values
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Performs cell-level normalization (log1p), highly variable gene selection
    (filter_genes_dispersion), and gene standardization (sc.pp.scale) prior to DESC
    clustering
  repo: https://github.com/theislab/scanpy
- name: DESC
  role: 'Downstream consumer of the preprocessed matrix: stacked autoencoder pretraining,
    Louvain-initialized clustering, and iterative self-learning with KL divergence'
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:5ec1a3179560d6e723ab38989ffcc4807f4c2eb2bdfd9af0bd851945e7197c50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scanpy
  tools:
  - DESC
  - Keras
  - Scanpy
  - TensorFlow
  - desc
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-rna-seq-preprocessing-normalization

## Summary

Preprocess raw single-cell RNA-seq UMI count matrices for DESC clustering via Scanpy: cell-level normalization (counts/total UMIs ×10,000, log1p), highly variable gene selection with filter_genes_dispersion, and gene-level standardization (sc.pp.scale, max_value=6) so deep embedding clustering receives a comparable, low-noise expression matrix.

## When to use

Use when raw UMI count matrices from scRNA-seq (e.g. Fluidigm C1, SMART-seq2, CEL-seq/2 or PBMC datasets) must be normalized, HVG-filtered, and standardized before clustering with DESC or similar deep-embedding methods.

## When NOT to use

- Input is already normalized/standardized expression data (e.g. a saved adata_desc.h5ad from a prior DESC run) — re-scaling would distort values
- Data types not based on UMI counts where library-size normalization ×10,000 and log1p are inappropriate
- Downstream methods that expect raw counts (e.g. probabilistic models like scVI that model count distributions directly) rather than log-normalized scaled values

## Inputs

- Raw UMI count matrix (cell × gene) from scRNA-seq protocols such as Fluidigm C1, SMART-seq2, CEL-seq, CEL-seq2
- AnnData (.h5ad) or similar count-matrix container (e.g. Kang et al. PBMC matrix from GEO accession GSE96583)

## Outputs

- Normalized, HVG-filtered, gene-standardized AnnData ready for desc.train
- Vector of highly variable genes selected by filter_genes_dispersion (e.g. top 1000 genes)

## How to apply

Load count matrices into a Scanpy AnnData object, then apply cell-level normalization by dividing each gene's UMI count by the cell's total UMIs, multiplying by 10,000, and natural-log transforming with sc.pp.log1p(adata). Select highly variable genes with sc.pp.filter_genes_dispersion(adata, n_top_genes=1000), which reduces dimensionality before autoencoder training. Standardize each gene by subtracting its mean across all cells and dividing by its standard deviation, capped with sc.pp.scale(adata, max_value=6) to limit outlier influence. This ordering (cell normalization → HVG filtering → gene scaling) is the article's stated pipeline for preparing input to DESC's stacked autoencoder, which pretrains on the standardized matrix to learn a low-dimensional representation before Louvain-based cluster center initialization and KL-divergence self-learning.

## Related tools

- **Scanpy** (Performs cell-level normalization (log1p), highly variable gene selection (filter_genes_dispersion), and gene standardization (sc.pp.scale) prior to DESC clustering) — https://github.com/theislab/scanpy
- **DESC** (Downstream consumer of the preprocessed matrix: stacked autoencoder pretraining, Louvain-initialized clustering, and iterative self-learning with KL divergence) — https://github.com/eleozzr/desc

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- After normalization, each cell's values correspond to UMI counts divided by total cell UMIs, multiplied by 10,000 and log1p-transformed
- Number of retained genes matches n_top_genes (e.g. 1000 highly variable genes) after sc.pp.filter_genes_dispersion
- Scaled gene values are bounded within ±6 (sc.pp.scale max_value=6) and have ~zero mean and unit variance per gene
- Downstream DESC training converges (proportion of cells changing cluster between steps falls below tol = 0.005)
- Downstream clustering reproduces expected metrics, e.g. ARI 0.945 and alpha/beta accuracies of 96.5%/98.3% on the four-protocol islet data

## Limitations

- README notes reproduction of the paper's results requires tensorflow < 2.0 (desc 2.0.3) and python 3.5.x/3.6.x
- No software version numbers are stated for Keras or Scanpy in the article text
- Batch-effect removal via DESC only succeeds as long as technical differences across batches are smaller than true biological variations; preprocessing alone does not correct batch effect
- Supplementary details (default autoencoder hyperparameters, Supplementary Table 3) are referenced but not included in the provided text

## Evidence

- [discussion] the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale.: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [discussion] Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [discussion] the cell level normalized values for each gene are standardized by subtracting the mean across all cells and divided by the standard deviation across all cells for the given gene.: "the cell level normalized values for each gene are standardized by subtracting the mean across all cells and divided by the standard deviation across all cells for the given gene."
- [methods] sc.pp.filter_genes_dispersion(adata,n_top_genes=1000): "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [discussion] a stacked autoencoder is used for pretraining and learning low-dimensional representation of the input gene expression matrix.: "a stacked autoencoder is used for pretraining and learning low-dimensional representation of the input gene expression matrix."
- [readme] Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper.: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
