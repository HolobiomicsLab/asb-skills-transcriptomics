---
name: single-cell-rna-seq-normalization-and-scaling
description: Use when a raw Seurat object of single-cell RNA-seq data must be prepared
  for Connectome's CreateConnectome — i.e. before ligand-receptor z-score calculation
  on the panc8, ifnb, or similar annotated scRNA-seq objects.
when_to_use_negative:
- Input is already normalized/scaled or is a feature table rather than a raw Seurat
  scRNA-seq object — re-normalizing would distort z-score-based edge weights.
- The goal is downstream network visualization only (NetworkPlot, CircosPlot) on an
  already-built connectome — no re-normalization is needed.
- Cell type identities or clusters are not yet annotated on the Seurat object; Connectome
  defines nodes from the active identity slot.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides NormalizeData and ScaleData used to normalize the data, identify
    mapped ligand/receptor genes, and scale those genes before connectome creation.
- name: Connectome
  role: Consumes the normalized/scaled Seurat object via CreateConnectome; its z-score-based
    edge attributes depend on this preprocessing.
  repo: https://github.com/msraredon/Connectome
- name: SeuratData
  role: Distributes the example scRNA-seq objects (panc8 cross-platform pancreas data,
    ifnb PBMC data) used as inputs to this normalization/scaling workflow.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:2a9adb2ea2005d4409e18496fa1b75e90b4984e44151198b746e607fc26d4e36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-normalization-and-scaling

## Summary

Normalize and scale single-cell RNA-seq expression data in Seurat before connectomic ligand-receptor mapping: NormalizeData on the raw counts, subsetting to ligand/receptor genes from a ground-truth database (e.g. ncomms8866_human from FANTOM5/Ramilowski 2015), and ScaleData on those genes so that edge z-scores in CreateConnectome are computed on comparable expression distributions.

## When to use

Use when a raw Seurat object of single-cell RNA-seq data must be prepared for Connectome's CreateConnectome — i.e. before ligand-receptor z-score calculation on the panc8, ifnb, or similar annotated scRNA-seq objects.

## When NOT to use

- Input is already normalized/scaled or is a feature table rather than a raw Seurat scRNA-seq object — re-normalizing would distort z-score-based edge weights.
- The goal is downstream network visualization only (NetworkPlot, CircosPlot) on an already-built connectome — no re-normalization is needed.
- Cell type identities or clusters are not yet annotated on the Seurat object; Connectome defines nodes from the active identity slot.

## Inputs

- Seurat 3.0 object (e.g. panc8 or ifnb from SeuratData) with active identity slot defining network nodes
- Ligand-receptor database table, e.g. Connectome::ncomms8866_human with Ligand.ApprovedSymbol / Receptor.ApprovedSymbol columns

## Outputs

- Normalized, gene-subsetted and scaled Seurat object ready as input to CreateConnectome
- Connectome edgelist (from CreateConnectome) with edge attributes including ligand/receptor z-scores, amenable to FilterConnectome

## How to apply

Starting from a Seurat 3.0 object with identities set (e.g. the 'celltype' annotation of the cross-platform panc8 dataset from SeuratData), run NormalizeData to normalize the data, then take the union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns from the ground-truth database (Connectome::ncomms8866_human for human, ncomms8866_mouse for mouse) and subset the object to those mapped ligand/receptor genes. Scale those subsetted genes with ScaleData so that per-gene z-scores are comparable across cell types. Only then call CreateConnectome (species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE), since downstream FilterConnectome thresholds (min.pct=0.1, min.z=0.25, remove.na=TRUE) depend on correctly scaled expression. Restricting analysis to clusters with > 75 captured cells avoids unstable per-identity statistics from sparse populations.

## Related tools

- **Seurat** (Provides NormalizeData and ScaleData used to normalize the data, identify mapped ligand/receptor genes, and scale those genes before connectome creation.)
- **Connectome** (Consumes the normalized/scaled Seurat object via CreateConnectome; its z-score-based edge attributes depend on this preprocessing.) — https://github.com/msraredon/Connectome
- **SeuratData** (Distributes the example scRNA-seq objects (panc8 cross-platform pancreas data, ifnb PBMC data) used as inputs to this normalization/scaling workflow.)

## Examples

```
panc8 <- NormalizeData(panc8); connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol, Connectome::ncomms8866_human$Receptor.ApprovedSymbol); panc8 <- ScaleData(panc8, features = connectome.genes)
```

## Evaluation signals

- The scaled object's variable list contains only genes present in the union of the database's Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns.
- CreateConnectome runs without error on the scaled object using min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE.
- FilterConnectome with min.pct=0.1, min.z=0.25, remove.na=TRUE yields a non-empty, parseable edgelist.
- Downstream Centrality rankings (e.g. endothelial cells as dominant VEGF-family receivers in panc8) qualitatively match the published pancreas topology.
- Identities with fewer than 75 captured cells are excluded from the resulting connectome.

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; further validation is required downstream.
- Output quality is bounded by the ground-truth database against which the original mapping takes place (FANTOM5 by default).
- The 'minimum cells per identity' filter (> 75 cells) drops rare cell populations from the network entirely.

## Evidence

- [methods] Normalization and gene subsetting precede connectome creation: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [readme] CreateConnectome input is a Seurat object; identities define nodes: [evidence span withheld — anchor status: not_found]
- [methods] Concrete CreateConnectome parameters on the panc8 data: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] Database gene union used for subsetting: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [methods] Clusters below 75 cells are excluded: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
