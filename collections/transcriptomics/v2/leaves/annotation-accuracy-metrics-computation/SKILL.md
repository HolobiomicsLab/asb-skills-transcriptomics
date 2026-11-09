---
name: annotation-accuracy-metrics-computation
description: Use when CellGEP/cellgep has assigned marker-GEP labels to single cells
  and you must quantify how faithfully those assignments match original cell-type
  or clustering labels.
when_to_use_negative:
- The target dataset contains cell types absent from the reference dataset's coverage
  (e.g., TMS cells annotated with MCA GEPs become 'unknown'), making purity comparisons
  misleading
- You need quantitative accuracy for high-resolution subtypes the reference lacks
  — e.g., MCA GEPs cannot precisely validate Mouse Brain neuronal subtypes because
  MCA lacks high-resolution neuronal cell types
- The cell labels being evaluated come from clustering rather than GEP-based assignment,
  since this skill validates the GEP-annotation rule specifically
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellGEP
  role: Produces the per-cell GEP annotation table (via the cellgep function) whose
    purities and ARI/NMI this skill evaluates
  repo: https://github.com/MaShisongLab/cellgep
- name: SingleCellGGM
  role: Upstream algorithm whose marker GEPs (from mca.marker.geps.txt) define the
    cell-type labels being scored
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: Seurat
  role: Builds the Seurat object, PCA/UMAP, and neighbor index (FindNeighbors(return.neighbor
    = T)) that feed the annotations being evaluated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:8eef4b817a5905a757dcc58b07748f48ccc4cc44fa46dd57e9a350ccb7cb97a7
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
  - ACTINN
  - CellAssign
  - CellGEP
  - RBGL
  - SCSA
  - Seurat
  - SingleCellGGM
  - SingleCellNet
  - SingleR
  - edgeR
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- annotation-accuracy-metric-computation
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# annotation-accuracy-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate GEP-based direct cell-type annotations produced by CellGEP against reference labels by computing per-GEP annotation purity (e.g., 99.6% for M1 oligodendrocytes, 99.2% for M3 T cells) plus ARI and NMI against cluster annotations.

## When to use

Use when CellGEP/cellgep has assigned marker-GEP labels to single cells and you must quantify how faithfully those assignments match original cell-type or clustering labels.

## When NOT to use

- The target dataset contains cell types absent from the reference dataset's coverage (e.g., TMS cells annotated with MCA GEPs become 'unknown'), making purity comparisons misleading
- You need quantitative accuracy for high-resolution subtypes the reference lacks — e.g., MCA GEPs cannot precisely validate Mouse Brain neuronal subtypes because MCA lacks high-resolution neuronal cell types
- The cell labels being evaluated come from clustering rather than GEP-based assignment, since this skill validates the GEP-annotation rule specifically

## Inputs

- Per-cell CellGEP annotation table from anno$annotation (GEP, GEP_id, Expression, GEPs_expressed, neighbor columns)
- Reference cell-type labels for the MCA dataset (e.g., oligodendrocyte and T-cell labels)
- Cluster annotations from comparison methods (scDHA, scMGCA) for ARI/NMI computation

## Outputs

- Per-GEP annotation purity values (e.g., M1 = 99.6% of 1,125 cells; M3 = 99.2% of 1,886 cells)
- ARI and NMI metrics against scDHA/scMGCA cluster annotations
- cellgep_annotation_metrics.csv containing the per-cell annotation table and purity/ARI/NMI metrics

## How to apply

After running cellgep with dynamic.cutoff=T over all GEPs (geps_to_use = 1:gep_num), take the per-cell annotation table (columns GEP, GEP_id, Expression, GEPs_expressed, neighbor columns) and cross-tabulate each GEP's assigned cells against reference labels from the source dataset (e.g., MCA oligodendrocyte and T-cell labels). Compute per-GEP purity as the fraction of a GEP's annotated cells sharing the dominant reference label — expected values are 1,120/1,125 (99.6%) for M1 (oligodendrocytes) and 99.2% of 1,886 cells for M3 (T cells). Additionally compute ARI and NMI between the GEP annotations and external cluster annotations (e.g., from scDHA and scMGCA). Write per-cell annotations and purity/ARI/NMI values to cellgep_annotation_metrics.csv; reproduced purities must match the reported 99.6% and 99.2% within a small tolerance.

## Related tools

- **CellGEP** (Produces the per-cell GEP annotation table (via the cellgep function) whose purities and ARI/NMI this skill evaluates) — https://github.com/MaShisongLab/cellgep
- **SingleCellGGM** (Upstream algorithm whose marker GEPs (from mca.marker.geps.txt) define the cell-type labels being scored) — https://github.com/MaShisongLab/SingleCellGGM
- **Seurat** (Builds the Seurat object, PCA/UMAP, and neighbor index (FindNeighbors(return.neighbor = T)) that feed the annotations being evaluated)

## Examples

```
anno = cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T)
```

## Evaluation signals

- M1 annotation purity reproduces 1,120/1,125 (99.6%) oligodendrocytes within a small tolerance
- M3 annotation purity reproduces 99.2% of 1,886 cells assigned as T cells
- Annotation table schema matches expected columns: GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, GEP_Ori, N1..N20
- ARI and NMI are computed against scDHA and scMGCA cluster annotations and recorded in cellgep_annotation_metrics.csv
- Every annotated cell has a GEP assignment consistent with the dynamic cutoff rule (expressed at >=1/2 of its maximal average expression, highest-expressed GEP wins, 20-neighbor KNN voting)

## Limitations

- Label transfer is constrained by the source dataset's cell-type coverage, leaving some cells annotated as 'unknown' (e.g., TMS cells annotated with MCA GEPs)
- Annotating the Mouse Brain dataset with MCA GEPs is compromised by the absence of high-resolution neuronal cell types in MCA
- SingleCellGGM relies exclusively on transcriptome data, so accuracy metrics cannot reflect other modalities (ATAC-seq, proteomics)

## Evidence

- [other] Applying the marker GEP assignment rule produced cell-type-specific annotations: 99.6% of cells annotated by M1 are oligodendrocytes: [evidence span withheld — anchor status: not_found]
- [readme] The cellgep function uses GEPs to annotate cells with a dynamic cutoff process: "The `cellgep`function will using the GEPs to annotate the cells in the novel dataset."
- [readme] Neighbors are identified via Seurat FindNeighbors with return.neighbor = T for KNN voting: "sc_object = FindNeighbors(sc_object, return.neighbor = T)"
- [abstract] GEPs enable direct cell-type annotation without clustering, the claim the metrics validate: "These GEPs enable direct cell-type annotation of individual cells without cell clustering"
- [discussion] Label transfer coverage limitation affects annotation completeness: "using GEPs from the MCA dataset to annotate the TMS dataset is constrained by the cell-type coverage in MCA, resulting in some cells in TMS being annotated as “unknown.”"
