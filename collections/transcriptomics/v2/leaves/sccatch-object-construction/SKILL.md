---
name: sccatch-object-construction
description: Use when you have a scRNA-seq expression matrix plus cluster assignments
  and need to wrap them into a scCATCH object (or convert a Seurat object) to run
  findmarkergene/findcelltype annotation.
when_to_use_negative:
- You do not yet have cluster assignments — clustering must be performed first, since
  inadequate clustering (too many or too few clusters) introduces errors into downstream
  annotation
- You need differential expression markers only and plan to use Seurat's one-against-all
  FindMarkers — scCATCH object construction is specific to scCATCH's paired-cluster
  annotation workflow
- Your data is not a cell-level expression matrix with discrete cluster labels (e.g.
  bulk RNA-seq or a precomputed marker gene table)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: R package providing createscCATCH() to build the scCATCH object from an expression
    matrix and cluster vector as the entry point of the annotation workflow
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Optional upstream source object; a scCATCH object can be created directly
    from a Seurat object's RNA assay normalized data and active cell identities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:15a9fdaacc88117f11cc82fd9447f866fbe3f6174cb47ea7397ac5b84b0a9949
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sccatch-object-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct a scCATCH object from a scRNA-seq expression matrix and cluster labels using createscCATCH, the mandatory first step of scCATCH's automatic annotation pipeline before findmarkergene and findcelltype. scCATCH also supports direct conversion from a Seurat object's normalized data slot.

## When to use

Use when you have a scRNA-seq expression matrix plus cluster assignments and need to wrap them into a scCATCH object (or convert a Seurat object) to run findmarkergene/findcelltype annotation.

## When NOT to use

- You do not yet have cluster assignments — clustering must be performed first, since inadequate clustering (too many or too few clusters) introduces errors into downstream annotation
- You need differential expression markers only and plan to use Seurat's one-against-all FindMarkers — scCATCH object construction is specific to scCATCH's paired-cluster annotation workflow
- Your data is not a cell-level expression matrix with discrete cluster labels (e.g. bulk RNA-seq or a precomputed marker gene table)

## Inputs

- scRNA-seq expression data matrix (sc_data, e.g. genes x cells counts or normalized data)
- Character vector of cluster assignments for each cell (sc_cluster)
- Seurat object (optional alternative source; extract [['RNA']]@data and Idents)

## Outputs

- scCATCH object (obj) containing the expression matrix and cluster information, ready for findmarkergene()

## How to apply

Call createscCATCH(data = sc_data, cluster = sc_cluster), passing the scRNA-seq data matrix as 'data' and a character vector of cluster labels as 'cluster' to build the scCATCH object that downstream functions operate on. If your data lives in a Seurat object, extract the normalized expression matrix via Seurat_obj[['RNA']]@data and the active identities via as.character(Idents(Seurat_obj)) as the cluster argument. The resulting object (e.g. obj) is then passed to findmarkergene(obj, species, marker, tissue, cancer) to identify per-cluster marker genes and to findcelltype(obj) for evidence-based annotation against CellMatch. Validate construction by confirming the object carries both the expression data and cluster information needed for the paired one-against-one cluster comparisons scCATCH performs instead of Seurat's one-against-all approach.

## Related tools

- **scCATCH** (R package providing createscCATCH() to build the scCATCH object from an expression matrix and cluster vector as the entry point of the annotation workflow) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Optional upstream source object; a scCATCH object can be created directly from a Seurat object's RNA assay normalized data and active cell identities)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster)
```

## Evaluation signals

- createscCATCH returns an obj of class scCATCH without error when given a valid data matrix and a cluster character of matching length
- The number of cluster labels in sc_cluster equals the number of cells (columns) in sc_data
- The constructed obj can be passed directly to findmarkergene(obj, species, marker, tissue, cancer) without re-specifying the data or cluster arguments
- When building from Seurat, the extracted matrix is Seurat_obj[['RNA']]@data (normalized data) and clusters come from as.character(Idents(Seurat_obj))

## Limitations

- scCATCH performance majorly depends on the reference database (CellMatch); a limited reference database potentially led to incorrect annotation of cell types
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into cell type annotation, so object construction inherits any clustering problems
- scCATCH requires R >= 4.0 per the README installation badge

## Evidence

- [methods] two-function automatic annotation: [evidence span withheld — anchor status: not_found]
