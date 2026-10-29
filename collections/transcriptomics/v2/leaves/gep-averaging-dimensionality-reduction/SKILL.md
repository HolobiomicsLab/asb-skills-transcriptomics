---
name: gep-averaging-dimensionality-reduction
description: Use when single-cell transcriptomes (e.g. MCA count matrix) need a low-dimensional,
  biologically interpretable embedding built from known marker GEPs, and when benchmarking
  that embedding against PCA, scDHA, or scMGCA with clustering metrics.
when_to_use_negative:
- The target cells' types are not covered by the source GEPs (e.g. high-resolution
  neuronal subtypes absent from MCA GEPs would compromise annotation/averaging precision)
- The dataset lacks identified GEPs and you only need an off-the-shelf latent-factor
  reduction with no interpretability requirement — PCA or similar is simpler
- Gene IDs in the count matrix cannot be matched to the GEP file format (Ensembl vs
  symbol mismatch would silently average over wrong genes)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellGEP
  role: Provides the average_by_gep function that computes per-cell GEP-averaged expression
    values
  repo: https://github.com/MaShisongLab/CellGEP
- name: Seurat
  role: Scaling of the GEP-averaged matrix, UMAP embedding, and visualization of the
    reduced space
- name: SingleCellGGM
  role: Upstream method that identified the marker GEPs (co-expression modules) used
    for averaging
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:22f931d15e31e209db0d0e4be2f0219f6cae82e2356cbc304c9d513d8fa0c632
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/CellGEP
  tools:
  - CellGEP
  - RBGL
  - Seurat
  - SingleCellGGM
  - edgeR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gep-averaging-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A dimension-reduction method from Xu et al. 2024 that embeds single-cell data by averaging gene expression over SingleCellGGM-derived gene expression programs (GEPs) instead of gene-level latent factors, enhancing the interpretability of single-cell analysis results.

## When to use

Use when single-cell transcriptomes (e.g. MCA count matrix) need a low-dimensional, biologically interpretable embedding built from known marker GEPs, and when benchmarking that embedding against PCA, scDHA, or scMGCA with clustering metrics.

## When NOT to use

- The target cells' types are not covered by the source GEPs (e.g. high-resolution neuronal subtypes absent from MCA GEPs would compromise annotation/averaging precision)
- The dataset lacks identified GEPs and you only need an off-the-shelf latent-factor reduction with no interpretability requirement — PCA or similar is simpler
- Gene IDs in the count matrix cannot be matched to the GEP file format (Ensembl vs symbol mismatch would silently average over wrong genes)

## Inputs

- Gene count matrix of the single-cell dataset (e.g. MCA), with gene names matching the GEP file format
- Marker GEP table with gene name and GEP name columns (data/mca.marker.geps.txt; Ensembl-ID version, or mca.marker.symbl.geps.txt for symbols)
- Original cell-type labels for the dataset
- Comparison embeddings (PCA, scDHA, scMGCA) of the same dataset

## Outputs

- Cell-by-GEP averaged expression matrix (137 GEPs for the MCA example)
- Scaled GEP-based UMAP embedding
- Comparison table of Davies-Bouldin Index and Calinski-Harabasz scores for all four embeddings (embedding_comparison_metrics.csv)

## How to apply

Load the gene count matrix and the 137 marker GEPs from data/mca.marker.geps.txt, then compute per-cell average expression over each GEP with the CellGEP function average_by_gep (gep_origin='mca'), yielding a cell-by-GEP matrix. Scale the GEP-averaged matrix and compute UMAP embeddings in Seurat, following the same procedure used for the PCA embedding. Evaluate the GEP-based UMAP against original cell-type labels using the Davies-Bouldin Index and Calinski-Harabasz score, and repeat the identical steps for PCA, scDHA, and scMGCA embeddings so all four are compared with identical metrics and labels. The rationale is that averaging over functionally enriched co-expression modules (GEPs) replaces opaque gene-space factors with interpretable, cell-type-specific programs.

## Related tools

- **CellGEP** (Provides the average_by_gep function that computes per-cell GEP-averaged expression values) — https://github.com/MaShisongLab/CellGEP
- **Seurat** (Scaling of the GEP-averaged matrix, UMAP embedding, and visualization of the reduced space)
- **SingleCellGGM** (Upstream method that identified the marker GEPs (co-expression modules) used for averaging) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
geps = read.table('data/mca.marker.geps.txt'); geps_expression = average_by_gep(expression, geps, 'mca', 'mca')
```

## Evaluation signals

- The GEP-based embedding should show a lower Davies-Bouldin Index than the PCA, scDHA, and scMGCA embeddings of the same dataset
- The GEP-based embedding should show a higher Calinski-Harabasz score than all three comparison methods
- The cell-by-GEP matrix must contain one averaged column per GEP (137 for the MCA marker GEPs) with cells as rows
- All four embeddings must be evaluated with identical metrics against identical original cell-type labels
- Results table saved as embedding_comparison_metrics.csv with a DBI/CH score pair per method

## Limitations

- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics
- Coverage is limited to the cell types represented in the source GEPs; cell types absent from the reference dataset cannot be separated by this reduction

## Evidence

- [results] benchmark metrics: [evidence span withheld — anchor status: not_found]
