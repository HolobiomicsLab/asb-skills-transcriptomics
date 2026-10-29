---
name: tissue-specific-marker-database-matching
description: Use when scRNA-seq data are already clustered and each cluster needs
  automatic, reproducible cell type annotation via a literature-backed tissue-specific
  marker database instead of manual marker inspection.
when_to_use_negative:
- Cells are not yet clustered — scCATCH requires a pre-computed cluster vector and
  does not perform clustering itself; inadequate clustering (too many or too few clusters)
  introduces annotation errors.
- The target tissue, cell types, or species are absent or poorly covered by CellMatch
  — the paper notes performance majorly depends on the reference database and a limited
  reference database can lead to incorrect annotation.
- Single-cell (cell-based) annotation is required rather than cluster-based annotation,
  e.g. when multiple distinct cell types may coexist within one cluster.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: scCATCH
  role: 'Primary toolkit: createscCATCH builds the object from data+cluster, findmarkergene()
    identifies cluster-specific markers under the paper''s thresholds, findcelltype()
    scores and assigns the top-1 cell type per cluster using CellMatch'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes,
    184 tissue types, 2,096 human/mouse references) against which marker genes are
    matched; also improved SingleR and CellAssign accuracies when applied externally
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
- name: Seurat
  role: Upstream clustering tool; a scCATCH object can be created directly from a
    Seurat object's RNA assay data and Idents
- name: rev_gene
  role: scCATCH helper for cross-species gene-symbol conversion (e.g. via geneinfo)
    so markers match the reference database
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:0f4525792085c70619fbf9d7eb5dc7f97438b939dc881b354d8b0351a1396c14
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
  - rev_gene
  - scCATCH
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tissue-specific-marker-database-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates each pre-computed scRNA-seq cluster by matching its cluster-specific marker genes against the CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references), scoring cell types on validated markers with literature support, and assigning the top-1 scoring cell type per cluster.

## When to use

Use when scRNA-seq data are already clustered and each cluster needs automatic, reproducible cell type annotation via a literature-backed tissue-specific marker database instead of manual marker inspection.

## When NOT to use

- Cells are not yet clustered — scCATCH requires a pre-computed cluster vector and does not perform clustering itself; inadequate clustering (too many or too few clusters) introduces annotation errors.
- The target tissue, cell types, or species are absent or poorly covered by CellMatch — the paper notes performance majorly depends on the reference database and a limited reference database can lead to incorrect annotation.
- Single-cell (cell-based) annotation is required rather than cluster-based annotation, e.g. when multiple distinct cell types may coexist within one cluster.

## Inputs

- scRNA-seq count matrix (e.g. demo_data)
- character vector of cluster labels for each cell (e.g. from a Seurat object's Idents)
- species, tissue, and cancer specification for findmarkergene()
- CellMatch database (cellmatch) or a custom cellmatch reference; geneinfo table for cross-species gene-symbol conversion

## Outputs

- scCATCH object updated with per-cluster marker genes and annotated cell types
- per-cluster cell type annotations (e.g. sccatch_annotations.csv with one label per input cluster)

## How to apply

Construct the scCATCH object from the count matrix and cluster vector with createscCATCH(data, cluster). Run findmarkergene() with species, marker, tissue, and cancer arguments, which identifies genes upregulated in only one cluster by paired comparison using the paper's marker criteria: log10 fold change >= 0.25, p < 0.05 (Wilcoxon), and expression in >= 25% of cells; this paired strategy avoids pseudo marker genes that arise from one-against-all approaches. Cross-species gene symbols are converted via rev_gene/geneinfo where needed. Run findcelltype() to score cell types per cluster on the validated marker genes and their supporting literature from CellMatch, assigning the top-1 scoring cell type to each cluster. Extract the per-cluster annotation results (e.g. write to sccatch_annotations.csv) and validate that one valid CellMatch cell type is returned per input cluster.

## Related tools

- **scCATCH** (Primary toolkit: createscCATCH builds the object from data+cluster, findmarkergene() identifies cluster-specific markers under the paper's thresholds, findcelltype() scores and assigns the top-1 cell type per cluster using CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **CellMatch** (Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) against which marker genes are matched; also improved SingleR and CellAssign accuracies when applied externally) — https://github.com/ZJUFanLab/scCATCH/tree/master/data
- **Seurat** (Upstream clustering tool; a scCATCH object can be created directly from a Seurat object's RNA assay data and Idents)
- **rev_gene** (scCATCH helper for cross-species gene-symbol conversion (e.g. via geneinfo) so markers match the reference database) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Output annotation file exists and contains exactly one cell type label per input cluster.
- Every assigned label is a valid cell type present in CellMatch (353 cell types / 686 subtypes).
- Identified marker genes satisfy the criteria: log10 fold change >= 0.25, p < 0.05 by Wilcoxon paired-cluster comparison, and expression in >= 25% of cells, upregulated in only one cluster.
- Qualitative benchmark: annotation accuracy in the range of the paper's reported 67%-100% (average 83%) across six published scRNA-seq datasets.
- Marker gene set contains no pseudo marker genes (i.e. no genes significantly upregulated in two or more clusters).

## Limitations

- Performance majorly depends on the reference database; a limited CellMatch coverage can lead to incorrect cell type annotation.
- Inadequate clustering analysis (too many or too few clusters) introduces errors into the annotation.
- Applying markers from species other than human or mouse requires custom reference handling; multiple clustering algorithms' effects on annotations have not yet been evaluated.

## Evidence

- [readme] The scCATCH mainly includes two function findmarkergene() and findcelltype() to realize the automatic annotation for each identified cluster: [evidence span withheld — anchor status: not_found]
