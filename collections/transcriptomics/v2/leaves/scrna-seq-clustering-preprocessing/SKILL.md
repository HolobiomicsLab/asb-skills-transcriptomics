---
name: scrna-seq-clustering-preprocessing
description: Use when raw or unclustered scRNA-seq data (e.g., 10X Genomics counts
  from mouse skin wound samples GSM3453535-GSM3453538) must be QC-filtered and assigned
  to cell groups before ligand-receptor communication inference.
when_to_use_negative:
- Input is already a clustered CellChat object or Seurat object with defined cell
  identities — clustering is complete and re-running QC could discard legitimate cell
  groups
- Data are spatial transcriptomics or pseudotime/continuous-state trajectories where
  discrete Louvain clustering is not the appropriate representation for communication
  inference
- Only a benchmark of inference tools on existing labels is needed — this skill covers
  preprocessing, not the communication inference itself
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: Downstream consumer of the clustered data; createCellChat(object, meta, group.by)
    requires pre-defined cell groups from the clustering step
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Downstream tool run on the same clustered datasets (v2.0) for cross-tool benchmarking
- name: SingleCellSignalR
  role: Downstream tool run on the same clustered datasets for false-positive-rate
    benchmarking (10.6% multi-subunit FPR target)
- name: iTALK
  role: Downstream tool run on the same clustered datasets for false-positive-rate
    benchmarking (14.3% multi-subunit FPR target)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:c6c275749c949a5806324c05635294a6f7d6cdbe672df2cc8dc34fc08f6608af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellPhoneDB
  - SingleCellSignalR
  - iTALK
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrna-seq-clustering-preprocessing

## Summary

Quality control, filtering, and cell clustering of scRNA-seq datasets so that discrete cell groups exist as a prerequisite for cell-cell communication inference with CellChat, CellPhoneDB, iTALK and SingleCellSignalR.

## When to use

Use when raw or unclustered scRNA-seq data (e.g., 10X Genomics counts from mouse skin wound samples GSM3453535-GSM3453538) must be QC-filtered and assigned to cell groups before ligand-receptor communication inference.

## When NOT to use

- Input is already a clustered CellChat object or Seurat object with defined cell identities — clustering is complete and re-running QC could discard legitimate cell groups
- Data are spatial transcriptomics or pseudotime/continuous-state trajectories where discrete Louvain clustering is not the appropriate representation for communication inference
- Only a benchmark of inference tools on existing labels is needed — this skill covers preprocessing, not the communication inference itself

## Inputs

- Raw 10X Genomics scRNA-seq count matrices from GEO (e.g., GSM3453535, GSM3453536, GSM3453537, GSM3453538)
- Per-cell QC metrics (UMI counts, gene counts, mitochondrial fraction)
- Cell-level metadata for group labeling

## Outputs

- QC-filtered expression matrix (e.g., 21,557 retained cells from 21,819)
- Cell cluster/group labels suitable for createCellChat(object, meta, group.by)
- Benchmark-ready clustered datasets for 90/80/70% geometric-sketching subsamples

## How to apply

Download the raw 10X datasets (e.g., GEO accessions GSM3453535, GSM3453536, GSM3453537, GSM3453538) and first apply dataset-specific QC filters — for embryonic skin, remove cells with UMI counts below 2500 or above 50000, fewer than 1000 genes, or mitochondrial fraction above 20%; for the mouse skin wound data, exclude red blood cells (retaining 21,557 of 21,819 cells). Remove contaminated cell groups identified during subclustering (e.g., CD3D-expressing T cells contaminating FIB subclustering, or immune cells lacking DC markers). Cluster cells (Louvain community detection) and only subcluster cell types whose abundance exceeds 5% of the dataset. The rationale is that cell clustering is a strict prerequisite for communication analysis with CellChat, CellPhoneDB, iTALK and SingleCellSignalR, so cluster quality directly determines downstream interaction inference; benchmark comparisons such as subsampling at 90/80/70% should be run on identically preprocessed data.

## Related tools

- **CellChat** (Downstream consumer of the clustered data; createCellChat(object, meta, group.by) requires pre-defined cell groups from the clustering step) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Downstream tool run on the same clustered datasets (v2.0) for cross-tool benchmarking)
- **SingleCellSignalR** (Downstream tool run on the same clustered datasets for false-positive-rate benchmarking (10.6% multi-subunit FPR target))
- **iTALK** (Downstream tool run on the same clustered datasets for false-positive-rate benchmarking (14.3% multi-subunit FPR target))

## Examples

```
cellchat <- createCellChat(object = data, meta = meta, group.by = "labels")
```

## Evaluation signals

- Cell counts match expected filtering outcomes: e.g., mouse skin wound data retains 21,557 of 21,819 cells after red blood cell exclusion
- QC thresholds applied as specified: UMI 2500–50000, genes >= 1000, mitochondrial fraction < 20% (embryonic skin)
- Subclustering restricted to cell types with abundance > 5% of the dataset
- Identical clustered inputs yield the reported cross-tool agreement: ~50% Jaccard overlap between CellChat and CellPhoneDB predictions
- Subsampled (90/80/70% geometric sketching) re-clusters remain stable enough that CellChat TPR/FPR/ACC can be computed against full-data predictions

## Limitations

- Cell clustering is a pre-requisite for these communication tools, so clustering errors propagate directly into inferred interactions
- Contaminated cell groups (e.g., CD3D-expressing T cells in FIB subclusters, immune cells lacking DC markers) must be manually identified and removed; automated QC alone will not catch them
- Red blood cells must be explicitly excluded; dataset-specific QC criteria differ (wound data vs. embryonic skin thresholds)

## Evidence

- [discussion] Cell clustering prerequisite: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [discussion] Red blood cell removal: "we excluded red blood cells and used the remaining 21,557 cells."
- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000"
- [discussion] Subclustering abundance filter: "we performed subclustering analysis on the cell types, whose abundance in the dataset was greater than 5% using the Louvain community detection method."
- [discussion] Contaminated group removal: "This contaminated TC population was removed for further analysis."
