---
name: matrix-sparsity-comparison-analysis
description: Use when you need to transform a normalized gene-by-cell scRNA-seq matrix
  into a pathway-by-cell activity score matrix via SVD (PC1 per pathway) and empirically
  verify that the pathway-level representation is less sparse than the gene-level
  matrix.
when_to_use_negative:
- The goal is per-gene differential expression or gene-level trait association rather
  than pathway-level activity summarization — the SVD PC1 collapse discards individual
  gene resolution.
- The scRNA-seq matrix is raw/unnormalized counts — the SVD-based PAS computation
  assumes LogNormalize normalization with scale.factor = 10000 has been applied first.
- Pathway definitions other than KEGG are required and cannot be recast into the 5–300
  gene size range expected by min.pathway.size/max.pathway.size filtering.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
tools:
- name: scPagwas
  role: Provides Genes_by_pathway_kegg pathway list, Single_data_input, and Pathway_pcascore_run
    to compute SVD-based PC1 pathway activity scores (pca_scCell_mat, pca_cell_df)
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Normalizes the RNA assay of the Seurat object with NormalizeData (LogNormalize,
    scale.factor = 10000) prior to pathway SVD
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:067c20ae858ea47c99fec67c815ba87a1210a280b0126d073984f4218fa3f154
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
  - Seurat
  - scPagwas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# matrix-sparsity-comparison-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scPagwas's pathway activation transformation — computing the SVD-based first principal component (PAS) of each KEGG pathway's gene-by-cell sub-matrix — and quantitatively compares zero-entry fractions between the normalized gene-level matrix and the resulting pathway-by-cell PAS matrix to demonstrate reduced sparsity and technical noise.

## When to use

Use when you need to transform a normalized gene-by-cell scRNA-seq matrix into a pathway-by-cell activity score matrix via SVD (PC1 per pathway) and empirically verify that the pathway-level representation is less sparse than the gene-level matrix.

## When NOT to use

- The goal is per-gene differential expression or gene-level trait association rather than pathway-level activity summarization — the SVD PC1 collapse discards individual gene resolution.
- The scRNA-seq matrix is raw/unnormalized counts — the SVD-based PAS computation assumes LogNormalize normalization with scale.factor = 10000 has been applied first.
- Pathway definitions other than KEGG are required and cannot be recast into the 5–300 gene size range expected by min.pathway.size/max.pathway.size filtering.

## Inputs

- Seurat object (.rds) of scRNA-seq counts, e.g., BMMC Seu_Hema_data.rds (35,582 cells) with an 'RNA' assay
- scPagwas Genes_by_pathway_kegg KEGG pathway list (gene symbols per pathway)

## Outputs

- Single-cell PAS matrix (pca_scCell_mat): pathways × cells, one row per retained KEGG pathway and one column per cell
- Cell-type PAS matrix (pca_cell_df)
- pas_matrix.csv containing the pathway-by-cell PAS matrix
- sparsity_comparison.csv with zero/near-zero entry fractions for gene-level vs pathway-level matrices

## How to apply

Load the scRNA-seq Seurat object (e.g., the BMMC Seu_Hema_data.rds with 35,582 cells) and normalize the RNA assay with Seurat NormalizeData using LogNormalize and scale.factor = 10000. Restrict scPagwas's Genes_by_pathway_kegg pathway list to pathways with at least 5 and at most 300 genes (min.pathway.size = 5, max.pathway.size = 300), intersecting pathway genes with genes present in the single-cell matrix. Run Single_data_input and Pathway_pcascore_run from scPagwas to compute, for each retained pathway, the SVD-based first principal component over its gene-by-cell sub-matrix, yielding the single-cell PAS matrix (pca_scCell_mat, pathways × cells) and the cell-type PAS matrix (pca_cell_df). Export pca_scCell_mat (e.g., to pas_matrix.csv), then compute the fraction of zero/near-zero entries in the gene-level normalized matrix versus the PAS matrix and save the comparison (e.g., sparsity_comparison.csv). The rationale is that collapsing genes into pathway activity scores enhances statistical power and reduces technical and biological confounders of scRNA-seq data.

## Related tools

- **scPagwas** (Provides Genes_by_pathway_kegg pathway list, Single_data_input, and Pathway_pcascore_run to compute SVD-based PC1 pathway activity scores (pca_scCell_mat, pca_cell_df)) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Normalizes the RNA assay of the Seurat object with NormalizeData (LogNormalize, scale.factor = 10000) prior to pathway SVD) — https://github.com/satijalab/seurat

## Examples

```
Seu_Hema_data <- NormalizeData(Seu_Hema_data, normalization.method = "LogNormalize", scale.factor = 10000); Pagwas <- Single_data_input(Pagwas, Single_data = Seu_Hema_data, Pathway_list = Genes_by_pathway_kegg, min.pathway.size = 5, max.pathway.size = 300); Pagwas <- Pathway_pcascore_run(Pagwas); write.csv(Pagwas$pca_scCell_mat, "pas_matrix.csv")
```

## Evaluation signals

- Schema check: pca_scCell_mat must have exactly one row per retained KEGG pathway (after 5–300 gene filtering and gene-set intersection) and one column per cell.
- Sparsity invariant: measured zero/near-zero fraction of the PAS matrix must be strictly lower than that of the gene-level normalized matrix.
- Dimensionality check: gene-level matrix is genes × cells; PAS matrix is pathways × cells with the same cell order, enabling row/column-wise correspondence.
- Each pathway's PAS equals the SVD first principal component of that pathway's gene-by-cell sub-matrix, so scores are continuous (no zero-inflated entries) even for sparse input pathways.
- Normalization check: input matrix values must match LogNormalize output with scale.factor = 10000 before transformation.

## Limitations

- Pathway analysis is restricted to canonical KEGG pathways only; non-KEGG pathway definitions are not supported by the default pipeline.
- Pathways are limited to 5–300 genes (min.pathway.size = 5, max.pathway.size = 300); larger or smaller gene sets are excluded from the PAS matrix.
- The reduction to PC1 per pathway discards higher-order components, so it captures only the dominant mode of variation within each pathway.

## Evidence

- [results] SVD-based PC1 over KEGG pathway gene sets reduces sparsity: [evidence span withheld — anchor status: not_found]
