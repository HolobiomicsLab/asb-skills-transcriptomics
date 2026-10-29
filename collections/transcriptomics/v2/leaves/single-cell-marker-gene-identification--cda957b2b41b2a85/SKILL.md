---
name: single-cell-marker-gene-identification
description: Use when annotating cell types of scRNA-seq clusters automatically and
  you need cluster marker genes that are upregulated in exactly one cluster, avoiding
  pseudo markers (e.g. CCL22, SWAP70, KLRF1) that arise from one-against-all comparisons
  and impair annotation consistency.
when_to_use_negative:
- The input is not clustered scRNA-seq count data (e.g. a bulk expression matrix or
  an already-annotated feature table) — clustering must precede marker gene identification
- You need marker genes for genes expressed across multiple cell types by design (e.g.
  broad immune activation signatures); the one-against-one filter retains only genes
  upregulated in exactly one cluster
- The tissue/species has no coverage in CellMatch — the article notes scCATCH performance
  majorly depends on the reference database, so a limited reference may lead to incorrect
  annotation regardless of marker gene quality
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Provides findmarkergene() for one-against-one paired cluster marker gene identification
    with the logFC 0.25 / p 0.05 / 25% cell filters, called on an object created by
    createscCATCH()
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Used as the one-against-all comparison baseline whose Wilcoxon rank-sum test
    produces pseudo marker genes (CCL22, SWAP70, KLRF1) that scCATCH's paired approach
    is designed to avoid
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database supplying known cell markers
    against which identified marker genes are matched during annotation
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:864d1f8013836336cae205c82f358eda2287cc75c89e88a293ea30ab49676741
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
  - CellMatch
  - Seurat
  - scCATCH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify potential marker genes for each single-cell RNA-seq cluster using scCATCH's one-against-one paired cluster comparison (findmarkergene), which avoids the pseudo marker genes produced by one-against-all approaches like Seurat's Wilcoxon rank-sum test. Marker genes must be significantly upregulated in only one cluster and meet filter thresholds before feeding into evidence-based cell type annotation.

## When to use

Use when annotating cell types of scRNA-seq clusters automatically and you need cluster marker genes that are upregulated in exactly one cluster, avoiding pseudo markers (e.g. CCL22, SWAP70, KLRF1) that arise from one-against-all comparisons and impair annotation consistency.

## When NOT to use

- The input is not clustered scRNA-seq count data (e.g. a bulk expression matrix or an already-annotated feature table) — clustering must precede marker gene identification
- You need marker genes for genes expressed across multiple cell types by design (e.g. broad immune activation signatures); the one-against-one filter retains only genes upregulated in exactly one cluster
- The tissue/species has no coverage in CellMatch — the article notes scCATCH performance majorly depends on the reference database, so a limited reference may lead to incorrect annotation regardless of marker gene quality

## Inputs

- scRNA-seq count matrix (e.g. Gierahn PBMC count matrix or packaged demo_data)
- Character vector of cluster labels for each cell (e.g. demo_cluster)
- species and tissue settings for the dataset (e.g. Human, PBMC)
- Optional custom marker database (cellmatch); otherwise the built-in CellMatch panel of 353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references

## Outputs

- Per-cluster marker gene table (marker_genes.csv / pbmc_marker_genes.csv) with log10 fold change, Wilcoxon p-value, and percent-expressed columns
- scCATCH object populated with available marker genes for each cluster, ready for findcelltype()
- List of flagged pseudo marker genes upregulated in >= 2 clusters (e.g. CCL22, SWAP70, KLRF1) when auditing one-against-all methods

## How to apply

Build an scCATCH object from the count matrix and cluster labels with createscCATCH(data, cluster). Run findmarkergene() with the species and tissue set and if_use_custom_marker = FALSE to trigger the built-in one-against-one paired cluster comparison, in which each cluster is compared against every other cluster. Apply the Figure 1A filter criteria: log10 fold change >= 0.25 (log10 scale), Wilcoxon rank-sum test p < 0.05, and expression in >= 25% of the cluster's cells (cell_min_pct = 0.25). Retain only genes significantly upregulated in exactly one cluster — this is the key rationale, since one-against-all methods (e.g. Seurat's Wilcoxon test) generate pseudo marker genes upregulated in multiple clusters, which led to only 40% annotation consistency (clusters 2 and 5 only) on the Gierahn PBMC dataset. Extract the per-cluster marker gene table from the returned scCATCH object and pass it onward to findcelltype() for CellMatch-based annotation.

## Related tools

- **scCATCH** (Provides findmarkergene() for one-against-one paired cluster marker gene identification with the logFC 0.25 / p 0.05 / 25% cell filters, called on an object created by createscCATCH()) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Used as the one-against-all comparison baseline whose Wilcoxon rank-sum test produces pseudo marker genes (CCL22, SWAP70, KLRF1) that scCATCH's paired approach is designed to avoid)
- **CellMatch** (Tissue-specific cell taxonomy reference database supplying known cell markers against which identified marker genes are matched during annotation)

## Examples

```
obj <- createscCATCH(data = demo_data, cluster = demo_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmatch', tissue = 'Blood')
```

## Evaluation signals

- Every retained gene row in marker_genes.csv satisfies log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of cells in its assigned cluster
- Each retained marker gene is significantly upregulated in exactly one cluster, not two or more (one-against-one invariant)
- Auditing a one-against-all run (e.g. Seurat) on the Gierahn PBMC dataset flags CCL22, SWAP70, and KLRF1 as pseudo markers upregulated in >= 2 clusters, and reproduces 40% annotation consistency with only clusters 2 (T cells) and 5 (monocytes) correct
- marker_genes.csv exists, is non-empty, and contains one gene set per input cluster
- The downstream findcelltype() call against CellMatch yields a cell type (top 1 score) for clusters with sufficient validated markers

## Limitations

- The paired comparison cannot rescue annotation when the reference database is limited: 'The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types via scCATCH.'
- Inadequate clustering analysis (too many or too few cells per cluster) would introduce errors into cell type annotation downstream of marker gene identification
- The article did not evaluate the effects of multiple clustering algorithms on cell type annotations, so marker gene quality is coupled to whichever clustering was supplied

## Evidence

- [intro] Pseudo marker gene failure mode: [evidence span withheld — anchor status: not_found]
