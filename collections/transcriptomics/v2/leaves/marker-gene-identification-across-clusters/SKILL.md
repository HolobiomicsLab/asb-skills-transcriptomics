---
name: marker-gene-identification-across-clusters
description: Use when you have a clustered scRNA-seq matrix and need cluster-level
  marker genes free of pseudo markers (upregulated in multiple clusters) before automatic
  cell type annotation.
when_to_use_negative:
- The goal is cell-level (not cluster-based) annotation — cell-based methods like
  SingleR, CellAssign, Garnett, scMap, or CHETAH are the appropriate comparison class
- The reference database (CellMatch) does not cover the tissue/cell types of interest
  — scCATCH performance majorly depends on the reference database and a limited one
  may cause incorrect annotation
- Clustering itself is inadequate (too many or too few clusters), which would introduce
  errors regardless of marker quality
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Provides createscCATCH() and findmarkergene() to build the object and perform
    paired cluster-vs-cluster marker gene identification with the log10FC ≥0.25, p<0.05,
    ≥25% expression filters
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Optional upstream source of clustering; scCATCH objects can be created directly
    from a Seurat object's RNA @data slot and Idents
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:2bd7917c9c345e9a0de7aa8d9231e4bf816d7a9aaf077290f3f92704e6232f88
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# marker-gene-identification-across-clusters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies potential marker genes for each cell cluster by paired one-against-all comparison across clusters (scCATCH findmarkergene), filtering for genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in ≥25% of cells, as the basis for downstream cluster annotation against CellMatch.

## When to use

Use when you have a clustered scRNA-seq matrix and need cluster-level marker genes free of pseudo markers (upregulated in multiple clusters) before automatic cell type annotation.

## When NOT to use

- The goal is cell-level (not cluster-based) annotation — cell-based methods like SingleR, CellAssign, Garnett, scMap, or CHETAH are the appropriate comparison class
- The reference database (CellMatch) does not cover the tissue/cell types of interest — scCATCH performance majorly depends on the reference database and a limited one may cause incorrect annotation
- Clustering itself is inadequate (too many or too few clusters), which would introduce errors regardless of marker quality

## Inputs

- scRNA-seq expression data matrix (sc_data) with cluster labels (sc_cluster), or a Seurat object
- species argument ('Human' or 'Mouse') for marker matching
- tissue and cancer arguments selecting the relevant CellMatch subpanel

## Outputs

- scCATCH object updated with candidate marker genes per cluster (findmarkergene result), ready for findcelltype annotation

## How to apply

Build an scCATCH object from the expression matrix and cluster labels with createscCATCH(data = sc_data, cluster = sc_cluster), or extract from a Seurat object via Seurat_obj[['RNA']]@data and as.character(Idents(Seurat_obj)). Then call findmarkergene(obj, species, marker, tissue, cancer), specifying species ('Human'/'Mouse') and the relevant tissue/cancer combination so the paired comparison is tissue-aware. The function performs a paired comparison of every cluster against every other cluster, retaining genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05 by Wilcoxon test) that are expressed in more than a quarter of cells (≥25%) — this avoids the pseudo marker genes produced by Seurat-style one-against-all testing. The resulting per-cluster marker list feeds findcelltype(obj), which scores cell types against the CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references). Rationale: restricting markers to cluster-specific upregulation and a ≥25% expression floor reduces incorrect annotation driven by pseudo markers.

## Related tools

- **scCATCH** (Provides createscCATCH() and findmarkergene() to build the object and perform paired cluster-vs-cluster marker gene identification with the log10FC ≥0.25, p<0.05, ≥25% expression filters) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Optional upstream source of clustering; scCATCH objects can be created directly from a Seurat object's RNA @data slot and Idents) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = cellmatch, tissue = 'Pancreas', cancer = 'Pancreatic cancer')
```

## Evaluation signals

- Marker set per cluster contains only genes upregulated in that single cluster (no pseudo markers shared across ≥2 clusters)
- Retained genes satisfy all three thresholds: log10 fold change ≥0.25, p < 0.05, and expression in ≥25% of cells
- species/marker/tissue/cancer arguments were supplied so the CellMatch panel matches the dataset's tissue
- Downstream findcelltype consistency on benchmark datasets falls within the reported 67%–100% range with ~83% average across the six validation datasets (Enge, Wu, Lindsey, Zheng, Zeisel, Heng)

## Limitations

- Performance depends on the reference database: the limited CellMatch panel may lead to incorrect cell type annotation
- Inadequate clustering (too many or too few cells per cluster) introduces errors into the marker-based annotation
- Effects of multiple clustering algorithms on the resulting annotations had not been evaluated by the authors

## Evidence

- [methods] The scCATCH mainly includes two function findmarkergene() and findcelltype(): [evidence span withheld — anchor status: not_found]
