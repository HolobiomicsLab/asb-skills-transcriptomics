---
name: kegg-go-pathway-enrichment-analysis
description: Use when you have a trained single-cell graph-embedding autoencoder (e.g.,
  scMGCA on 'Qx Limb Muscle' or PDAC scRNA-seq data) and need to interpret its compressed
  transcriptomic space by identifying biological functions of its top pivotal genes.
when_to_use_negative:
- The analysis goal is cell clustering or batch-effect correction rather than functional
  interpretation of the autoencoder's latent space
- The input is already a candidate gene list not derived from an autoencoder weight
  matrix (the ranking pipeline W2 → W1 → r2 would not apply)
- The data are bulk transcriptomics rather than single-cell data from the compressed
  transcription space of a scRNA-seq model
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMGCA
  role: Graph convolutional autoencoder that produces the latent embedding and weight
    matrices from which the top 200 pivotal genes are ranked
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing: normalizes the scRNA-seq gene expression matrix and selects
    highly variable genes'
- name: TensorFlow
  role: Framework in which the scMGCA core model is built and trained
- name: scikit-learn
  role: Supplies K-means and spectral clustering used to initialize the clustering
    centers of the self-optimized clustering
- name: ClusterProfiler
  role: Performs KEGG and GO over-representation analysis on the top 200 pivotal genes
    with pvalueCutoff=0.01
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:63530b8fc8feb35226502bdae17897a92329b6a332787d74def73ea539558835
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterProfiler
  - SCANPY
  - TensorFlow
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kegg-go-pathway-enrichment-analysis

## Summary

Functional-genomics interpretation of a deep single-cell model (scMGCA) by ranking pivotal genes from the graph-embedding autoencoder's weight matrices and running KEGG/GO over-representation analysis on the top 200 with ClusterProfiler at pvalueCutoff=0.01.

## When to use

Use when you have a trained single-cell graph-embedding autoencoder (e.g., scMGCA on 'Qx Limb Muscle' or PDAC scRNA-seq data) and need to interpret its compressed transcriptomic space by identifying biological functions of its top pivotal genes.

## When NOT to use

- The analysis goal is cell clustering or batch-effect correction rather than functional interpretation of the autoencoder's latent space
- The input is already a candidate gene list not derived from an autoencoder weight matrix (the ranking pipeline W2 → W1 → r2 would not apply)
- The data are bulk transcriptomics rather than single-cell data from the compressed transcription space of a scRNA-seq model

## Inputs

- Preprocessed scRNA-seq count matrix (normalized, highly variable genes selected after removing genes not expressed in cells)
- Trained scMGCA latent embedding and GCN weight matrices W1 and W2 (128 latent dimensions)
- Ranked gene list (top 200 pivotal genes extracted from r2)

## Outputs

- Ranked gene list of the top 200 pivotal genes
- KEGG/GO over-representation analysis table of significant terms at adjusted p-value ≤ 0.01
- pivotal_genes_enrichment.csv combining ranked genes and enrichment results

## How to apply

Train scMGCA in TensorFlow (two GCN layers, multinomial-based decoder, KL-divergence self-optimized clustering with centers initialized by the better of K-means and spectral clustering from scikit-learn) on a normalized scRNA-seq matrix with highly variable genes selected after filtering out genes not expressed in cells. Compute the standard deviation across the 128 latent dimensions of W2, retain the top 50% (128 × 50% = 64) highest-variance dimensions to filter W1, then rank genes using the standard deviation and weight of the latent embedded representation and extract the top 200 highly variable genes from r2. Run KEGG and GO over-representation analysis on these 200 pivotal genes with ClusterProfiler using pvalueCutoff=0.01, and write the ranked gene list and enrichment results to pivotal_genes_enrichment.csv. The rationale is that the genes weighted most heavily in the latent space summarize the principal compressed transcription space, so their enriched pathways reveal the regulatory mechanisms the model captures.

## Related tools

- **scMGCA** (Graph convolutional autoencoder that produces the latent embedding and weight matrices from which the top 200 pivotal genes are ranked) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing: normalizes the scRNA-seq gene expression matrix and selects highly variable genes)
- **TensorFlow** (Framework in which the scMGCA core model is built and trained)
- **scikit-learn** (Supplies K-means and spectral clustering used to initialize the clustering centers of the self-optimized clustering)
- **ClusterProfiler** (Performs KEGG and GO over-representation analysis on the top 200 pivotal genes with pvalueCutoff=0.01)

## Examples

```
library(clusterProfiler); go <- enrichGO(gene = pivotal_genes, pvalueCutoff = 0.01); kegg <- enrichKEGG(gene = pivotal_genes, pvalueCutoff = 0.01)
```

## Evaluation signals

- The W2 dimension filtering retains exactly 64 dimensions (128 × 50% = 64) of highest variance before ranking genes
- The extracted gene set contains exactly 200 highly variable genes taken from the top of the r2 ranking
- The enrichment output must contain significant KEGG/GO terms at adjusted p-value ≤ 0.01 that reproduce the reported tumor-associated and cell-signalling pathway hits
- The expected GO finding reproduces enrichment in chemotaxis and migration of neutrophils in biological processes
- pivotal_genes_enrichment.csv is written and contains both the ranked gene list and the enrichment results table

## Limitations

- Number of highly variable genes d, KNN neighbor count k, restart probability α, and loss weights γ1, γ2, γ3 are not specified numerically, hindering exact reproduction
- No software version numbers are reported for Python, TensorFlow, SCANPY, or the R packages used
- No specific dataset accessions or repository identifiers are provided; benchmark data are only described as coming from public repositories
- The stopping condition is only that the maximum number of iterations is achieved, without a stated iteration count

## Evidence

- [results] Top 200 genes selection from latent representation: "we selected the top 200 highly expressed genes using the standard deviation and weight of the latent embedded representation"
- [discussion] W2 dimension filtering rule: "we select the top 50% (128 × 50% = 64) of highest variance from"
- [discussion] Dropout-motivated gene filtering in preprocessing: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [results] Normalization and HVG selection precede modeling: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [readme] scMGCA package description: "`scMGCA` is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and"
- [abstract] Rationale for genomic interpretation of latent space: "we perform genomic interpretation on the key compressed transcriptomic space of the graph-embedding autoencoder"
