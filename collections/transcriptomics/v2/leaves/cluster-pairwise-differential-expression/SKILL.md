---
name: cluster-pairwise-differential-expression
description: Use when preprocessing scRNA-seq clusters for automatic cell type annotation
  and you need cluster marker genes free of pseudo markers from one-against-all testing.
when_to_use_negative:
- Input is a precomputed marker gene list or feature table rather than a raw expression
  matrix with cluster labels — create the scCATCH object from the matrix instead
- The goal is a full one-against-all differential expression profile per cluster (e.g.,
  general DE analysis), since this skill deliberately discards genes upregulated in
  more than one cluster
- Clustering is inadequate (too many or too few cells per cluster), which the article
  notes introduces errors into downstream annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Provides createscCATCH to construct the cluster-labeled object and findmarkergene
    to run the paired one-against-one cluster differential test with the logfc/pvalue/cell_min_pct
    filters
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: 'Contrast baseline: Seurat uses one-against-all marker gene identification,
    which scCATCH''s paired comparison replaces to avoid pseudo marker genes'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:538650d03a06887bdf59b92f7b7e13d340533dceddf32cd65191104f1ac2a45a
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cluster-pairwise-differential-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify potential marker genes for each single-cell RNA-seq cluster using scCATCH's one-against-one paired comparison (findmarkergene), which avoids pseudo marker genes produced by one-against-all approaches by retaining genes significantly upregulated in only one cluster.

## When to use

Use when preprocessing scRNA-seq clusters for automatic cell type annotation and you need cluster marker genes free of pseudo markers from one-against-all testing.

## When NOT to use

- Input is a precomputed marker gene list or feature table rather than a raw expression matrix with cluster labels — create the scCATCH object from the matrix instead
- The goal is a full one-against-all differential expression profile per cluster (e.g., general DE analysis), since this skill deliberately discards genes upregulated in more than one cluster
- Clustering is inadequate (too many or too few cells per cluster), which the article notes introduces errors into downstream annotation

## Inputs

- scRNA-seq expression data matrix (e.g., demo_data or a Seurat RNA @data slot)
- Character vector of cluster labels for each cell (e.g., demo_cluster)
- Species and tissue specification for the CellMatch reference context

## Outputs

- scCATCH object populated with available marker genes per cluster
- Per-cluster marker gene table (e.g., marker_genes.csv) with fold change, p-value, and percent-expressed values

## How to apply

Build a scCATCH object with createscCATCH(data = sc_data, cluster = sc_cluster) from the expression matrix and cluster labels, then call findmarkergene with species, marker, tissue, and cancer settings. Unlike Seurat's one-against-all scheme, findmarkergene performs a paired comparison of each cluster against every other cluster and retains a gene as a marker only if it is significantly upregulated in exactly one cluster: log10 fold change >= 0.25, Wilcoxon rank-sum test p < 0.05, and expression in >= 25% of that cluster's cells (cell_min_pct = 0.25). Leave if_use_custom_marker = FALSE to use the built-in one-against-one comparison, or supply custom markers if external knowledge must be incorporated. The rationale is that pseudo marker genes (significantly upregulated in at least two clusters rather than one) cause incorrect cell type annotation downstream in findcelltype. Extract the available marker genes from the returned object and write the per-cluster marker gene table (e.g., marker_genes.csv) for use by the evidence-based scoring step.

## Related tools

- **scCATCH** (Provides createscCATCH to construct the cluster-labeled object and findmarkergene to run the paired one-against-one cluster differential test with the logfc/pvalue/cell_min_pct filters) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Contrast baseline: Seurat uses one-against-all marker gene identification, which scCATCH's paired comparison replaces to avoid pseudo marker genes)

## Examples

```
obj <- createscCATCH(data = demo_data, cluster = demo_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmatch', tissue = 'Bone marrow', cancer = 'all')
```

## Evaluation signals

- marker_genes.csv exists, is non-empty, and every retained gene row satisfies log10 fold change >= 0.25
- Every retained gene has Wilcoxon rank-sum test p < 0.05
- Every retained gene is expressed in >= 25% of cells in its assigned cluster
- Each retained gene is significantly upregulated in exactly one cluster (no gene upregulated in two or more clusters survives the paired comparison)
- The returned scCATCH object contains non-empty available marker genes for each cluster

## Limitations

- Downstream annotation performance majorly depends on the reference database; a limited CellMatch database potentially led to incorrect cell type annotation
- Genes upregulated in more than one cluster are discarded by design, so this step yields markers specific to single clusters only
- Inadequate clustering (too many or too few cells per cluster) would introduce errors into this marker identification and the subsequent annotation

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
