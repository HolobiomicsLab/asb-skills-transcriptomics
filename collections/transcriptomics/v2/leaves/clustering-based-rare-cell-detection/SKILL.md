---
name: clustering-based-rare-cell-detection
description: Use when the input is a raw scRNA-seq count matrix (genes in columns,
  cells in rows) and the goal is to detect rare cell populations (e.g., <1% of cells)
  that standard one-time clustering pipelines like Seurat tend to miss.
when_to_use_negative:
- The goal is comprehensive major cell type annotation rather than rare population
  detection — the paper positions tools like Seurat for comprehensive major-type identification,
  with scCAD targeting rare types overlooked during the clustering phase
- The input is already normalized, feature-selected, or a precomputed embedding/feature
  table rather than a raw count matrix (the scCAD call applies its own normalization
  unless normalization=False)
- The task is comparing multiple rare-cell methods across datasets rather than running
  one method — that benchmarking orchestration (running CellSIUS, EDGE, FiRE, SCA,
  etc. with the paper's settings) is a separate workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: 'Core method: ensemble feature selection, iterative cluster decomposition,
    isolation-forest anomaly scoring, and independence-score rare-type calling'
  repo: https://github.com/xuyp-csu/scCAD
- name: Seurat
  role: Preprocessing/clustering baseline used to define major cell types for CellSIUS
    and EDGE comparisons
- name: CellSIUS
  role: Comparator method run on major types from Seurat clustering; third-ranked
    F1 (0.2812) on the 25-dataset benchmark
  repo: https://github.com/Novartis/CellSIUS
- name: EDGE
  role: Comparator method run on Seurat-preprocessed matrices; successful rare-type
    calls in 16 of 25 datasets
  repo: https://github.com/shawnstat/EDGE
- name: FiRE
  role: Comparator method with IQR-based thresholding on the benchmark
  repo: https://github.com/princethewinner/FiRE
- name: GapClust
  role: Comparator method; successful rare-type calls in 12 of 25 datasets
  repo: https://github.com/fabotao/GapClust
- name: GiniClust
  role: Comparator method (GiniClust2/3) scored as a global clustering with rare calls
    for clusters below the size threshold
  repo: https://github.com/dtsoucas/GiniClust2
- name: RaceID
  role: Comparator method scored as a global clustering with rare calls for clusters
    below the size threshold
  repo: https://github.com/dgrun/RaceID3_StemID2_package
- name: SCISSORS
  role: Comparator method scored as a global clustering with rare calls for clusters
    below the size threshold
- name: CIARA
  role: Comparator method scored as a global clustering with rare calls for clusters
    below the size threshold
  repo: https://github.com/ScialdoneLab/CIARA
- name: SCA
  role: Surprisal component analysis comparator (via shannonca); second-ranked F1
    (0.3359) on the 25-dataset benchmark
  repo: https://github.com/bendemeo/shannonca
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:8808a06745a231c7d97c4977191ff1450229281110d3c24b0a22892706d904d1
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
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# clustering-based-rare-cell-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify rare cell types in scRNA-seq data by iteratively decomposing clusters on their most differential signals, scoring decomposed sub-clusters with an isolation forest, and calling rare types via an independence score threshold (default I=0.7). scCAD achieves the highest overall F1 (0.4172) across 25 benchmark datasets, rare types in 20 of them.

## When to use

Use when the input is a raw scRNA-seq count matrix (genes in columns, cells in rows) and the goal is to detect rare cell populations (e.g., <1% of cells) that standard one-time clustering pipelines like Seurat tend to miss.

## When NOT to use

- The goal is comprehensive major cell type annotation rather than rare population detection — the paper positions tools like Seurat for comprehensive major-type identification, with scCAD targeting rare types overlooked during the clustering phase
- The input is already normalized, feature-selected, or a precomputed embedding/feature table rather than a raw count matrix (the scCAD call applies its own normalization unless normalization=False)
- The task is comparing multiple rare-cell methods across datasets rather than running one method — that benchmarking orchestration (running CellSIUS, EDGE, FiRE, SCA, etc. with the paper's settings) is a separate workflow

## Inputs

- Raw scRNA-seq count matrix (cells × genes), e.g., an h5 file such as the 1%Jurkat.h5 demo (1556 cells, 32,738 genes)
- Optional cell names, gene names, and a dataset name string
- Parameter overrides: rare_h (R, default 0.01), merge_h (THM, default 0.3), overlap_h (I, default 0.7), random seed (default 2023)

## Outputs

- List of rare sub-clusters (cell name sets) flagged as rare cell types
- Anomaly/independence score per sub-cluster (np.array[n sub-clusters])
- Sub-cluster label assignment for every cell (np.array[n cells])
- List of differentially expressed genes (top 20 per cluster) used to define each rare sub-cluster
- Benchmark-style summary: per-rare-type F1 against original study annotations and count of datasets with rare types recovered above the 30% threshold

## How to apply

Preprocess the count matrix by retaining genes expressed in at least three cells, then log-normalize with cell-specific size factors followed by a log2 transform with a pseudo-count of 1. scCAD then performs ensemble feature selection (top 2000 highly variable genes plus top 2000 random-forest-importance genes), PCA to 40 components with a k=15 nearest-neighbor graph and Louvain clustering, and iteratively decomposes any cluster larger than R=1% of total cells based on the most differential signals. Decomposed sub-clusters are merged when their centers are closer than THM (the median nearest-neighbor center distance), after which an isolation forest is run on the top 20 differential genes per cluster to compute anomaly scores. Clusters with an independence score exceeding I=0.7 (the default) are flagged as rare; on the 25 real scRNA-seq benchmarks this configuration yielded the highest F1 (0.4172 vs SCA 0.3359 and CellSIUS 0.2812) and successful rare-type identification (>30% recovery criterion) in 20 datasets.

## Related tools

- **scCAD** (Core method: ensemble feature selection, iterative cluster decomposition, isolation-forest anomaly scoring, and independence-score rare-type calling) — https://github.com/xuyp-csu/scCAD
- **Seurat** (Preprocessing/clustering baseline used to define major cell types for CellSIUS and EDGE comparisons)
- **CellSIUS** (Comparator method run on major types from Seurat clustering; third-ranked F1 (0.2812) on the 25-dataset benchmark) — https://github.com/Novartis/CellSIUS
- **EDGE** (Comparator method run on Seurat-preprocessed matrices; successful rare-type calls in 16 of 25 datasets) — https://github.com/shawnstat/EDGE
- **FiRE** (Comparator method with IQR-based thresholding on the benchmark) — https://github.com/princethewinner/FiRE
- **GapClust** (Comparator method; successful rare-type calls in 12 of 25 datasets) — https://github.com/fabotao/GapClust
- **GiniClust** (Comparator method (GiniClust2/3) scored as a global clustering with rare calls for clusters below the size threshold) — https://github.com/dtsoucas/GiniClust2
- **RaceID** (Comparator method scored as a global clustering with rare calls for clusters below the size threshold) — https://github.com/dgrun/RaceID3_StemID2_package
- **SCISSORS** (Comparator method scored as a global clustering with rare calls for clusters below the size threshold)
- **CIARA** (Comparator method scored as a global clustering with rare calls for clusters below the size threshold) — https://github.com/ScialdoneLab/CIARA
- **SCA** (Surprisal component analysis comparator (via shannonca); second-ranked F1 (0.3359) on the 25-dataset benchmark) — https://github.com/bendemeo/shannonca

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Reproduced per-rare-type F1 against original study annotations should rank scCAD highest (F1 = 0.4172) above SCA (0.3359) and CellSIUS (0.2812) across the 25 real datasets
- Using the 30% recovery criterion (|S_pre ∩ S_t| / |S_t| > 30%), scCAD should successfully identify rare types in 20 of 25 datasets, ahead of SCA (18), CellSIUS (16), EDGE (16), and GapClust (12)
- On the demo 1%Jurkat.h5 dataset (1540 293T cells, 16 Jurkat cells), the returned rare sub-clusters should be dominated by Jurkat cells when cross-tabulated against the ground-truth labels, and execution should take roughly 40 seconds
- Decomposition should only fire on clusters exceeding R=1% of total cells, and a cluster is called rare only when its independence score exceeds I=0.7
- Over 86% of the genes scCAD selects should be rare cell type-specific on average, providing a sanity check on the ensemble feature selection

## Limitations

- Exact version numbers for most comparator packages (CellSIUS, CIARA, EDGE, FiRE, GapClust, GiniClust2/3, RaceID3, SCISSORS) are not stated; only 'latest version' for SCA
- Source data availability/accession IDs for the 25 benchmark datasets are not provided in the section, complicating exact reproduction of the benchmark table
- No statistical method was used to pre-determine sample size and the authors note absence of randomization for most experiments
- Existing rare cell identification approaches (and by extension any such method, including scCAD) are sensitive to clusters formed by one-time clustering using partial or global gene expression, where rare types may be overlooked in the clustering phase

## Evidence

- [discussion] Decomposition applies to clusters above 1% of cells: [evidence span withheld — anchor status: not_found]
- [discussion] Independence score threshold I defaults to 0.7: [evidence span withheld — anchor status: not_found]
- [discussion] Isolation forest on top 20 differential genes per cluster: [evidence span withheld — anchor status: not_found]
