---
name: clustering-quality-metric-computation
description: Use when comparing two or more cell-by-embedding matrices of the same
  single-cell dataset against identical cell-type labels — e.g., benchmarking a 137-GEP
  dimension reduction against PCA, scDHA, or scMGCA embeddings.
when_to_use_negative:
- No ground-truth cell-type labels exist for the cells — DBI and CH both require pre-assigned
  labels to score cluster/embedding quality
- The compared embeddings were built on different subsets of cells or evaluated against
  different label sets — scores are then not comparable
- The goal is a clustering algorithm itself rather than scoring separability of an
  existing embedding against known labels
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellGEP
  role: Provides average_by_gep to compute the cell-by-GEP matrix (137 MCA marker
    GEPs) whose UMAP embedding is one of the methods being scored
  repo: https://github.com/MaShisongLab/cellgep
- name: Seurat
  role: Scales the GEP-averaged expression matrix and computes the UMAP embedding
    that is then evaluated with DBI/CH metrics
  repo: https://satijalab.org/seurat/
- name: SingleCellGGM
  role: Origin of the 137 marker GEPs (identified from the MCA dataset via single-cell
    GCN analysis) used for the GEP-averaged embedding under comparison
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:88d0be84d195a1b935d3c08729e9d7c21a832fd3c349653b98e79cfc40250da7
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
  - Seurat
  - SingleCellGGM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# clustering-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Davies-Bouldin Index (DBI) and Calinski-Harabasz (CH) scores to quantitatively compare how well different cell embeddings (e.g., GEP-averaged UMAP vs. PCA, scDHA, scMGCA) separate known cell types, as done in the SingleCellGGM/CellGEP benchmark on the MCA dataset.

## When to use

Use when comparing two or more cell-by-embedding matrices of the same single-cell dataset against identical cell-type labels — e.g., benchmarking a 137-GEP dimension reduction against PCA, scDHA, or scMGCA embeddings.

## When NOT to use

- No ground-truth cell-type labels exist for the cells — DBI and CH both require pre-assigned labels to score cluster/embedding quality
- The compared embeddings were built on different subsets of cells or evaluated against different label sets — scores are then not comparable
- The goal is a clustering algorithm itself rather than scoring separability of an existing embedding against known labels

## Inputs

- Cell embedding matrices for all compared methods (GEP-averaged, PCA, scDHA, scMGCA) on the same cells
- Original MCA cell-type label vector for every cell
- GEP marker table 'data/mca.marker.geps.txt' (gene name, GEP name; 137 GEPs)
- MCA gene count matrix (cells x genes) used to build the embeddings

## Outputs

- Per-method Davies-Bouldin Index and Calinski-Harabasz score pairs
- Comparison table saved as embedding_comparison_metrics.csv

## How to apply

Build each candidate embedding on the same gene count matrix and the same set of cells: for the GEP-based embedding, use CellGEP's average_by_gep with gep_origin='mca' and the 137 marker GEPs from data/mca.marker.geps.txt to get a cell-by-GEP matrix, then scale it and compute UMAP with Seurat; generate PCA, scDHA, and scMGCA embeddings of the same data in parallel. For every embedding, compute the Davies-Bouldin Index and Calinski-Harabasz score using the identical original MCA cell-type labels so scores are directly comparable (lower DBI = better separation, higher CH = better separation). Assemble all four DBI/CH pairs into a comparison table such as embedding_comparison_metrics.csv. Correct application means the metric pair is computed on identically scaled, identically labeled data across methods, so the only differing variable is the embedding method itself.

## Related tools

- **CellGEP** (Provides average_by_gep to compute the cell-by-GEP matrix (137 MCA marker GEPs) whose UMAP embedding is one of the methods being scored) — https://github.com/MaShisongLab/cellgep
- **Seurat** (Scales the GEP-averaged expression matrix and computes the UMAP embedding that is then evaluated with DBI/CH metrics) — https://satijalab.org/seurat/
- **SingleCellGGM** (Origin of the 137 marker GEPs (identified from the MCA dataset via single-cell GCN analysis) used for the GEP-averaged embedding under comparison) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
geps_expression = average_by_gep(expression, geps, 'mca', 'novel_sc_dataset'); from sklearn.metrics import davies_bouldin_score, calinski_harabasz_score; dbi = davies_bouldin_score(umap_embedding, mca_labels); ch = calinski_harabasz_score(umap_embedding, mca_labels)
```

## Evaluation signals

- The GEP-based embedding reproduces the paper's favorable performance: lower Davies-Bouldin Index than the PCA, scDHA, and scMGCA embeddings
- The GEP-based embedding achieves a higher Calinski-Harabasz score than all three competing embeddings
- All four DBI/CH score pairs were computed against the identical MCA cell-type label vector on the same cells
- The output embedding_comparison_metrics.csv contains exactly four method rows (GEP, PCA, scDHA, scMGCA), each with one DBI and one CH value

## Limitations

- Both DBI and CH are label-dependent internal metrics; they can only be applied when authoritative cell-type annotations (e.g., MCA labels) are available
- The favorable benchmark result was shown for the 137 MCA marker GEPs; results may differ for GEPs identified from other reference datasets
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities, so embeddings are transcriptome-derived only

## Evidence

- [other] DBI/CH scoring of GEP-averaged embedding: [evidence span withheld — anchor status: not_found]
- [other] Benchmark rationale and expected outcome: [evidence span withheld — anchor status: not_found]
- [abstract] GEP dimension reduction motivation: "We also proposed a dimension-reduction method through averaging by GEPs for single-cell analysis, enhancing the interpretability of results."
- [readme] average_by_gep produces the cell-by-GEP matrix: "The function `average_by_gep`calculates the marker GEP's average expression values in the novel dataset."
