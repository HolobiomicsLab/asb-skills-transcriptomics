---
name: scrna-seq-normalization-hvg-selection
description: Use when a raw scRNA-seq count matrix (e.g., from Tabula Muris, 'Qx Limb
  Muscle', or PDAC datasets) must be normalized and reduced to highly variable genes
  prior to training a graph convolutional autoencoder such as scMGCA.
when_to_use_negative:
- Input is already a normalized, HVG-filtered expression matrix (e.g., a denoised
  output from DCA or scVI) — re-filtering and re-normalizing can discard signal
- The analysis goal is marker-gene annotation or differential expression with Seurat/SCANPY
  on pre-clustered data rather than building an autoencoder input
- Bulk RNA-seq or spatial transcriptomics data, where the single-cell dropout-reduction
  rationale for filtering all-zero genes does not apply in the same way
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Scalable toolkit used to normalize the scRNA-seq gene expression matrix and
    select highly variable genes during preprocessing
- name: scMGCA
  role: Downstream consumer of the normalized HVG matrix; its two-layer GCN autoencoder
    trains on the preprocessed data to produce latent embeddings and weight matrices
    W1/W2
  repo: https://pypi.org/project/scMGCA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:5833888c2281b9c6b16a9605000e3fc25ea2b52589ed07c0a4f735cb599eeca7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scrna-seq-normalization-hvg-selection

## Summary

Preprocess scRNA-seq count matrices by filtering out genes not expressed in cells, normalizing the gene expression matrix, and selecting highly variable genes before feeding them into the scMGCA graph-embedding autoencoder. This step reduces the impact of high dropout rates and retains only high-information genes for downstream clustering and genomic interpretation.

## When to use

Use when a raw scRNA-seq count matrix (e.g., from Tabula Muris, 'Qx Limb Muscle', or PDAC datasets) must be normalized and reduced to highly variable genes prior to training a graph convolutional autoencoder such as scMGCA.

## When NOT to use

- Input is already a normalized, HVG-filtered expression matrix (e.g., a denoised output from DCA or scVI) — re-filtering and re-normalizing can discard signal
- The analysis goal is marker-gene annotation or differential expression with Seurat/SCANPY on pre-clustered data rather than building an autoencoder input
- Bulk RNA-seq or spatial transcriptomics data, where the single-cell dropout-reduction rationale for filtering all-zero genes does not apply in the same way

## Inputs

- Raw scRNA-seq count matrix (genes × cells), e.g., the benchmark 'Qx Limb Muscle' dataset from public repositories
- AnnData/SCANPY object holding the gene expression matrix
- Number of highly variable genes d to retain

## Outputs

- Normalized gene expression matrix restricted to genes expressed in at least one cell
- Set of the top d highly variable genes used as input features for scMGCA training
- Preprocessed AnnData object suitable for graph construction (cell-PPMI matrix) and GCN encoding

## How to apply

First, filter out genes that are not expressed in any cells, which reduces the impact of the high dropout rates typical of scRNA-seq data. Then normalize the scRNA-seq gene expression matrix (in SCANPY, e.g., via normalize_total) and select the top d highly variable genes, where d is the number of genes retaining high-level information for the study. These preprocessed genes become the input features for scMGCA's two GCN layers, which further derive weight matrices (W1, W2) for later pivotal-gene extraction. The chosen HVG set directly conditions both the clustering quality (cell-PPMI graph built on the KNN graph of the normalized data) and the subsequent extraction of the top 200 pivotal genes using the standard deviation and weight of the latent embedded representation.

## Related tools

- **SCANPY** (Scalable toolkit used to normalize the scRNA-seq gene expression matrix and select highly variable genes during preprocessing)
- **scMGCA** (Downstream consumer of the normalized HVG matrix; its two-layer GCN autoencoder trains on the preprocessed data to produce latent embeddings and weight matrices W1/W2) — https://pypi.org/project/scMGCA

## Examples

```
import scanpy as sc; sc.pp.filter_genes(adata, min_cells=1); sc.pp.normalize_total(adata); sc.pp.highly_variable_genes(adata, n_top_genes=d)
```

## Evaluation signals

- All genes retained after filtering show nonzero expression in at least one cell (no all-zero rows in the count matrix)
- The normalized matrix contains a defined set of exactly d highly variable genes flagged by the SCANPY HVG call
- Downstream scMGCA training converges and produces a 128-dimensional latent embedding from the HVG input
- Clustering metrics (e.g., ARI/ASW) on the preprocessed data match or approach the paper's reported benchmark performance on 'Qx Limb Muscle', 'QS heart', 'Muraro', and 'Adam' datasets
- The subsequent pivotal-gene extraction yields a ranked list of 200 genes whose KEGG/GO enrichment (pvalueCutoff=0.01) reproduces significant terms consistent with the paper's findings

## Limitations

- The number of highly variable genes d is not given a numeric value in the article; practitioners must choose it and this choice conditions downstream pivotal-gene results
- Software version numbers for Python, TensorFlow, and SCANPY are unstated, so exact reproducibility of the preprocessing behavior may vary across versions
- No specific dataset accessions are provided; benchmark data are only described as coming from public repositories, complicating input verification
- The KNN neighbor count k and restart probability α used downstream for graph construction are not specified in the article

## Evidence

- [results] Normalized the scRNA-seq gene expression matrix and selected highly variable genes: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [discussion] Filter out genes not expressed in cells to reduce dropout impact: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [discussion] Top d highly variable genes carry high-level information: "the top d highly variable genes with high-level information are used in the study"
- [intro] SCANPY is the scalable toolkit for single-cell gene expression analysis: "SCANPY is a scalable toolkit for analyzing single-cell gene expression data built jointly with anndata"
- [discussion] scMGCA is publicly available on PyPI and built on TensorFlow: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
