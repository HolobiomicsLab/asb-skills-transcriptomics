---
name: marker-gep-based-cell-label-transfer
description: Use when annotating cells in a novel single-cell dataset (e.g. TMS FACS)
  using pre-identified marker GEPs from a reference dataset (e.g. MCA) without cell
  clustering.
when_to_use_negative:
- The source dataset's cell-type coverage is insufficient for the target (e.g. annotating
  the Mouse Brain dataset with MCA GEPs, where high-resolution neuronal subtypes are
  absent)
- The reference marker GEPs were not generated via SingleCellGGM co-expression analysis
  and thus lack cell-type-specific expression structure
- Gene IDs in the novel dataset's count matrix cannot be matched to the GEP gene ID
  format (Ensembl vs symbol)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellGEP
  role: 'Core algorithm: annotates cells in the novel dataset from GEP expression
    and neighbor matrices via the cellgep function; sourced from cellgep.R'
  repo: https://github.com/MaShisongLab/cellgep
- name: Seurat
  role: Builds the Seurat object, runs PCA/UMAP, and computes neighbor indices with
    FindNeighbors(return.neighbor=T)
- name: SingleCellGGM
  role: Identifies the reference marker GEPs (e.g. from mouse MCA) via single-cell
    GGM co-expression network analysis
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: edgeR
  role: R dependency required by average_by_gep when computing GEP average expression
    in the novel dataset
- name: graph
  role: R dependency required by the cellgep function
- name: RBGL
  role: R dependency required by the cellgep function
- name: Harmony
  role: Integrates reference and target datasets so that cells annotated with the
    same GEP can be confirmed to co-localize in a shared embedding
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:11b6ce0da2a8acafee1565a48e090e539ca30ff8522ffe240e8b5c7784b2a391
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/cellgep
  tools:
  - CellGEP
  - Harmony
  - RBGL
  - Seurat
  - SingleCellGGM
  - edgeR
  - graph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# marker-gep-based-cell-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CellGEP-based transfer of cell-type labels across single-cell datasets using marker gene expression programs (GEPs) identified in a reference study, enabling annotation of individual cells without clustering.

## When to use

Use when annotating cells in a novel single-cell dataset (e.g. TMS FACS) using pre-identified marker GEPs from a reference dataset (e.g. MCA) without cell clustering.

## When NOT to use

- The source dataset's cell-type coverage is insufficient for the target (e.g. annotating the Mouse Brain dataset with MCA GEPs, where high-resolution neuronal subtypes are absent)
- The reference marker GEPs were not generated via SingleCellGGM co-expression analysis and thus lack cell-type-specific expression structure
- Gene IDs in the novel dataset's count matrix cannot be matched to the GEP gene ID format (Ensembl vs symbol)

## Inputs

- Gene count matrix of the novel single-cell dataset (row.names are gene names matching the GEP gene IDs; Ensembl ID or symbol format)
- Marker GEP table (2 columns: gene name, GEP name), e.g. data/mca.marker.geps.txt or data/mca.marker.symbl.geps.txt
- Seurat object with PCA/UMAP computed for the novel dataset

## Outputs

- Per-cell annotation table (anno$annotation) with columns GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, and neighbor votes N1..N20
- Seurat metadata column marker_gep and a labeled UMAP DimPlot

## How to apply

Build a Seurat object from the target gene count matrix, run PCA/UMAP, and compute neighbor indices with FindNeighbors(return.neighbor=TRUE), extracting sc_object@neighbors$RNA.nn@nn.idx. Read the 70 MCA marker GEPs from data/mca.marker.geps.txt (Ensembl IDs; use mca.marker.symbl.geps.txt if gene IDs are symbols) and compute per-cell GEP average expression via average_by_gep(expression, geps, 'mca', 'novel_sc_dataset'). Run cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=TRUE), where the dynamic cutoff process determines per-GEP expression cutoffs, yielding anno$annotation whose first column is the marker_gep label. Attach the labels to Seurat metadata and visualize with DimPlot(group.by='marker_gep', label=TRUE); optionally integrate reference and target via Harmony to confirm same-GEP cells co-localize.

## Related tools

- **CellGEP** (Core algorithm: annotates cells in the novel dataset from GEP expression and neighbor matrices via the cellgep function; sourced from cellgep.R) — https://github.com/MaShisongLab/cellgep
- **Seurat** (Builds the Seurat object, runs PCA/UMAP, and computes neighbor indices with FindNeighbors(return.neighbor=T))
- **SingleCellGGM** (Identifies the reference marker GEPs (e.g. from mouse MCA) via single-cell GGM co-expression network analysis) — https://github.com/MaShisongLab/SingleCellGGM
- **edgeR** (R dependency required by average_by_gep when computing GEP average expression in the novel dataset)
- **graph** (R dependency required by the cellgep function)
- **RBGL** (R dependency required by the cellgep function)
- **Harmony** (Integrates reference and target datasets so that cells annotated with the same GEP can be confirmed to co-localize in a shared embedding)

## Examples

```
anno = cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T)
```

## Evaluation signals

- Annotation table covers all target cells (e.g. 110,824 TMS FACS cells) with a GEP label or an unannotated ('unknown') state, matching the reported 94,582 annotated / 16,242 unannotated split
- anno$annotation has the expected schema: GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, N1..N20 neighbor-vote columns
- Number of GEPs used equals the row count of geps_expression$expression (gep_num = dim(geps_expression$expression)[1], 70 MCA GEPs)
- DimPlot on UMAP grouped by marker_gep shows coherent spatial domains of same-GEP cells
- After Harmony integration of MCA and TMS, cells carrying the same GEP label from both datasets co-localize in the shared embedding

## Limitations

- Label transfer is constrained by the cell-type coverage of the source dataset, leaving some target cells annotated as 'unknown' (e.g. TMS cells annotated with MCA GEPs)
- Annotation precision is compromised when the source lacks high-resolution subtypes needed by the target (e.g. neuronal subtypes absent from MCA)
- CellGEP is transcriptome-only and its performance depends on gene ID format matching between the target matrix and the GEP file (Ensembl vs symbol)

## Evidence
