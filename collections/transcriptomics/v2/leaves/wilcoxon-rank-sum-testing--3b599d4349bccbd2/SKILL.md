---
name: wilcoxon-rank-sum-testing
description: Use when identifying per-cluster marker genes from a clustered single-cell
  RNA-seq matrix prior to cell type annotation, and when avoiding pseudo marker genes
  that are upregulated in multiple clusters.
when_to_use_negative:
- Input is already a curated marker gene list or feature table rather than a raw expression
  matrix with cluster labels
- The goal is one-against-all differential expression rather than scCATCH's paired
  one-against-one comparison to eliminate pseudo marker genes upregulated in multiple
  clusters
- Clusters are inadequately resolved (too many or too few clusters), which the article
  notes introduces errors into downstream cell type annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Implements findmarkergene() with Wilcoxon rank-sum one-against-one paired
    cluster comparison and the logfc/pvalue/cell_min_pct filters
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Contrasted one-against-all marker gene method whose pseudo marker genes motivate
    the paired Wilcoxon approach
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:ca1923c5db7fea70156d21733a011a8da7f40482a35360dc2cffced32b0608e9
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
  - scCATCH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# wilcoxon-rank-sum-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Wilcoxon rank-sum (Wilcoxon rank-sum/one-against-one paired cluster) testing within scCATCH's findmarkergene step to identify genes significantly upregulated in only one cluster, filtering by log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells. This paired-comparison design avoids pseudo marker genes produced by one-against-all approaches.

## When to use

Use when identifying per-cluster marker genes from a clustered single-cell RNA-seq matrix prior to cell type annotation, and when avoiding pseudo marker genes that are upregulated in multiple clusters.

## When NOT to use

- Input is already a curated marker gene list or feature table rather than a raw expression matrix with cluster labels
- The goal is one-against-all differential expression rather than scCATCH's paired one-against-one comparison to eliminate pseudo marker genes upregulated in multiple clusters
- Clusters are inadequately resolved (too many or too few clusters), which the article notes introduces errors into downstream cell type annotation

## Inputs

- scRNA-seq expression data matrix (sc_data) as used by createscCATCH
- Character vector of cluster assignments (sc_cluster) for each cell
- Species and tissue specification for the paired comparison context (e.g., demo_data with demo_cluster)

## Outputs

- scCATCH object with available marker genes per cluster
- Per-cluster marker gene table (e.g., marker_genes.csv) with fold change, Wilcoxon p-value, and percent-expressed columns

## How to apply

Build a scCATCH object from the scRNA-seq matrix and cluster labels using createscCATCH(data, cluster), then call findmarkergene(obj, species, marker, tissue, cancer) with if_use_custom_marker = FALSE to trigger scCATCH's built-in one-against-one paired cluster comparison, where each cluster is compared against every other cluster. Retain as potential marker genes only those significantly upregulated in exactly one cluster, applying the Figure 1A thresholds: log10 fold change >= 0.25 (log10 scale), Wilcoxon rank-sum p < 0.05, and expression in at least 25% (cell_min_pct = 0.25) of the cluster's cells. The rationale, per the article, is that one-against-all methods (e.g., Seurat's) can yield pseudo marker genes significantly upregulated in at least two clusters, which would lead to incorrect cell type annotation. Extract the per-cluster marker gene table from the returned scCATCH object for downstream findcelltype scoring against the CellMatch database.

## Related tools

- **scCATCH** (Implements findmarkergene() with Wilcoxon rank-sum one-against-one paired cluster comparison and the logfc/pvalue/cell_min_pct filters) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Contrasted one-against-all marker gene method whose pseudo marker genes motivate the paired Wilcoxon approach)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmatch', tissue = 'Bone marrow', cancer = 'all')
```

## Evaluation signals

- Every retained marker gene row satisfies log10 fold change >= 0.25 and Wilcoxon p < 0.05
- Every retained gene is expressed in >= 25% of cells (cell_min_pct = 0.25) in its assigned cluster
- Each retained gene is significantly upregulated in exactly one cluster, not in two or more clusters (no pseudo marker genes)
- The output marker_genes.csv exists and is non-empty, with per-cluster gene tables present for each input cluster

## Limitations

- Downstream annotation performance majorly depends on the reference database (CellMatch); a limited reference database can lead to incorrect cell type annotation even with correct marker genes
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into the marker gene identification and annotation process
- One-against-all marker gene identification (as in Seurat) may produce pseudo marker genes upregulated in at least two clusters, leading to incorrect annotation if not replaced by the paired comparison

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
