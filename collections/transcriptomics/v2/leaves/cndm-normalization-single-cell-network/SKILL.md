---
name: cndm-normalization-single-cell-network
description: Use when a c-CSN-transformed conditional network degree matrix (CNDM)
  must be normalized for compatibility with Seurat, t-SNE, SC3, or SIMLR clustering,
  e.g., to reproduce ARI = 0.73 on Tabula Muris droplet1 (GSE109774).
when_to_use_negative:
- The goal is causal gene-regulatory network construction or inferring directions
  of gene associations — c-CSN is explicitly not designed for this.
- The input is a plain gene expression matrix and the target is standard expression-level
  analysis (e.g., differential expression on counts) rather than transformed network-degree
  data.
- The goal is differentiation-trajectory direction or per-cell potency — use the CCSN-based
  NFE (network flow entropy) analysis instead of CNDM clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3481
tools:
- name: c-CSN
  role: Constructs the per-cell CCSNs and integrates them into the CNDM that this
    skill normalizes for downstream use
  repo: https://github.com/LinLi-0909/c-CSN
- name: Seurat
  role: Performs dimension reduction and clustering on the normalized CNDM (Tabula
    Muris, ARI = 0.73 benchmark)
- name: t-SNE
  role: Nonlinear dimension reduction used to visualize tissue-group separation of
    the normalized CNDM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:cbefdf21854f1472c84928003b4db202c7a9c8daf6cb4939c5140260cfd7634e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - Seurat
  - c-CSN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3481
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cndm-normalization-single-cell-network

## Summary

Normalizes the c-CSN-derived conditional network degree matrix (CNDM, m genes x n cells) so it behaves like a gene expression matrix for standard scRNA-seq pipelines, enabling Seurat/t-SNE dimension reduction and clustering that reproduce ARI = 0.73 on Tabula Muris droplet1.

## When to use

Use when a c-CSN-transformed conditional network degree matrix (CNDM) must be normalized for compatibility with Seurat, t-SNE, SC3, or SIMLR clustering, e.g., to reproduce ARI = 0.73 on Tabula Muris droplet1 (GSE109774).

## When NOT to use

- The goal is causal gene-regulatory network construction or inferring directions of gene associations — c-CSN is explicitly not designed for this.
- The input is a plain gene expression matrix and the target is standard expression-level analysis (e.g., differential expression on counts) rather than transformed network-degree data.
- The goal is differentiation-trajectory direction or per-cell potency — use the CCSN-based NFE (network flow entropy) analysis instead of CNDM clustering.

## Inputs

- c-CSN-derived conditional network degree matrix (CNDM, m genes x n cells) from the Tabula Muris droplet1 dataset (GSE109774)
- Raw scRNA-seq count matrix feeding the c-CSN transformation (m genes x n cells)
- Ground-truth cell-origin annotations for bladder, trachea, and spleen (for ARI validation)

## Outputs

- Normalized CNDM directly consumable by Seurat and other standard scRNA-seq pipelines
- PCA/t-SNE embeddings of cells visualized as a t-SNE map with three tissue-separated groups
- Per-cell Seurat cluster assignments
- Adjusted Rand index (ARI) of clusters vs. known cell origins, expected ~0.73 (0.65-0.80)

## How to apply

Integrate the G per-cell CCSNs into the degree matrix C-k = (1/G) sum Czg(k) = cijk and assemble the m genes x n cells CNDM. Normalize the CNDM using the c-CSN methods' normalization so that standard scRNA-seq tools accept it directly, since most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices. Run PCA (linear) and t-SNE (nonlinear) dimension reduction, then Seurat clustering (alternatives in the paper: K-means, hierarchical clustering, K-medoids, SC3, SIMLR). Decision point: on Tabula Muris droplet1 (>20,000 bladder, trachea, and spleen cells) the t-SNE map must segregate three dominant groups matching tissue origins. Compute the adjusted Rand index of Seurat clusters against known cell origins; acceptance is reproducing ARI = 0.73 within a tolerance of roughly 0.65-0.80. As a sanity comparison, the same pipeline on the raw gene expression matrix (GEM) should score lower ARI, because CNDM performs obviously better than GEM on all datasets.

## Related tools

- **c-CSN** (Constructs the per-cell CCSNs and integrates them into the CNDM that this skill normalizes for downstream use) — https://github.com/LinLi-0909/c-CSN
- **Seurat** (Performs dimension reduction and clustering on the normalized CNDM (Tabula Muris, ARI = 0.73 benchmark))
- **t-SNE** (Nonlinear dimension reduction used to visualize tissue-group separation of the normalized CNDM)

## Examples

```
library(Seurat); seu <- CreateSeuratObject(counts = cndm_norm); seu <- RunPCA(seu); seu <- RunTSNE(seu, dims = 1:2)
```

## Evaluation signals

- ARI of Seurat clusters vs. known bladder/trachea/spleen origins falls within 0.65-0.80, reproducing the reported ARI = 0.73 on Tabula Muris droplet1.
- t-SNE map of the normalized CNDM shows clear separation into three dominant groups matching the three tissue origins.
- The normalized CNDM is accepted by Seurat (CreateSeuratObject/RunPCA/RunTSNE) without reformatting, confirming gene-expression-matrix compatibility.
- Shape check: CNDM is m genes x n cells with n > 20,000 cells for Tabula Muris droplet1 (GSE109774).
- Clustering ARI on CNDM exceeds ARI on the raw gene expression matrix (GEM) under the identical pipeline, since CNDM performs obviously better than GEM on all datasets.

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase of building G per-cell CCSNs; the authors state a parallel computation scheme is desired.
- c-CSN is not designed to construct causal gene association networks, and the directions of the gene associations cannot be obtained.
- CNDM entries are gene-gene interaction degrees (cijk), not expression levels, so expression-scale interpretations do not transfer directly.

## Evidence

- [methods] cndm_definition: "an m × n matrix conditional network degree matrix (CNDM) is obtained"
- [methods] tabula_muris_dataset: "The Tabula Muris droplet1 dataset [47] comprising more than 20,000 cells from three tissues (bladder, trachea, and spleen)"
- [methods] dimension_reduction: "We used PCA [35] and t-SNE [36] which respectively represent linear and nonlinear methods, to perform dimension reduction"
- [methods] cndm_vs_gem: "CNDM performs obviously better than GEM on all datasets"
- [abstract] gem_compatibility: "Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
