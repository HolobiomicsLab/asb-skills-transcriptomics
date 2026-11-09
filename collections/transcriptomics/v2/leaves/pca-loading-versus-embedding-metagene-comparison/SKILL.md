---
name: pca-loading-versus-embedding-metagene-comparison
description: Use when a gene-embedding workflow has identified a candidate metagene
  (e.g. an IFIT1/ISG15/OAS1 cluster in interferon beta-stimulated PBMCs) and you need
  to demonstrate whether the program is cell-type-independent or an artifact of linear-model
  loading structure.
when_to_use_negative:
- Input data has already been normalized/log-transformed and scaled into a feature
  table — GeneVector requires raw counts in .X for its mutual-information discretization
  (bin 0 = zero expression, nonzero split into 10 quantile-based bins)
- The goal is batch correction or cell type annotation rather than evaluating whether
  a transcriptional program is confounded by cell type in linear loadings
- The phenotype program of interest involves gene repression or mutually exclusive
  expression — GeneVector's signed MI cannot distinguish activation from repression,
  so the contrast would be uninterpretable for such programs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Trains the gene embedding (GeneVectorDataset, GeneVector model, emb_dimension=100)
    and derives metagenes via GeneEmbedding.get_adata(resolution=40) and get_metagenes
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Hosts the raw-count AnnData input and variable-gene selection (seurat_v3 flavor)
    / entropy-based quality control before embedding; used for per-cell-type scaled
    expression and visualization
- name: LDVAE
  role: Fits the 10-latent-dimension linear model whose latent loadings are extracted
    as the linear-model counterfactual to the embedding metagene
- name: PyTorch
  role: Backend for GeneVector's neural embedding training (model.train with threshold=1e-6)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:75e8ac53e08156306b8263cc8653d1a65976724152f59bf19becf4a5f2393ac6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - LDVAE
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pca-loading-versus-embedding-metagene-comparison

## Summary

A benchmarking skill that contrasts gene-embedding-derived metagenes (e.g. GeneVector Leiden clusters of gene vectors) against linear-model loadings (PCA components, LDVAE latent loadings) on the same scRNA-seq data to test whether an embedding captures a phenotype program (e.g. the ISG program in interferon beta-stimulated PBMCs) without confounding by cell type, whereas linear loadings conflate it with cell-type markers.

## When to use

Use when a gene-embedding workflow has identified a candidate metagene (e.g. an IFIT1/ISG15/OAS1 cluster in interferon beta-stimulated PBMCs) and you need to demonstrate whether the program is cell-type-independent or an artifact of linear-model loading structure.

## When NOT to use

- Input data has already been normalized/log-transformed and scaled into a feature table — GeneVector requires raw counts in .X for its mutual-information discretization (bin 0 = zero expression, nonzero split into 10 quantile-based bins)
- The goal is batch correction or cell type annotation rather than evaluating whether a transcriptional program is confounded by cell type in linear loadings
- The phenotype program of interest involves gene repression or mutually exclusive expression — GeneVector's signed MI cannot distinguish activation from repression, so the contrast would be uninterpretable for such programs

## Inputs

- Scanpy AnnData object with raw counts in .X (e.g. interferon beta-stimulated PBMC scRNA-seq with stimulated/control labels and cell type annotations)
- Trained GeneVector embedding file genes.vec (input and output weight vectors)
- Cell type annotation labels in adata.obs (e.g. T cell, myeloid, B cell)
- Condition label (stimulated vs control) per cell

## Outputs

- Metagene membership table mapping Leiden clusters to genes (e.g. ISG cluster: IFIT1, IFIT2, IFIT3, ISG15, ISG20, RSAD2, OAS1, MX1)
- Per-cell-type stimulated-versus-control scaled metagene expression values
- PCA PC2 contribution table and 10-latent-dimension LDVAE loading table showing co-occurrence of CXCL10, IFITM3, PRF1, LAG3 with ISG loadings
- Summary CSV (e.g. isg_metagene_summary.csv) with the embedding-versus-linear-model contrast

## How to apply

