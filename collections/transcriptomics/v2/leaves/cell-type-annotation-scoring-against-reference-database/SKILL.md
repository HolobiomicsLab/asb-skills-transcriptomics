---
name: cell-type-annotation-scoring-against-reference-database
description: Use when scRNA-seq clusters (e.g., from Seurat) need automatic, knowledge-free
  cell type annotation by scoring cluster marker genes against a tissue-specific cell
  taxonomy reference database rather than manual marker inspection.
when_to_use_negative:
- The cell types, species, or tissue of the sample are poorly represented in the CellMatch
  reference database — the article notes limited reference content can lead to incorrect
  cell type annotation
- Clustering itself is inadequate (too many or too few clusters), since clustering
  errors propagate into annotation
- The task is cell-level annotation rather than cluster-level annotation, e.g., comparing
  against cell-based methods like SingleR, CellAssign, Garnett, scMap, or CHETAH
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: 'Core toolkit: createscCATCH() builds the object, findmarkergene() identifies
    cluster marker genes, findcelltype() scores and assigns cell types against CellMatch'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Upstream clustering and preprocessing; provides the data matrix and cluster
    identities fed into createscCATCH()
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database used by findcelltype()
    for evidence-based scoring of candidate cell types
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:90f714b52945a2c14bec653de4beaa1b57d45d9fb6e99b678e626f308fd5c265
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-annotation-scoring-against-reference-database

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate single-cell RNA-seq clusters automatically by identifying cluster marker genes and scoring candidate cell types against the tissue-specific CellMatch taxonomy database (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) with an evidence-based score, assigning the top-1 (highest-scoring) cell type per cluster.

## When to use

Use when scRNA-seq clusters (e.g., from Seurat) need automatic, knowledge-free cell type annotation by scoring cluster marker genes against a tissue-specific cell taxonomy reference database rather than manual marker inspection.

## When NOT to use

- The cell types, species, or tissue of the sample are poorly represented in the CellMatch reference database — the article notes limited reference content can lead to incorrect cell type annotation
- Clustering itself is inadequate (too many or too few clusters), since clustering errors propagate into annotation
- The task is cell-level annotation rather than cluster-level annotation, e.g., comparing against cell-based methods like SingleR, CellAssign, Garnett, scMap, or CHETAH

## Inputs

- scRNA-seq expression data matrix (sc_data), e.g., Seurat_obj[['RNA']]@data
- Character vector of cluster assignments (sc_cluster), e.g., as.character(Idents(Seurat_obj))
- Species specification (human or mouse) and target tissue/cancer combination

## Outputs

- scCATCH object with per-cluster marker genes identified by findmarkergene()
- Per-cluster top-1 cell type annotations from findcelltype()
- Cluster-consistent rate when compared to manual annotations (e.g., Table 2-style consistency_table.csv)

## How to apply

Build an scCATCH object with createscCATCH(data = sc_data, cluster = sc_cluster) from the expression matrix and cluster assignments (e.g., Seurat identities via Idents(Seurat_obj)). Run findmarkergene(obj, species, marker, tissue, cancer) using paired one-against-rest comparison so that genes significantly upregulated in only one cluster (log10 fold change >= 0.25, p < 0.05, Wilcoxon test) and expressed in >= 25% of cells are retained as markers, filtering pseudo marker genes that are upregulated in multiple clusters. Then run findcelltype(obj) to score cell types per cluster on the basis of validated marker genes and their supporting literature in CellMatch, assigning the cell type with the highest score (top 1) to each cluster. Select species (human or mouse) and restrict tissue/cancer combinations to the sample's origin to improve specificity. Compare automatic annotations against published manual labels to estimate the cluster-consistent rate, which for validated datasets falls within 67%-100% with an average of 83%.

## Related tools

- **scCATCH** (Core toolkit: createscCATCH() builds the object, findmarkergene() identifies cluster marker genes, findcelltype() scores and assigns cell types against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Upstream clustering and preprocessing; provides the data matrix and cluster identities fed into createscCATCH())
- **CellMatch** (Tissue-specific cellular taxonomy reference database used by findcelltype() for evidence-based scoring of candidate cell types) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Each annotated dataset's cluster-consistent rate falls within the reported 67%-100% range
- Average cluster-consistent rate across the six published datasets (Enge, Wu, Lindsey, Zheng, Zeisel, Heng) equals approximately 83%
- Retained marker genes satisfy all three criteria: log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells in the cluster
- Each cluster receives exactly one top-1 (highest-scoring) cell type assignment
- Internal (CellMatch-recorded) and external (unrecorded) datasets both show average consistency near 83%

## Limitations

- Performance majorly depends on the reference database; a limited CellMatch database potentially led to incorrect annotation of cell types
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into the annotation
- Pseudo marker genes (significantly upregulated in at least two clusters rather than one) can cause incorrect annotation if not filtered by paired comparison

## Evidence
