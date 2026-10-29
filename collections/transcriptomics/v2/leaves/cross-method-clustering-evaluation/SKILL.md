---
name: cross-method-clustering-evaluation
description: Use when you must rank several rare-cell-type or clustering methods over
  a common set of annotated scRNA-seq datasets and report F1 plus rare-type recovery
  counts in a single benchmark table.
when_to_use_negative:
- The datasets lack ground-truth rare cell type annotations, since F1 and the 30%
  recovery criterion are computed against original study annotations
- You need to identify rare cells in a single dataset rather than comparatively benchmark
  multiple methods
- Input is a feature/significance table rather than raw cell-level expression matrices
  that can be passed through the shared preprocessing pipeline
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Method under test; iteratively decomposes clusters (R=1%, THM merge, isolation
    forest on top 20 differential genes, I=0.7) to produce predicted rare cell sets
  repo: github.com/xuyp-csu/scCAD
- name: CellSIUS
  role: Comparator method run on major cell types obtained from Seurat clustering;
    global clustering scored rare below size threshold
  repo: github.com/Novartis/CellSIUS
- name: SCA
  role: Comparator method (shannonca); 15-nearest-neighbor Euclidean graph in 50 dimensions
    with Leiden clustering at resolution 1.0; ranked second with F1 0.3359
  repo: github.com/bendemeo/shannonca
- name: EDGE
  role: Comparator method run on Seurat-preprocessed matrices; tied with CellSIUS
    at 16 successful datasets
  repo: github.com/shawnstat/EDGE
- name: FiRE
  role: Comparator method applied with IQR-based thresholding
  repo: github.com/princethewinner/FiRE
- name: GapClust
  role: Comparator method; 12 datasets with successful rare-type identification
  repo: github.com/fabotao/GapClust
- name: GiniClust2
  role: Comparator method scored as a global clustering
  repo: github.com/dtsoucas/GiniClust2
- name: GiniClust3
  role: Comparator method scored as a global clustering
  repo: github.com/rdong08/GiniClust3
- name: RaceID
  role: Comparator method (RaceID3) scored as a global clustering
  repo: github.com/dgrun/RaceID3_StemID2_package
- name: SCISSORS
  role: Comparator method scored as a global clustering
- name: CIARA
  role: Comparator method scored as a global clustering
  repo: github.com/ScialdoneLab/CIARA
- name: Seurat
  role: Provides major-type clustering and preprocessing matrices used to seed CellSIUS
    and EDGE comparisons
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:2093d15d7466ee8ac0729cd447b7f724873188e1093ce736996b0cff936146c7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - CIARA
  - CellSIUS
  - EDGE
  - FiRE
  - GapClust
  - GiniClust
  - GiniClust2
  - GiniClust3
  - RaceID
  - SCA
  - SCISSORS
  - Seurat
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# cross-method-clustering-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks multiple rare-cell-type identification/clustering methods (e.g., scCAD vs. CellSIUS, EDGE, FiRE, SCA, GapClust, GiniClust2/3, RaceID, SCISSORS, CIARA) on a common panel of scRNA-seq datasets and compiles a ranked F1-score table with rare-type recovery counts.

## When to use

Use when you must rank several rare-cell-type or clustering methods over a common set of annotated scRNA-seq datasets and report F1 plus rare-type recovery counts in a single benchmark table.

## When NOT to use

- The datasets lack ground-truth rare cell type annotations, since F1 and the 30% recovery criterion are computed against original study annotations
- You need to identify rare cells in a single dataset rather than comparatively benchmark multiple methods
- Input is a feature/significance table rather than raw cell-level expression matrices that can be passed through the shared preprocessing pipeline

## Inputs

- 25 annotated real scRNA-seq benchmark datasets (e.g., from Supplementary Table 1) with original rare-cell-type annotations
- Predicted rare cell sets from each comparator method (CellSIUS, EDGE, FiRE, GapClust, GiniClust2/3, RaceID, SCISSORS, CIARA, SCA, scCAD)
- Gene expression matrices (cells × genes, e.g., .h5 files like the 1%Jurkat.h5 demo) for preprocessing

## Outputs

