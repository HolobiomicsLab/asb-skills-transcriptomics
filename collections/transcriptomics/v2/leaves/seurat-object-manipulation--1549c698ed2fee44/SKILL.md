---
name: seurat-object-manipulation
description: Use when a Seurat-format scRNA-seq object must be normalized (LogNormalize,
  scale.factor=10000) and passed to scPagwas for pathway activation transformation,
  or when comparing gene-level vs pathway-level matrix sparsity.
when_to_use_negative:
- The input is not a Seurat object with an 'RNA' assay, or cells/genes have not passed
  basic filtering (scPagwas expects min_clustercells=10 and pre-processed single-cell
  data)
- The goal is gene-level, per-gene scoring of GWAS association rather than pathway-level
  transformation (e.g. disease-specific gene sets as in scDRS/MAGMA-based workflows)
- The pathways of interest fall outside canonical, experimentally validated KEGG pathways
  or outside the 5-300 gene size window
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides the Seurat object format and the NormalizeData function (LogNormalize,
    scale.factor=10000) used to prepare the scRNA-seq matrix before PAS computation
  repo: https://github.com/satijalab/seurat
- name: scPagwas
  role: Supplies Genes_by_pathway_kegg, Single_data_input, and Pathway_pcascore_run
    to compute the SVD-based PAS matrix and cell-type PAS matrix from the normalized
    Seurat object
  repo: https://github.com/dengchunyu/scPagwas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:2de15504332fd6b65bb526724746ff1c38fd97f4dc37acb944078ed89dd0aeb3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/satijalab/seurat
  tools:
  - Seurat
  - scPagwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# seurat-object-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loading, normalizing, and gene-restricting a Seurat scRNA-seq object so that scPagwas can transform the normalized gene-by-cell matrix into a SVD-based pathway activity score (PAS) matrix over KEGG pathways. This collapses sparse gene-level expression into denser pathway-level scores, reducing technical noise.

## When to use

Use when a Seurat-format scRNA-seq object must be normalized (LogNormalize, scale.factor=10000) and passed to scPagwas for pathway activation transformation, or when comparing gene-level vs pathway-level matrix sparsity.

## When NOT to use

- The input is not a Seurat object with an 'RNA' assay, or cells/genes have not passed basic filtering (scPagwas expects min_clustercells=10 and pre-processed single-cell data)
- The goal is gene-level, per-gene scoring of GWAS association rather than pathway-level transformation (e.g. disease-specific gene sets as in scDRS/MAGMA-based workflows)
- The pathways of interest fall outside canonical, experimentally validated KEGG pathways or outside the 5-300 gene size window

## Inputs

- Seurat object (.rds) containing a raw or normalized RNA assay, e.g. Seu_Hema_data.rds (35,582 BMMC cells)
- Genes_by_pathway_kegg pathway list (gene symbols per KEGG pathway) bundled with the scPagwas R package
- scPagwas functions Single_data_input and Pathway_pcascore_run

## Outputs

- pca_scCell_mat: single-cell PAS matrix (retained KEGG pathways x cells), written to pas_matrix.csv
- pca_cell_df: cell-type-level PAS matrix
- sparsity_comparison.csv: fraction of zero/near-zero entries in the gene-level normalized matrix vs the pathway-level PAS matrix

## How to apply

Load the Seurat object (e.g. Seu_Hema_data.rds with 35,582 BMMC cells) and normalize the RNA assay with Seurat NormalizeData using LogNormalize and scale.factor=10000, since scPagwas expects scRNA-seq data in Seurat format with an 'RNA' assay that is already normalized. Load the Genes_by_pathway_kegg pathway list shipped with scPagwas and restrict it to pathways containing 5-300 genes (min.pathway.size=5, max.pathway.size=300), intersecting pathway genes with genes present in the single-cell matrix. Call Single_data_input and Pathway_pcascore_run from scPagwas to compute, for each retained KEGG pathway, the SVD-based first principal component of its gene-by-cell sub-matrix, yielding the single-cell PAS matrix (pca_scCell_mat, pathways x cells) and the cell-type PAS matrix (pca_cell_df). The rationale is that pathway activity scoring collapses the functional actions of genes in the same biological pathway, enhancing statistical power and reducing sparsity and technical noise inherent in single-cell expression data.

## Related tools

- **Seurat** (Provides the Seurat object format and the NormalizeData function (LogNormalize, scale.factor=10000) used to prepare the scRNA-seq matrix before PAS computation) — https://github.com/satijalab/seurat
- **scPagwas** (Supplies Genes_by_pathway_kegg, Single_data_input, and Pathway_pcascore_run to compute the SVD-based PAS matrix and cell-type PAS matrix from the normalized Seurat object) — https://github.com/dengchunyu/scPagwas

## Examples

```
Single_data <- readRDS("Seu_Hema_data.rds"); Single_data <- NormalizeData(Single_data, assay="RNA", normalization.method="LogNormalize", scale.factor=10000)
```

## Evaluation signals

- The PAS matrix (pca_scCell_mat) has exactly one row per retained KEGG pathway and one column per cell (35,582 for the BMMC dataset)
- All retained pathways satisfy the size filter: at least 5 and at most 300 genes after intersecting with genes in the single-cell matrix
- Measured sparsity (fraction of zero/near-zero entries) of the pathway-level PAS matrix is strictly lower than that of the gene-level normalized matrix
- The gene-level input matrix was normalized with LogNormalize and scale.factor=10000 before Pathway_pcascore_run
- sparsity_comparison.csv contains both gene-level and pathway-level sparsity values, with the pathway-level value smaller, consistent with the paper's finding

## Limitations

- Pathway analysis is restricted to canonical pathways identified in the KEGG database because these have been experimentally validated; other pathway resources are not covered
- Single-cell PAS computation in R has substantial memory demands; the authors provide a Python version (scPagwas_py) synchronized with scPagwas 2.0 for computational efficiency
- scPagwas expects single-cell data already in Seurat format with an 'RNA' assay that is normalized; passing raw unnormalized counts can invalidate the SVD-based PAS computation

## Evidence

- [readme] Seurat input expectation: [evidence span withheld — anchor status: external_doc]
