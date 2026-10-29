---
name: single-cell-dataset-label-matching
description: 'Use when a single-cell count matrix needs a biologically interpretable,
  cell-type-separating embedding: e.g., the MCA dataset must be embedded via the 137
  marker GEPs and benchmarked against PCA, scDHA, or scMGCA using Davies-Bouldin and
  Calinski-Harabasz scores.'
when_to_use_negative:
- The input is already an embedding or feature-reduced table rather than a raw gene
  count matrix — average_by_gep expects a gene count matrix with gene names as row
  names.
- Gene identifiers of the dataset do not match the GEP table format (e.g., Ensembl
  IDs when only the symbol-version GEP file is available, or vice versa), since gene-name
  matching is required for averaging.
- The goal is discovering novel cell-type structure rather than matching cells to
  the known GEP/cell-type vocabulary of the reference dataset; the article notes that
  label transfer is constrained by the cell-type coverage of the source dataset, leaving
  uncovered cells annotated as 'unknown'.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellGEP
  role: Provides average_by_gep, which computes per-cell average expression over each
    of the 137 marker GEPs to build the cell-by-GEP embedding matrix
  repo: github.com/MaShisongLab/CellGEP
- name: Seurat
  role: Creates the single-cell object, scales the GEP-averaged matrix, and computes
    UMAP embeddings evaluated by the DBI/CH metrics
  repo: https://satijalab.org/seurat/
- name: SingleCellGGM
  role: 'Upstream source of the marker GEPs: identifies GGM-derived gene co-expression
    modules (GEPs) from the reference MCA dataset used for averaging'
  repo: MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:7cf9da04faa24d234287a0b561119dea4a940b256adfe63b0233671ef2974530
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  tools:
  - CellGEP
  - Seurat
  - SingleCellGGM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-dataset-label-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces single-cell transcriptome dimensionality by averaging gene expression over marker gene expression programs (GEPs) — e.g., the 137 MCA GEPs identified by SingleCellGGM — so cell-type identity is directly encoded in the embedding, enabling comparison and label matching across datasets against methods like PCA, scDHA, and scMGCA.

## When to use

Use when a single-cell count matrix needs a biologically interpretable, cell-type-separating embedding: e.g., the MCA dataset must be embedded via the 137 marker GEPs and benchmarked against PCA, scDHA, or scMGCA using Davies-Bouldin and Calinski-Harabasz scores.

## When NOT to use

- The input is already an embedding or feature-reduced table rather than a raw gene count matrix — average_by_gep expects a gene count matrix with gene names as row names.
- Gene identifiers of the dataset do not match the GEP table format (e.g., Ensembl IDs when only the symbol-version GEP file is available, or vice versa), since gene-name matching is required for averaging.
- The goal is discovering novel cell-type structure rather than matching cells to the known GEP/cell-type vocabulary of the reference dataset; the article notes that label transfer is constrained by the cell-type coverage of the source dataset, leaving uncovered cells annotated as 'unknown'.

## Inputs

- MCA (mouse cell atlas) gene count matrix with gene names as row identifiers
- Marker GEP table from 'data/mca.marker.geps.txt' (column 1: gene ID, column 2: GEP name; 137 GEPs) or 'mca.marker.symbl.geps.txt' for gene-symbol IDs
- Original MCA cell-type labels for embedding evaluation
- Competing embeddings (PCA, scDHA, scMGCA) of the same dataset for benchmarking

## Outputs

- Cell-by-GEP average expression matrix (cells x 137 GEPs) from average_by_gep
- Scaled GEP-based UMAP embedding computed with Seurat
- Comparison table of Davies-Bouldin Index and Calinski-Harabasz scores saved as embedding_comparison_metrics.csv

## How to apply

Load the single-cell gene count matrix and the marker GEP table (e.g., 'data/mca.marker.geps.txt' with 137 GEPs from the MCA dataset; use 'mca.marker.symbl.geps.txt' if genes are in symbol rather than Ensembl ID format). Compute a cell-by-GEP matrix by per-cell average expression over each GEP via the CellGEP function average_by_gep (gep_origin='mca'), then scale this matrix and compute UMAP coordinates with Seurat. Quantify cell-type separation by computing the Davies-Bouldin Index (lower is better) and Calinski-Harabasz score (higher is better) of the GEP-based embedding against original cell-type labels. Re-run the identical scaling/UMAP/metric steps on PCA, scDHA, and scMGCA embeddings of the same cells and labels so all four methods are compared on identical inputs. The rationale is that each GEP is a functionally enriched, cell-type-specific gene module, so averaging by GEP yields dimensions whose values directly indicate which cell-type program a cell expresses — which also underpins label transfer, since the same GEPs annotate cells across datasets without clustering.

## Related tools

- **CellGEP** (Provides average_by_gep, which computes per-cell average expression over each of the 137 marker GEPs to build the cell-by-GEP embedding matrix) — github.com/MaShisongLab/CellGEP
- **Seurat** (Creates the single-cell object, scales the GEP-averaged matrix, and computes UMAP embeddings evaluated by the DBI/CH metrics) — https://satijalab.org/seurat/
- **SingleCellGGM** (Upstream source of the marker GEPs: identifies GGM-derived gene co-expression modules (GEPs) from the reference MCA dataset used for averaging) — MaShisongLab/SingleCellGGM

## Examples

```
require(edgeR); source('cellgep.R'); geps = read.table('data/mca.marker.geps.txt'); geps_expression = average_by_gep(expression, geps, 'mca', 'novel_sc_dataset')
```

## Evaluation signals

- The GEP-based embedding must achieve a lower Davies-Bouldin Index and a higher Calinski-Harabasz score than the PCA, scDHA, and scMGCA embeddings on the same labels (paper's reported favorable performance).
- The cell-by-GEP matrix from average_by_gep must have exactly 137 GEP columns (one per marker GEP) and one row per cell.
- All four methods must be evaluated with identical metrics and identical MCA cell-type labels before comparison.
- The final deliverable 'embedding_comparison_metrics.csv' must contain a DBI/CH score pair for each of the four methods (GEP, PCA, scDHA, scMGCA).
- Gene names in the count matrix must match the gene-ID format of the GEP file used ('mca.marker.geps.txt' for Ensembl IDs, 'mca.marker.symbl.geps.txt' for symbols).

## Limitations

- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics.
- Label transfer is constrained by the cell-type coverage of the source dataset: annotating TMS with MCA GEPs left some cells annotated as 'unknown'.
- High-resolution neuronal subtypes absent from MCA would compromise annotation of brain datasets; GEP-based GO/MP enrichment for the Mouse Brain dataset offered limited insight into functional differences among neuronal subsets.
- SingleCellGGM recommends large cell numbers (>5000) and pre-filtering of low-expression genes before GEP identification.

## Evidence