First train the gene embedding on raw-count AnnData (GeneVectorDataset on .X, GeneVector model with emb_dimension=100, train 1000 iterations at threshold=1e-6, output genes.vec), then load it with GeneEmbedding(dataset, vector='average') — averaging input and output weights is recommended — and rank neighbors of a seed gene (e.g. compute_similarities('IFIT1')). Cluster the embedding into metagenes via get_adata(resolution=40) and get_metagenes(gdata), selecting the Leiden cluster containing the expected program genes (IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1 for the ISG program). Compute mean scaled metagene expression per cell and compare stimulated versus control cells within each annotated cell type to verify the program is elevated across all cell types rather than in one lineage. As the counterfactual, fit PCA and a 10-latent-dimension LDVAE model on the same data and extract PC2 contributions and latent loadings, recording whether cell-type-specific genes (myeloid CXCL10/IFITM3, T cell PRF1/LAG3) co-occur with the program's genes in the linear loadings. Write metagene membership, per-cell-type stimulated-vs-control scaled expression, and the PCA/LDVAE contrast table (e.g. isg_metagene_summary.csv) as the comparison output.

## Related tools

- **GeneVector** (Trains the gene embedding (GeneVectorDataset, GeneVector model, emb_dimension=100) and derives metagenes via GeneEmbedding.get_adata(resolution=40) and get_metagenes) — https://github.com/nceglia/genevector
- **Scanpy** (Hosts the raw-count AnnData input and variable-gene selection (seurat_v3 flavor) / entropy-based quality control before embedding; used for per-cell-type scaled expression and visualization)
- **LDVAE** (Fits the 10-latent-dimension linear model whose latent loadings are extracted as the linear-model counterfactual to the embedding metagene)
- **PyTorch** (Backend for GeneVector's neural embedding training (model.train with threshold=1e-6))

## Examples

```
gene_embed = GeneEmbedding("genes.vec", dataset, vector="average"); gdata = gene_embed.get_adata(resolution=40); mgs = gene_embed.get_metagenes(gdata)
```

## Evaluation signals

- The embedding metagene selected at resolution=40 contains only interferon pathway genes (IFIT1, IFIT2, IFIT3, ISG15, ISG20, RSAD2, OAS1, MX1) with no cell-type markers among its members
- Mean scaled metagene expression is elevated in stimulated versus control cells within every annotated cell type, confirming cell-type independence
- PCA PC2 contributions and LDVAE loadings show ISG genes co-occurring with myeloid genes (CXCL10, IFITM3) and T cell markers (PRF1, LAG3), reproducing the expected linear-model confounding
- compute_similarities('IFIT1') ranks other ISG genes (e.g. IFIT2, IFIT3, ISG15, RSAD2) as top nearest neighbors in the averaged-weight gene embedding
- Summary CSV contains the three required blocks: metagene membership, per-cell-type stimulated-vs-control expression, and the PCA/LDVAE contrast table

## Limitations

- GeneVector cannot distinguish activation from repression because its mutual information is agnostic to the direction of the statistical dependency, so only activation-type programs (like ISG induction) are valid contrast targets
- Empirical MI estimation is limited by the number of cells, count fidelity, and the discretization strategy (10 quantile bins), which affects metagene stability at low cell counts
- MI computation is computationally expensive; without a fast backend (numba, Rust, or CUDA) a 2000-gene x 5000-cell run took ~26 hours in legacy v0.2
- Benchmarking metrics referenced in the article (ARI, kBET, cLISI, silhouette score) are only in Supplementary Fig. 6A–L, so quantitative comparison metrics may not be fully reproducible from the main text

## Evidence

- [discussion] In interferon-stimulated PBMCs, we identify a cell type independent ISG metagene: "In interferon-stimulated PBMCs, we identify a cell type independent ISG metagene"
- [readme] Training produces two vector files (input and output weights). Using the average of both weights is recommended for best results.: [evidence span withheld — anchor status: external_doc]
- [discussion] The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [discussion] GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency: "GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency"
- [methods] model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100) model.train(1000, threshold=1e-6): "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
