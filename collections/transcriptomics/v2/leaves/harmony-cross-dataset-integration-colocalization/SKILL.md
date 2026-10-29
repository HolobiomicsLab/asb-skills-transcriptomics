---
name: harmony-cross-dataset-integration-colocalization
description: Use when cells in two independent scRNA-seq datasets have been assigned
  labels from a common set of marker GEPs (e.g., 70 MCA GEPs annotating 94,582 TMS
  cells) and the transfer must be validated in a joint embedding.
when_to_use_negative:
- The target dataset contains cell types absent from the source reference (e.g., TMS
  cell types not covered by the 70 MCA GEPs), which will leave cells annotated as
  'unknown' and cannot be validated by colocalization
- Only one dataset is available or labels come from different annotation schemes that
  cannot be mapped to a shared GEP vocabulary
- The goal is quantifying differential expression between conditions rather than validating
  cross-study label consistency
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Harmony
  role: Integrates MCA and TMS datasets into a shared embedding so same-GEP cells
    can be checked for colocalization after label transfer
- name: Seurat
  role: Builds per-dataset Seurat objects, runs PCA/UMAP, and renders DimPlot visualizations
    of the integrated annotations
- name: CellGEP
  role: Produces the GEP-based marker_gep annotations in the target dataset that are
    validated in the Harmony joint embedding
  repo: https://github.com/MaShisongLab/cellgep
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:53da3771785b17d670a11b4b44f1d171f244f322ba29d310d0111ba8b922e461
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Harmony cross-dataset integration and label-transfer colocalization check

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates two single-cell datasets (e.g., MCA and TMS FACS) with Harmony to embed them in a shared space, then verifies that cells carrying the same GEP-derived cell-type label from both datasets co-localize, validating cross-study label transfer.

## When to use

Use when cells in two independent scRNA-seq datasets have been assigned labels from a common set of marker GEPs (e.g., 70 MCA GEPs annotating 94,582 TMS cells) and the transfer must be validated in a joint embedding.

## When NOT to use

- The target dataset contains cell types absent from the source reference (e.g., TMS cell types not covered by the 70 MCA GEPs), which will leave cells annotated as 'unknown' and cannot be validated by colocalization
- Only one dataset is available or labels come from different annotation schemes that cannot be mapped to a shared GEP vocabulary
- The goal is quantifying differential expression between conditions rather than validating cross-study label consistency

## Inputs

- Processed expression matrices from two single-cell datasets (e.g., MCA reference and TMS FACS target) with cells carrying GEP-based annotations (marker_gep metadata column)
- Per-dataset Seurat objects with PCA/UMAP embeddings
- CellGEP annotation table (anno$annotation) with GEP labels for target-dataset cells

## Outputs

- Harmony-integrated joint embedding of the two datasets
- UMAP/DimPlot visualization showing co-localization of same-GEP-labeled cells from both datasets
- Qualitative validation of cross-study cell-type label transfer (Figure 6 outcome)

## How to apply

After CellGEP annotates cells in the target dataset (e.g., 110,824 TMS FACS cells annotated with 70 MCA GEPs via the cellgep function with dynamic.cutoff=TRUE), integrate the source and target datasets with Harmony so that dataset-specific batch effects are removed while biological variation is preserved. Then inspect the shared embedding (e.g., UMAP built on Harmony-corrected embeddings) and check that cells assigned the same GEP label — one set from the MCA reference, one from the TMS target — occupy the same region of the space. Colocalization of same-GEP cells from both datasets is the evidence that the GEP-based labels are conserved across studies and the transfer is biologically meaningful; dispersed same-label cells indicate the label transfer does not hold in the shared space.

## Related tools

- **Harmony** (Integrates MCA and TMS datasets into a shared embedding so same-GEP cells can be checked for colocalization after label transfer)
- **Seurat** (Builds per-dataset Seurat objects, runs PCA/UMAP, and renders DimPlot visualizations of the integrated annotations)
- **CellGEP** (Produces the GEP-based marker_gep annotations in the target dataset that are validated in the Harmony joint embedding) — https://github.com/MaShisongLab/cellgep

## Examples

```
sc_object@meta.data$marker_gep = anno$annotation[,1]; DimPlot(sc_object, group.by = 'marker_gep', label = T)
```

## Evaluation signals

- Cells carrying the same GEP label from both MCA and TMS co-localize in similar cell spaces in the Harmony-integrated embedding
- The CellGEP annotation table upstream of integration covers 110,824 TMS FACS cells with 94,582 annotated and 16,242 unannotated, matching the reported Figure 6 outcome
- DimPlot(group.by='marker_gep', label=TRUE) on the integrated object shows each GEP cluster containing cells from both datasets rather than dataset-separated clusters
- Dataset-specific batch structure is reduced (source and target cells intermix within same-label regions) rather than remaining split by dataset

## Limitations

- Label transfer is constrained by the cell-type coverage of the source dataset: MCA GEPs left 16,242 TMS cells annotated as 'unknown'
- Using MCA GEPs to annotate datasets with high-resolution subtypes absent from MCA (e.g., neuronal subtypes in the Mouse Brain dataset) would compromise annotation precision
- Harmony integration supports but does not by itself prove label correctness; colocalization is a qualitative confirmation

## Evidence

- [other] TMS annotation counts: [evidence span withheld — anchor status: not_found]
- [readme] Integration step in workflow: [evidence span withheld — anchor status: not_found]