- benchmark_f1_scores.csv: per-method F1 score and rare-type identification success counts across datasets
- Per-dataset, per-rare-type F1, accuracy, G-mean, Cohen's Kappa, and MCC values
- Ranked method comparison table matching reported values (scCAD 0.4172, SCA 0.3359, CellSIUS 0.2812)

## How to apply

Preprocess every dataset identically before running any method: retain genes expressed in at least three cells, then apply log-normalization with cell-specific size factors and a log2 transform with pseudo-count 1. Run each method with its published settings — e.g., CellSIUS on Seurat major-type clusters, SCA via a 15-nearest-neighbor Euclidean graph in 50 dimensions with Leiden resolution 1.0, and CellSIUS/CIARA/EDGE/GiniClust2/3/SCISSORS/SCA scored as global clusterings where clusters below the 1% (or 5%) size threshold are called rare. Combine each method's predicted rare cell sets per dataset and compute F1 per rare cell type against the original study annotations (plus accuracy, G-mean, Cohen's Kappa, and MCC). Apply the 30% recovery criterion — a method successfully identifies rare type t when |S_pre ∩ S_t| / |S_t| > 30% — to count datasets with at least one successfully identified rare type. Compile F1 scores and success counts into a summary CSV and check the reproduced ranking against the reported one (scCAD F1 = 0.4172, SCA 0.3359, CellSIUS 0.2812; success counts 20, 18, 16, 16, 12).

## Related tools

- **scCAD** (Method under test; iteratively decomposes clusters (R=1%, THM merge, isolation forest on top 20 differential genes, I=0.7) to produce predicted rare cell sets) — github.com/xuyp-csu/scCAD
- **CellSIUS** (Comparator method run on major cell types obtained from Seurat clustering; global clustering scored rare below size threshold) — github.com/Novartis/CellSIUS
- **SCA** (Comparator method (shannonca); 15-nearest-neighbor Euclidean graph in 50 dimensions with Leiden clustering at resolution 1.0; ranked second with F1 0.3359) — github.com/bendemeo/shannonca
- **EDGE** (Comparator method run on Seurat-preprocessed matrices; tied with CellSIUS at 16 successful datasets) — github.com/shawnstat/EDGE
- **FiRE** (Comparator method applied with IQR-based thresholding) — github.com/princethewinner/FiRE
- **GapClust** (Comparator method; 12 datasets with successful rare-type identification) — github.com/fabotao/GapClust
- **GiniClust2** (Comparator method scored as a global clustering) — github.com/dtsoucas/GiniClust2
- **GiniClust3** (Comparator method scored as a global clustering) — github.com/rdong08/GiniClust3
- **RaceID** (Comparator method (RaceID3) scored as a global clustering) — github.com/dgrun/RaceID3_StemID2_package
- **SCISSORS** (Comparator method scored as a global clustering)
- **CIARA** (Comparator method scored as a global clustering) — github.com/ScialdoneLab/CIARA
- **Seurat** (Provides major-type clustering and preprocessing matrices used to seed CellSIUS and EDGE comparisons)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Reproduced F1 table ranks scCAD highest at 0.4172, followed by SCA (0.3359) and CellSIUS (0.2812)
- Success counts match reported values: scCAD 20, SCA 18, CellSIUS 16, EDGE 16, GapClust 12 out of 25 datasets
- Every rare type call uses the 30% recovery criterion: |S_pre ∩ S_t| / |S_t| > 30%
- All methods consume identically preprocessed matrices (genes in ≥3 cells retained; log-normalization with cell-specific size factors and log2 transform, pseudo-count 1)
- Output CSV (benchmark_f1_scores.csv) contains F1 plus accuracy, G-mean, Cohen's Kappa, and MCC per method

## Limitations

- Exact version numbers for most comparator packages (CellSIUS, CIARA, EDGE, FiRE, GapClust, GiniClust2/3, RaceID3, SCISSORS) were not stated; only SCA was pinned to 'latest version' (bendemeo/shannonca)
- Source data accession IDs for the 25 benchmark datasets are not given in the main section (Supplementary Table 1 is required)
- scCAD execution time can be non-trivial (the 1% Jurkat demo takes ~40 seconds), and only rare-cluster assignments, not global annotation, are returned

## Evidence

- [discussion] Rare-call threshold used by the method under test: [evidence span withheld — anchor status: not_found]
