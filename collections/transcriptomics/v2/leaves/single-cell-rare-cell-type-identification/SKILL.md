---
name: single-cell-rare-cell-type-identification
description: Use when scRNA-seq count or log-normalized expression matrices (cells
  x genes, e.g. .h5 with X/Y/gn/cn fields) need rare sub-populations below ~1% detected
  beyond global clustering.
when_to_use_negative:
- The goal is comprehensive identification of major cell types (e.g. standard Seurat
  Louvain clustering), not specialized rare-type detection
- The input is already sub-clustered and rare populations are larger than ~1% and
  recoverable by conventional clustering, so decomposition adds no signal
- The data is not scRNA-seq expression (e.g. spatial coordinates, bulk RNA-seq), since
  thresholds like rare_h=0.01 and the isolation-forest scoring assume single-cell
  expression distributions
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scCAD
  role: 'Core algorithm: ensemble feature selection, iterative cluster decomposition,
    isolation-forest anomaly scoring and rare sub-cluster calling'
  repo: https://github.com/xuyp-csu/scCAD
- name: h5py
  role: Reads the example 1%Jurkat.h5 dataset (X matrix, Y labels, gene names gn,
    cell names cn)
- name: numpy
  role: Parses matrix and name arrays, converts values to float and UTF-8 strings,
    indexes sub-cluster membership
- name: pandas
  role: Assembles rare sub-cluster membership, scores, per-cell labels and DE gene
    lists into result tables (e.g. jurkat_sccad_results.csv)
- name: Seurat
  role: Provides major-type clustering and preprocessed matrices used by comparators
    such as CellSIUS and EDGE in the benchmark
- name: SCA
  role: Surprisal component analysis comparator run via a 15-nearest-neighbor Euclidean
    graph in 50 dimensions with Leiden clustering at resolution 1.0
  repo: https://github.com/bendemeo/shannonca
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:78db7a15d73763ddb2075765fbe08260ca6769b3eae99d3341f248cc8418f006
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
  - CIARA
  - CellSIUS
  - EDGE
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - SCA
  - SCISSORS
  - Seurat
  - h5py
  - numpy
  - pandas
  - scCAD
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# single-cell-rare-cell-type-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify rare cell types in scRNA-seq data using scCAD, which iteratively decomposes clusters on the most differential signals, scores sub-clusters with an isolation forest, and flags rare types with an independence score above I=0.7.

## When to use

Use when scRNA-seq count or log-normalized expression matrices (cells x genes, e.g. .h5 with X/Y/gn/cn fields) need rare sub-populations below ~1% detected beyond global clustering.

## When NOT to use

- The goal is comprehensive identification of major cell types (e.g. standard Seurat Louvain clustering), not specialized rare-type detection
- The input is already sub-clustered and rare populations are larger than ~1% and recoverable by conventional clustering, so decomposition adds no signal
- The data is not scRNA-seq expression (e.g. spatial coordinates, bulk RNA-seq), since thresholds like rare_h=0.01 and the isolation-forest scoring assume single-cell expression distributions

## Inputs

- scRNA-seq expression matrix with cells in rows and genes in columns (raw counts or log-normalized; .h5 file with X, Y, gn, cn fields like 1%Jurkat.h5)
- Optional gene names and cell names arrays (UTF-8 strings; auto-generated if not provided)
- Optional ground-truth cell-type labels (Y field) for validating rare sub-cluster enrichment

## Outputs

- result: list of rare sub-clusters identified by scCAD
- score: np.array of independence scores per sub-cluster
- sub_clusters: np.array of per-cell sub-cluster assignments
- degs_list: list of candidate differentially expressed genes per rare sub-cluster
- Saved results directory (save_path) including full results when save_full=True

## How to apply

Load the expression matrix (cells x genes), gene and cell names, e.g. from 1%Jurkat.h5 via h5py, then call scCAD.scCAD with defaults normalization=True, seed=2023, rare_h=0.01, merge_h=0.3, overlap_h=0.7. scCAD retains genes expressed in at least three cells, log-normalizes with cell-specific size factors plus log2(pseudo-count 1), selects the top 2000 HVGs plus top 2000 random-forest-importance genes, runs PCA (top 40 PCs) with k=15 KNN and Louvain clustering, and iteratively decomposes clusters larger than R=1% of cells. Decomposed clusters are merged at THM=median nearest-neighbor center distance, then an isolation forest on each cluster's top 20 differential genes yields independence scores; clusters scoring above I=0.7 are called rare. Unpack the four returns — result, score, sub_clusters, degs_list — and validate that rare sub-clusters are enriched for known rare cells (e.g. the 16 Jurkat cells among 1540 293T cells).

## Related tools

- **scCAD** (Core algorithm: ensemble feature selection, iterative cluster decomposition, isolation-forest anomaly scoring and rare sub-cluster calling) — https://github.com/xuyp-csu/scCAD
- **h5py** (Reads the example 1%Jurkat.h5 dataset (X matrix, Y labels, gene names gn, cell names cn))
- **numpy** (Parses matrix and name arrays, converts values to float and UTF-8 strings, indexes sub-cluster membership)
- **pandas** (Assembles rare sub-cluster membership, scores, per-cell labels and DE gene lists into result tables (e.g. jurkat_sccad_results.csv))
- **Seurat** (Provides major-type clustering and preprocessed matrices used by comparators such as CellSIUS and EDGE in the benchmark)
- **SCA** (Surprisal component analysis comparator run via a 15-nearest-neighbor Euclidean graph in 50 dimensions with Leiden clustering at resolution 1.0) — https://github.com/bendemeo/shannonca

## Examples

```
import h5py, numpy as np; data = np.array(h5py.File('./1%Jurkat.h5')['X']); result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', save_path='./')
```

## Evaluation signals

- Run returns exactly four outputs (result, score, sub_clusters, degs_list); a run completing in ~40 seconds on 1%Jurkat.h5 is expected
- Identified rare sub-clusters are enriched for the 16 Jurkat cells among the 1540 293T cells in the demo dataset
- Independence scores of called rare sub-clusters exceed the default threshold I=0.7
- On the 25-dataset benchmark, expect highest F1 (reported 0.4172 vs SCA 0.3359, CellSIUS 0.2812) and rare-type identification success in 20 of 25 datasets under the 30% recovery criterion
- Rare cell type-specific genes should account for over 86% of selected genes on average

## Limitations

- Requires in-memory operations — datasets must fit in RAM on a standard computer; tested only on Linux (Ubuntu 16.04.7) and macOS (Monterey 12.6.3) with Python 3.7.13
- Rare sub-cluster identification is a random/seeded process (seed=2023 default); merge behavior depends on THM (median nearest-neighbor center distance) which can be sensitive to dataset scale
- Genes expressed in fewer than three cells are dropped, and clusters smaller than R=1% of cells are not decomposed, so ultra-rare populations below these cutoffs may be missed
- Demo run takes ~40 seconds even on a 1556-cell dataset; runtime scales with dataset size (runtime estimation was relegated to Supplementary Note 5 / Supplementary Table 21, not the main text)

## Evidence

- [discussion] Default independence score threshold I=0.7: [evidence span withheld — anchor status: not_found]
- [discussion] Isolation forest on top 20 differential genes: [evidence span withheld — anchor status: not_found]
- [discussion] Cluster decomposition trigger threshold R=1%: [evidence span withheld — anchor status: not_found]
