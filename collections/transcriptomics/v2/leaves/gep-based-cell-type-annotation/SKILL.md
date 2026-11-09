---
name: gep-based-cell-type-annotation
description: Use when a novel single-cell count matrix should be annotated per cell
  using reference marker GEPs (e.g., mca.marker.geps.txt) instead of clustering, and
  annotation purity against reference labels must be quantified.
when_to_use_negative:
- The target dataset contains cell types absent from the reference GEP set (e.g.,
  TMS cells annotated with MCA GEPs become 'unknown', or high-resolution neuronal
  subtypes in Mouse Brain not covered by MCA GEPs).
- A clustering-based annotation or cross-modality (ATAC-seq, proteomics) approach
  is required — SingleCellGGM/CellGEP relies exclusively on transcriptome data.
- Gene identifiers in the count matrix do not match the GEP file format (Ensembl vs
  symbol mismatch).
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellGEP
  role: R package whose average_by_gep and cellgep functions compute GEP-averaged
    expression and assign cell types via dynamic cutoff and neighbor voting
  repo: https://github.com/MaShisongLab/cellgep
- name: SingleCellGGM
  role: Identifies the marker GEPs in the reference dataset (e.g., MCA) used for annotation
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: Seurat
  role: Creates the Seurat object, runs PCA/UMAP, and computes cell neighbors with
    FindNeighbors(return.neighbor = T)
- name: edgeR
  role: Required dependency sourced with cellgep.R for computing GEP-averaged expression
- name: graph
  role: Required R dependency for running the cellgep annotation function
- name: RBGL
  role: Required R dependency for running the cellgep annotation function
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:45f6d3cd739e4bff1935e2199a780c05f4d6df58ec2c060b891162f81650c985
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/cellgep
  tools:
  - CellGEP
  - RBGL
  - Seurat
  - SingleCellGGM
  - edgeR
  - graph
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gep-based-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Direct cell-type annotation of individual single-cell transcriptome cells using pre-identified marker gene expression programs (GEPs), without cell clustering, via the CellGEP R package (cellgep function) with GEP-averaged expression, a dynamic expression cutoff, and KNN neighbor voting.

## When to use

Use when a novel single-cell count matrix should be annotated per cell using reference marker GEPs (e.g., mca.marker.geps.txt) instead of clustering, and annotation purity against reference labels must be quantified.

## When NOT to use

- The target dataset contains cell types absent from the reference GEP set (e.g., TMS cells annotated with MCA GEPs become 'unknown', or high-resolution neuronal subtypes in Mouse Brain not covered by MCA GEPs).
- A clustering-based annotation or cross-modality (ATAC-seq, proteomics) approach is required — SingleCellGGM/CellGEP relies exclusively on transcriptome data.
- Gene identifiers in the count matrix do not match the GEP file format (Ensembl vs symbol mismatch).

## Inputs

- Novel single-cell gene count matrix with genes as row.names (format matching the GEP file, e.g., Ensembl IDs)
- Marker GEP table (2 columns: gene name, GEP name), e.g., 'data/mca.marker.geps.txt' or 'mca.marker.symbl.geps.txt'
- Neighbor index matrix (nn.idx) from Seurat FindNeighbors(return.neighbor = T) on the same dataset

## Outputs

- Per-cell annotation table (anno$annotation) with columns GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, GEP_Ori, and N1..N20 neighbor votes
- Purity/ARI/NMI metrics against reference labels (e.g., cellgep_annotation_metrics.csv)
- DimPlot of cells grouped by assigned marker_gep

## How to apply

Build a Seurat object for the novel dataset, run PCA and UMAP, and compute neighbors with FindNeighbors(return.neighbor = T), storing nn.idx as the neighbor matrix (20 neighbors). Load marker GEPs from 'data/mca.marker.geps.txt' (Ensembl ID version; use the symbol version for gene symbols) and compute GEP-averaged expression on the count matrix with average_by_gep, which requires edgeR and sourcing cellgep.R. Run cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T): a GEP is called expressed when its averaged value meets a dynamically determined fraction of its maximum, the highest-expressed GEP wins per cell, and KNN neighbor voting consolidates labels. Validate by comparing annotation purity against reference cell labels — e.g., on MCA, M1 should annotate 1,120/1,125 (99.6%) oligodendrocytes and M3 99.2% of 1,886 T cells — and report ARI/NMI against cluster annotations. Visualize assignments with DimPlot(group.by = 'marker_gep', label = T).

## Related tools

- **CellGEP** (R package whose average_by_gep and cellgep functions compute GEP-averaged expression and assign cell types via dynamic cutoff and neighbor voting) — https://github.com/MaShisongLab/cellgep
- **SingleCellGGM** (Identifies the marker GEPs in the reference dataset (e.g., MCA) used for annotation) — https://github.com/MaShisongLab/SingleCellGGM
- **Seurat** (Creates the Seurat object, runs PCA/UMAP, and computes cell neighbors with FindNeighbors(return.neighbor = T))
- **edgeR** (Required dependency sourced with cellgep.R for computing GEP-averaged expression)
- **graph** (Required R dependency for running the cellgep annotation function)
- **RBGL** (Required R dependency for running the cellgep annotation function)

## Examples

```
require(Seurat); require(edgeR); require(graph); require(RBGL); source('cellgep.R'); geps = read.table('data/mca.marker.geps.txt'); geps_expression = average_by_gep(expression, geps, 'mca', 'novel_sc_dataset'); anno = cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T)
```

## Evaluation signals

- M1 annotation purity: 1,120 of 1,125 cells (99.6%) assigned to oligodendrocytes must match the reference label within a small tolerance
- M3 annotation purity: 99.2% of the 1,886 annotated cells must be T cells
- Annotation table schema check: columns GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, GEP_Ori, N1..N20 present with 'na' for non-voting neighbors
- ARI and NMI computed against independent cluster annotations (e.g., scDHA, scMGCA) should be reported in cellgep_annotation_metrics.csv
- Each cell receives exactly one winning GEP (highest-expressed GEP among those exceeding its dynamic cutoff)

## Limitations

- Label transfer is constrained by the cell-type coverage of the reference dataset, leaving some target cells annotated as 'unknown' (e.g., TMS cells annotated with MCA GEPs).
- Annotating the Mouse Brain dataset with MCA GEPs would be compromised by the absence of high-resolution neuronal cell types in MCA.
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities (ATAC-seq, single-cell proteomics).

## Evidence

- [results] A GEP is called expressed when its averaged value meets the dynamically determined cutoff (fraction of its maximum): [evidence span withheld — anchor status: not_found]
- [results] of 1,125 cells annotated by M1, 1,120 (99.6%) are oligodendrocytes, and 99.2% of the 1,886 cells annotated by M3 are T cells: [evidence span withheld — anchor status: not_found]
