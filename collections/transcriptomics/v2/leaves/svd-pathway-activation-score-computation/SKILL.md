---
name: svd-pathway-activation-score-computation
description: Use when a normalized gene-by-cell scRNA-seq matrix must be transformed
  into a pathway-by-cell activity matrix (pca_scCell_mat) prior to downstream association
  testing, or when quantifying how pathway-level aggregation reduces single-cell sparsity.
when_to_use_negative:
- The input is already a pathway-level or feature-reduced matrix, or the analysis
  goal is gene-level (not pathway-level) inference
- Pathways of interest fall outside the 5-300 gene size window the paper validated
  (smaller than 5 or larger than 300 genes)
- The scRNA-seq matrix has not been normalized (scPagwas expects the Seurat 'RNA'
  assay already normalized with LogNormalize)
tools:
- name: scPagwas
  role: Provides Single_data_input and Pathway_pcascore_run plus the Genes_by_pathway_kegg
    pathway list to compute SVD-based first-principal-component pathway activity scores
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat 4.3.0
  role: Normalizes the RNA assay (NormalizeData, LogNormalize, scale.factor = 10000)
    before the SVD-based PAS transformation
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:6773d78d39fa1497817dfd8113c613349b17532ef8d263dc8b64f51b7894433b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/dengchunyu/scPagwas
  tools:
  - Seurat 4.3.0
  - scPagwas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# svd-pathway-activation-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pathway activity scores (PAS) by applying SVD to a normalized gene-by-cell scRNA-seq matrix and extracting the first principal component of each KEGG pathway's gene-by-cell sub-matrix, collapsing thousands of sparse gene measurements into a dense pathway-by-cell matrix that reduces technical noise and sparsity.

## When to use

Use when a normalized gene-by-cell scRNA-seq matrix must be transformed into a pathway-by-cell activity matrix (pca_scCell_mat) prior to downstream association testing, or when quantifying how pathway-level aggregation reduces single-cell sparsity.

## When NOT to use

- The input is already a pathway-level or feature-reduced matrix, or the analysis goal is gene-level (not pathway-level) inference
- Pathways of interest fall outside the 5-300 gene size window the paper validated (smaller than 5 or larger than 300 genes)
- The scRNA-seq matrix has not been normalized (scPagwas expects the Seurat 'RNA' assay already normalized with LogNormalize)

## Inputs

- Normalized gene-by-cell scRNA-seq matrix from a Seurat object (e.g. Seu_Hema_data.rds, 35,582 cells) with LogNormalize and scale.factor = 10000
- KEGG pathway gene list (Genes_by_pathway_kegg) from the scPagwas package
- Pathway size bounds: min.pathway.size = 5, max.pathway.size = 300

## Outputs

- Single-cell PAS matrix (pca_scCell_mat): one row per retained KEGG pathway, one column per cell
- Cell-type PAS matrix (pca_cell_df)
- pas_matrix.csv (written pca_scCell_mat)
- sparsity_comparison.csv comparing fraction of zero/near-zero entries in gene-level vs pathway-level matrices

## How to apply

Normalize the RNA assay of the Seurat object with Seurat NormalizeData using LogNormalize and scale.factor = 10000 (e.g. the BMMC Seu_Hema_data object with 35,582 cells). Load the Genes_by_pathway_kegg pathway list from scPagwas and restrict it to pathways with 5-300 genes (min.pathway.size = 5, max.pathway.size = 300), keeping only pathway genes present in the single-cell matrix. Run Single_data_input and Pathway_pcascore_run from scPagwas to compute the SVD-based first principal component of each pathway's gene-by-cell sub-matrix, yielding the single-cell PAS matrix (pca_scCell_mat, pathways x cells) and the cell-type PAS matrix (pca_cell_df). The rationale, per the paper, is that collapsing functional actions of genes in the same biological pathway enhances statistical power and reduces technical and biological confounders of scRNA-seq data.

## Related tools

- **scPagwas** (Provides Single_data_input and Pathway_pcascore_run plus the Genes_by_pathway_kegg pathway list to compute SVD-based first-principal-component pathway activity scores) — https://github.com/dengchunyu/scPagwas
- **Seurat 4.3.0** (Normalizes the RNA assay (NormalizeData, LogNormalize, scale.factor = 10000) before the SVD-based PAS transformation) — https://github.com/satijalab/seurat

## Examples

```
Pagwas <- Single_data_input(Pagwas, assay="RNA", Single_data=Seu_Hema_data); Pagwas <- Pathway_pcascore_run(Pagwas, Pathway_list=Genes_by_pathway_kegg, min.pathway.size=5, max.pathway.size=300)
```

## Evaluation signals

- The PAS matrix (pca_scCell_mat) has exactly one row per retained KEGG pathway and one column per cell of the input Seurat object
- Measured fraction of zero/near-zero entries in the PAS matrix is lower than in the gene-level normalized matrix (sparsity_comparison.csv)
- All retained pathways contain between 5 and 300 genes after intersecting pathway genes with genes present in the single-cell matrix
- Pathway-level expression variances show reduced technical noise relative to individual gene-level magnitudes, consistent with the paper's finding

## Limitations

- Pathway analysis is restricted to canonical KEGG pathways only, because these have been experimentally validated
- Statistical association at pathway level does not imply causality and may reflect indirect identification of causal associations
- The transformation requires substantial memory for large single-cell datasets in R; a Python version (scPagwas_py) synchronized with scPagwas2.0 was developed to address this

## Evidence
