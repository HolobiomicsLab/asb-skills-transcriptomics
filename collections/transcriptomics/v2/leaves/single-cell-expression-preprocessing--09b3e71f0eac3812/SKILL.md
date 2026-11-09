---
name: single-cell-expression-preprocessing
description: Use when a raw cell-by-gene scRNA-seq count matrix (e.g., .h5 or similar
  with X, Y, gene and cell names) must be filtered, normalized, and reduced to PCA/KNN/Louvain
  I-clusters before rare cluster detection or anomaly scoring.
when_to_use_negative:
- The expression matrix is already log-normalized — in that case pass normalization=False
  to scCAD to avoid double-normalizing
- The task is downstream anomaly scoring of precomputed sub-clusters (isolation forest
  on top 20 differential genes, independence score threshold I=0.7) rather than preprocessing
- The input is not a cell-by-gene count matrix (e.g., a feature table or metadata
  sheet), which cannot be gene-filtered or PCA-embedded by this recipe
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAD
  role: Implements the preprocessing recipe (gene filtering, log-normalization with
    cell-specific size factors, 40-PC PCA, k=15 KNN graph, Louvain I-clustering) inside
    its main function with normalization=True and seed=2023
  repo: https://github.com/xuyp-csu/scCAD
- name: Scanpy
  role: Dependency (version 1.9.1) used in the scCAD environment for single-cell preprocessing
    operations
- name: python-louvain
  role: Dependency (version 0.16) providing Louvain community detection for the initial
    I-clustering on the 15-NN graph
- name: h5py
  role: Dependency (version 3.7.0) used to load the 1%Jurkat.h5 example matrix ('X',
    'Y', 'gn', 'cn' arrays) into memory
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:613f41fc87f5627098183f7450121f5609ad6967fcb40cb76986d1d9580ce5a3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - Scanpy
  - h5py
  - python-louvain
  - scCAD
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-expression-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw scRNA-seq count matrices (e.g., the 1%Jurkat.h5 example with 1540 293T + 16 Jurkat cells) by retaining genes expressed in at least three cells, log-normalizing with cell-specific size factors plus log2(x+1) transformation, and embedding via 40-PC PCA with a 15-nearest-neighbor Euclidean graph and Louvain clustering to produce initial I-clusters. This is the deterministic front-end that scCAD's rare cell detection depends on.

## When to use

Use when a raw cell-by-gene scRNA-seq count matrix (e.g., .h5 or similar with X, Y, gene and cell names) must be filtered, normalized, and reduced to PCA/KNN/Louvain I-clusters before rare cluster detection or anomaly scoring.

## When NOT to use

- The expression matrix is already log-normalized — in that case pass normalization=False to scCAD to avoid double-normalizing
- The task is downstream anomaly scoring of precomputed sub-clusters (isolation forest on top 20 differential genes, independence score threshold I=0.7) rather than preprocessing
- The input is not a cell-by-gene count matrix (e.g., a feature table or metadata sheet), which cannot be gene-filtered or PCA-embedded by this recipe

## Inputs

- Raw scRNA-seq expression matrix with cells in rows and genes in columns (e.g., 1%Jurkat.h5 containing 'X', 'Y', 'gn', 'cn' arrays; 1556 cells x 32,738 genes in the example)
- Optional cell name and gene name vectors (byte-encoded strings decoded to UTF-8); generated automatically if omitted

## Outputs

- Normalized, filtered expression matrix (genes expressed in <3 cells removed; log2(size-factor-normalized + 1) values)
- Initial Louvain I-cluster labels per cell (top 40 PCs, 15-NN Euclidean graph) feeding cluster decomposition
- scCAD return values: sub-cluster label assignment per cell (np.array[n cells]), rare sub-cluster list, per-sub-cluster scores, and DEG lists

## How to apply

Load the expression matrix with cells in rows and genes in columns (e.g., via h5py from 1%Jurkat.h5, coercing to float and decoding byte-encoded gene/cell names as UTF-8). Filter genes expressed in at least three cells, then apply log-normalization with cell-specific size factors followed by log2(x+1) transformation (pseudo-count of 1). Compute the top 40 principal components, build a 15-nearest-neighbor Euclidean graph, and run Louvain clustering (python-louvain) to obtain the initial I-clusters. scCAD exposes this via its main function with normalization=True (default) and seed=2023 for reproducibility; the subsequent rapid clustering module reuses the same PCA 40 PCs / KNN k=15 / Louvain recipe when decomposing large clusters. Rationale: retaining only genes expressed in ≥3 cells reduces noise, and the fixed PCA/KNN/Louvain recipe gives a stable I-clustering whose decomposition yields D-clusters and M-clusters.

## Related tools

- **scCAD** (Implements the preprocessing recipe (gene filtering, log-normalization with cell-specific size factors, 40-PC PCA, k=15 KNN graph, Louvain I-clustering) inside its main function with normalization=True and seed=2023) — https://github.com/xuyp-csu/scCAD
- **Scanpy** (Dependency (version 1.9.1) used in the scCAD environment for single-cell preprocessing operations)
- **python-louvain** (Dependency (version 0.16) providing Louvain community detection for the initial I-clustering on the 15-NN graph)
- **h5py** (Dependency (version 3.7.0) used to load the 1%Jurkat.h5 example matrix ('X', 'Y', 'gn', 'cn' arrays) into memory)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The filtered/normalized matrix retains only genes expressed in at least three cells and contains log2(x+1)-transformed values
- PCA yields exactly the top 40 principal components and the neighbor graph uses k=15 under Euclidean metric before Louvain
- Per-cell output (sub_clusters) has length equal to the input cell count (1556 rows for the 1%Jurkat example)
- The decomposition isolates at least one small (sub-1%) M-cluster corresponding to the 16 Jurkat cells against 1540 293T cells
- Re-running with the default seed=2023 reproduces the same cluster labels

## Limitations

- The gene filter cutoff (expressed in at least three cells) and thresholds (R=0.01, or 30/n when the dataset has fewer than 3000 cells; THM median nearest-neighbor center distance) were tuned on the benchmarked datasets and may need adjustment for very small or very large datasets
- RAM constraints: scCAD requires only a standard computer but assumes the full expression matrix fits in memory (in-memory operations)
- Package versions are pinned in requirements.txt (e.g., Python 3.7.13, Scanpy 1.9.1, numpy 1.21.5); other versions are untested
- Supported OS are Linux and macOS only (tested on Ubuntu 16.04.7 and macOS Monterey 12.6.3)

## Evidence

- [discussion] PCA/KNN/Louvain parameters: [evidence span withheld — anchor status: not_found]
