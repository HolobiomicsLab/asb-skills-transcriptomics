---
name: embedding-benchmark-comparison-across-methods
description: Use when comparing a novel single-cell dimension-reduction embedding
  (e.g., GEP-averaged expression) against established methods like PCA, scDHA, or
  scMGCA on the same labeled dataset with identical clustering-quality metrics.
when_to_use_negative:
- The dataset lacks ground-truth cell-type labels, since DBI and CH scores must be
  computed against the original MCA cell-type labels
- The candidate embedding is not computed on the same cells and labels as the baseline
  embeddings, making the metric comparison invalid
- The goal is gene-level network inference rather than evaluating cell-embedding quality
  (use SingleCellGGM network construction instead)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CellGEP
  role: Provides average_by_gep to compute per-cell average expression over the 137
    MCA marker GEPs, producing the GEP-based embedding
  repo: github.com/MaShisongLab/CellGEP
- name: Seurat
  role: Scales the GEP-averaged matrix and computes the UMAP embeddings that are scored
    with DBI and CH metrics
  repo: https://satijalab.org/seurat/
- name: SingleCellGGM
  role: Upstream method that identified the marker GEPs used for the GEP-based dimension
    reduction being benchmarked
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:90756bf4d4d9d860077388de64e11a515a0c20c3aef60284a843059a92dd4886
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# embedding-benchmark-comparison-across-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a cell embedding produced by GEP-based dimension reduction (averaging expression over the 137 MCA marker GEPs) against PCA, scDHA, and scMGCA embeddings using Davies-Bouldin Index and Calinski-Harabasz scores computed against the original MCA cell-type labels.

## When to use

Use when comparing a novel single-cell dimension-reduction embedding (e.g., GEP-averaged expression) against established methods like PCA, scDHA, or scMGCA on the same labeled dataset with identical clustering-quality metrics.

## When NOT to use

- The dataset lacks ground-truth cell-type labels, since DBI and CH scores must be computed against the original MCA cell-type labels
- The candidate embedding is not computed on the same cells and labels as the baseline embeddings, making the metric comparison invalid
- The goal is gene-level network inference rather than evaluating cell-embedding quality (use SingleCellGGM network construction instead)

## Inputs

- MCA gene count matrix (cells x genes)
- 137 marker GEPs from 'data/mca.marker.geps.txt'
- Original MCA cell-type labels
- PCA, scDHA, and scMGCA embeddings of the same MCA dataset

## Outputs

- Scaled GEP-averaged (cell x 137 GEP) expression matrix
- UMAP embeddings for each of the four methods
- Davies-Bouldin Index and Calinski-Harabasz score per method
- Comparison table saved as embedding_comparison_metrics.csv

## How to apply

Compute a cell-by-GEP matrix by running CellGEP's average_by_gep (gep_origin='mca') on the MCA gene count matrix with the 137 marker GEPs from 'data/mca.marker.geps.txt'. Scale the GEP-averaged matrix and compute UMAP embeddings in Seurat, then calculate the Davies-Bouldin Index and Calinski-Harabasz score of the embedding against the original MCA cell-type labels. Repeat the identical scaling/UMAP/metric steps for PCA, scDHA, and scMGCA embeddings of the same dataset so all four embeddings are evaluated with identical metrics and identical labels. Assemble the four DBI/CH score pairs into a comparison table (e.g., embedding_comparison_metrics.csv). The method is validated if the GEP-based embedding achieves a lower DBI and higher CH score than the three competing embeddings, matching the paper's reported favorable performance.

## Related tools

- **CellGEP** (Provides average_by_gep to compute per-cell average expression over the 137 MCA marker GEPs, producing the GEP-based embedding) — github.com/MaShisongLab/CellGEP
- **Seurat** (Scales the GEP-averaged matrix and computes the UMAP embeddings that are scored with DBI and CH metrics) — https://satijalab.org/seurat/
- **SingleCellGGM** (Upstream method that identified the marker GEPs used for the GEP-based dimension reduction being benchmarked) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
geps = read.table('data/mca.marker.geps.txt'); geps_expression = average_by_gep(expression, geps, 'mca', 'mca')
```

## Evaluation signals

- The GEP-based UMAP embedding has a lower Davies-Bouldin Index than the PCA, scDHA, and scMGCA embeddings
- The GEP-based UMAP embedding has a higher Calinski-Harabasz score than all three competing embeddings
- All four embeddings are evaluated with identical metrics (DBI and CH) and identical MCA cell-type labels
- The GEP-averaged matrix has exactly 137 columns, one per marker GEP
- The four DBI/CH score pairs assemble cleanly into embedding_comparison_metrics.csv

## Limitations

- Benchmark validity depends on the quality of the original MCA cell-type labels used for scoring
- The GEP set is limited to the 137 marker GEPs identified from MCA via SingleCellGGM; datasets with cell types absent from MCA are not covered (analogous to label transfer yielding 'unknown' annotations)
- SingleCellGGM relies exclusively on transcriptome data, so the GEP-based embedding cannot incorporate other modalities like ATAC-seq or single-cell proteomics

## Evidence
