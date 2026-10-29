---
name: dynamic-expression-cutoff-annotation
description: Use when a reference set of cell-type-specific marker GEPs (e.g., 70
  MCA identity GEPs from mca.marker.geps.txt) must directly annotate cells in an independent
  single-cell dataset without clustering, enabling universal cell-type label transfer.
when_to_use_negative:
- The source reference dataset lacks cell types present in the target (e.g., MCA's
  limited coverage leaves 16,242 TMS cells unannotated), so missing labels would be
  misread as annotation failure
- The target dataset contains high-resolution subtypes absent from the reference (e.g.,
  annotating Mouse Brain neurons with MCA GEPs), which would compromise annotation
  precision
- A clustering-based annotation pipeline is required instead of clustering-free, GEP-based
  single-cell labeling
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellGEP
  role: 'Core algorithm: cellgep() votes over the 20-neighbor graph to assign GEP
    labels with dynamic per-GEP expression cutoffs'
  repo: https://github.com/MaShisongLab/cellgep
- name: Seurat
  role: Builds the target-dataset object, runs PCA/UMAP, and computes the 20-nearest-neighbor
    index matrix via FindNeighbors(return.neighbor = T)
- name: edgeR
  role: R dependency required by average_by_gep when computing GEP average expression
    in the novel dataset
- name: graph
  role: R dependency required by cellgep() during annotation
- name: RBGL
  role: R dependency required by cellgep() during annotation
- name: Harmony
  role: Integrates MCA and TMS embeddings to validate that cells with the same GEP
    label co-localize across datasets
- name: SingleCellGGM
  role: Upstream method that identified the 70 MCA marker GEPs used as the reference
    label-transfer signature
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:473bb91a5307901f806f08cd55f2fe3cee759e9f34611804f59dfa63cdb6c8c7
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# dynamic-expression-cutoff-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate individual cells in a novel single-cell dataset by transferring marker gene expression programs (GEPs) identified in a reference dataset via the CellGEP algorithm, using neighbor voting over a Seurat kNN graph and dynamic per-GEP expression cutoffs, demonstrated by labeling 94,582 TMS FACS cells with 70 MCA identity GEPs.

## When to use

Use when a reference set of cell-type-specific marker GEPs (e.g., 70 MCA identity GEPs from mca.marker.geps.txt) must directly annotate cells in an independent single-cell dataset without clustering, enabling universal cell-type label transfer.

## When NOT to use

- The source reference dataset lacks cell types present in the target (e.g., MCA's limited coverage leaves 16,242 TMS cells unannotated), so missing labels would be misread as annotation failure
- The target dataset contains high-resolution subtypes absent from the reference (e.g., annotating Mouse Brain neurons with MCA GEPs), which would compromise annotation precision
- A clustering-based annotation pipeline is required instead of clustering-free, GEP-based single-cell labeling

## Inputs

- Gene count matrix of the novel dataset (cells x genes, row.names as gene names matching GEP gene IDs)
- Reference marker GEP table (2 columns: gene name, GEP name; e.g., data/mca.marker.geps.txt with 70 MCA GEPs)
- Seurat object for the novel dataset (PCA and UMAP computed)
- Neighbor index matrix from Seurat FindNeighbors (20 nearest neighbors per cell)

## Outputs

- GEP expression matrix (per-cell average expression of each marker GEP via average_by_gep)
- Annotation table anno$annotation with columns GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, and neighbor votes N1..N20
- Per-cell GEP label vector attached to Seurat metadata as 'marker_gep' (unannotated cells left blank/unknown)
- UMAP DimPlot colored by transferred GEP labels; optional Harmony-integrated MCA-TMS embedding confirming co-localization

## How to apply

Build a Seurat object from the target gene count matrix, run PCA and UMAP, and compute neighbors with FindNeighbors(return.neighbor = TRUE), extracting sc_object@neighbors$RNA.nn@nn.idx as a 20-neighbor index matrix. Read the 70 MCA marker GEPs (Ensembl-ID version data/mca.marker.geps.txt, or mca.marker.symbl.geps.txt for symbols) and compute per-cell GEP average expression with average_by_gep, passing gep_origin = 'mca'. Call cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff = TRUE); the dynamic.cutoff option determines, per GEP, the cutoff above which the GEP counts as expressed, avoiding fixed global thresholds. Attach anno$annotation[,1] as the 'marker_gep' metadata column and visualize with DimPlot(group.by = 'marker_gep', label = TRUE). Cells whose GEP expression falls below all dynamic cutoffs remain unannotated ('unknown'), reflecting the reference dataset's cell-type coverage.

## Related tools

- **CellGEP** (Core algorithm: cellgep() votes over the 20-neighbor graph to assign GEP labels with dynamic per-GEP expression cutoffs) — https://github.com/MaShisongLab/cellgep
- **Seurat** (Builds the target-dataset object, runs PCA/UMAP, and computes the 20-nearest-neighbor index matrix via FindNeighbors(return.neighbor = T))
- **edgeR** (R dependency required by average_by_gep when computing GEP average expression in the novel dataset)
- **graph** (R dependency required by cellgep() during annotation)
- **RBGL** (R dependency required by cellgep() during annotation)
- **Harmony** (Integrates MCA and TMS embeddings to validate that cells with the same GEP label co-localize across datasets)
- **SingleCellGGM** (Upstream method that identified the 70 MCA marker GEPs used as the reference label-transfer signature) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
geps = read.table('data/mca.marker.geps.txt'); geps_expression = average_by_gep(expression, geps, 'mca', 'novel_sc_dataset'); anno = cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T)
```

## Evaluation signals

- The annotation table covers all target cells (110,824 TMS FACS cells), with 94,582 cells receiving a GEP annotation and 16,242 unannotated, matching the reported Figure 6 outcome
- anno$annotation contains the expected schema: GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, and neighbor votes N1..N20
- Each GEP's Expression value exceeds the dynamically determined cutoff implied by GEPs_expressed (e.g., 'M002-1.01' encoding GEP, expression, GEP_mean 0.107, GEP_sd 0.121)
- DimPlot on UMAP shows spatially coherent clusters of cells sharing the same marker_gep label
- After Harmony integration of MCA and TMS, cells carrying the same GEP label from both datasets co-localize in the shared embedding

## Limitations

- Label transfer is constrained by the cell-type coverage of the source dataset: MCA coverage limits left some TMS cells annotated as 'unknown' (16,242 cells)
- Using MCA GEPs to annotate high-resolution neuronal subtypes (e.g., Mouse Brain dataset) would be compromised by the absence of fine-grained neuronal cell types in MCA
- SingleCellGGM-derived GEPs rely exclusively on transcriptome data, neglecting other single-cell modalities such as ATAC-seq or single-cell proteomics
- The GEP file format must match the target gene identifiers: use mca.marker.geps.txt for Ensembl IDs and mca.marker.symbl.geps.txt for gene symbols

## Evidence

- [methods] FindNeighbors with return.neighbor = TRUE yields the neighbor index matrix: [evidence span withheld — anchor status: not_found]
- [discussion] MCA GEPs annotated 94,582 TMS cells via 70 GEPs, leaving 16,242 unannotated: [evidence span withheld — anchor status: not_found]
