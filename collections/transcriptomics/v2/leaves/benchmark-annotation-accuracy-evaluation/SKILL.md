---
name: benchmark-annotation-accuracy-evaluation
description: Use when a new or existing cluster-level cell type annotation method
  must be quantitatively benchmarked on scRNA-seq datasets whose true cluster labels
  are known from published literature.
when_to_use_negative:
- The datasets have no literature-validated ground-truth cell type labels for clusters,
  so no consistent rate can be computed
- The goal is per-cell annotation rather than cluster-level annotation; scCATCH annotates
  clusters, not individual cells
- The tissue or species of the input data is absent from the CellMatch reference database,
  since performance majorly depends on the reference database and limited references
  may cause incorrect annotation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Automatic cluster-level cell type annotation via findmarkergene() and findcelltype()
    against CellMatch; the method being benchmarked
  repo: https://github.com/ZJUFanLab/scCATCH
- name: cellmatch
  role: Tissue-specific cell taxonomy reference database used to validate marker genes
    and derive candidate cell types for scoring
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
- name: Seurat
  role: Popular marker gene identification method against which scCATCH's benchmark
    performance was compared
- name: SingleR
  role: Cell-based annotation method used in the performance comparison; benchmarked
    with and without CellMatch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:dc6cd669e48fe6f67774ed641e2a3bcd8015fb8e3d33b77eb5bf9db50a3e62c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - Seurat
  - SingleR
  - cellmatch
  - scCATCH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# benchmark-annotation-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates the accuracy of automatic cell type annotation (e.g., by scCATCH against its CellMatch database) on literature-validated benchmark scRNA-seq datasets by comparing per-cluster assigned cell types to literature-reported cell types and computing a consistent rate.

## When to use

Use when a new or existing cluster-level cell type annotation method must be quantitatively benchmarked on scRNA-seq datasets whose true cluster labels are known from published literature.

## When NOT to use

- The datasets have no literature-validated ground-truth cell type labels for clusters, so no consistent rate can be computed
- The goal is per-cell annotation rather than cluster-level annotation; scCATCH annotates clusters, not individual cells
- The tissue or species of the input data is absent from the CellMatch reference database, since performance majorly depends on the reference database and limited references may cause incorrect annotation

## Inputs

- scRNA-seq count matrix for each benchmark dataset (Chen kidney, Xin pancreatic islet, Gierahn peripheral blood)
- Cluster label vector for each dataset
- Literature-reported cell type for each cluster (ground truth)
- CellMatch reference database (cell types, subtypes, tissue types, human and mouse references)

## Outputs

- Per-cluster assigned cell types (top-1 score) per dataset
- Consistent Rate: proportion of clusters whose assigned type matches the literature-reported type
- Benchmark annotation consistency table (e.g., sccatch_benchmark_annotations.csv)

## How to apply

Load each benchmark count matrix (e.g., Chen kidney, Xin pancreatic islet, Gierahn peripheral blood) with its cluster label vector and construct an scCATCH object via createscCATCH(data, cluster). Run findmarkergene() with the appropriate species and tissue to identify marker genes under the paired one-against-all criteria (log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of cells in the cluster), then match them against the CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references). Run findcelltype() to score candidate cell types per cluster on validated markers and supporting literature and assign the top-1 type. Compare every cluster's assigned type to the literature-reported type and compute the proportion of consistent clusters (Consistent Rate); write per-cluster assignments and consistency results to a CSV such as sccatch_benchmark_annotations.csv. Success is judged by a high consistent rate — the paper reports 100% on these three benchmark datasets (67%–100%, average 83%, across six published datasets).

## Related tools

- **scCATCH** (Automatic cluster-level cell type annotation via findmarkergene() and findcelltype() against CellMatch; the method being benchmarked) — https://github.com/ZJUFanLab/scCATCH
- **cellmatch** (Tissue-specific cell taxonomy reference database used to validate marker genes and derive candidate cell types for scoring) — https://github.com/ZJUFanLab/scCATCH/tree/master/data
- **Seurat** (Popular marker gene identification method against which scCATCH's benchmark performance was compared)
- **SingleR** (Cell-based annotation method used in the performance comparison; benchmarked with and without CellMatch)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- All clusters across the three benchmark datasets (Chen, Xin, Gierahn) receive the literature-reported cell type (Consistent Rate of 100%, matching Table 1 and Figure 2)
- Marker gene identification applied the paired one-against-all criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, expression in >= 25% of cells in the cluster
- Every cluster receives a top-1 (highest-score) cell type based on validated marker genes and supporting literature
- Per-cluster assignments and consistency results are written in the expected output schema (e.g., sccatch_benchmark_annotations.csv)
- Across six published scRNA-seq datasets, consistent rates fall within the reported range of 67%–100% (average 83%)

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited CellMatch database may lead to incorrect annotation of cell types
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into cell type annotation
- The major challenge of cell-based strategies — multiple cell types present in one cluster — and pseudo marker genes from one-against-all comparisons can both degrade annotation accuracy
- Effects of multiple clustering algorithms on cell type annotations were not yet evaluated

## Evidence

- [abstract] Benchmark protocol: "scCATCH accurately annotated 67%-100% (average, 83%) clusters in six published scRNA-seq datasets originating from various tissues."
- [intro] Marker gene criteria: "genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%) would be considered marker genes"
