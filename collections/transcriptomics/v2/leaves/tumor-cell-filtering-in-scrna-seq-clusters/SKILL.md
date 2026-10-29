---
name: tumor-cell-filtering-in-scrna-seq-clusters
description: Use when annotating normal cells within a tumor scRNA-seq dataset (e.g.
  TNBC GSE176078) and malignant clusters must be excluded first so cluster markers
  and cell-type calls reflect non-malignant biology only.
when_to_use_negative:
- The dataset contains no malignant cells (e.g. PBMC 3k) — there is nothing to filter
  and QC-based subsetting alone suffices
- The goal is to annotate or characterize the tumor cells themselves; this skill removes
  them from the annotation scope
- Tumor clusters have not yet been identified — cluster-level exclusion requires prior
  malignant-cell labels, not just per-cell QC metrics
tools:
- name: Seurat
  role: Clusters cells and provides subset() to drop annotated tumor-cell clusters
    while retaining normal cells
- name: EasyCellType
  role: Performs marker-based annotation of the retained non-malignant clusters via
    easyct() against CellMarker (GSEA, p_cut=0.3)
  repo: rx-li/EasyCellType
- name: scCATCH
  role: Independent reference-based annotation (CellMatch) of the filtered clusters
    for comparison
  repo: ZJUFanLab/scCATCH
- name: SingleR
  role: Reference-based annotation of the filtered clusters used as the third comparison
    call
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:401882ea8aa7a0f56279c873ee62fe130138fc757332e200b8eccc7c753c47aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Seurat
  - SingleR
  - clusterProfiler
  - org.Hs.eg.db
  - scCATCH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tumor-cell-filtering-in-scrna-seq-clusters

## Summary

Removes previously annotated malignant tumor-cell clusters from a tumor scRNA-seq dataset (e.g. the Wu et al. 2021a TNBC dataset, GSE176078) so that downstream marker-based annotation with EasyCellType/SingleR/scCATCH operates only on non-malignant populations (CAFs, T cells, PVL, epithelial, endothelial).

## When to use

Use when annotating normal cells within a tumor scRNA-seq dataset (e.g. TNBC GSE176078) and malignant clusters must be excluded first so cluster markers and cell-type calls reflect non-malignant biology only.

## When NOT to use

- The dataset contains no malignant cells (e.g. PBMC 3k) — there is nothing to filter and QC-based subsetting alone suffices
- The goal is to annotate or characterize the tumor cells themselves; this skill removes them from the annotation scope
- Tumor clusters have not yet been identified — cluster-level exclusion requires prior malignant-cell labels, not just per-cell QC metrics

## Inputs

- Seurat object after clustering (or raw 10X/GEO count matrix, e.g. GSE176078 from Wu et al. 2021a TNBC dataset)
- List or labels of annotated tumor-cell clusters to exclude (from prior dataset annotation)

## Outputs

- Filtered Seurat object containing only non-malignant clusters (CAFs, T cells, PVL, epithelial, endothelial)
- tnbc_cluster_annotations.csv — per-cluster annotation table integrating EasyCellType, SingleR, and scCATCH calls

## How to apply

Start from the count matrix (for TNBC, download GSE176078 from GEO and use the published tumor-cluster annotations from Wu et al. 2021a to identify malignant clusters). Process all cells with the standard Seurat workflow — CreateSeuratObject with min.cells=3 and min.features=200, mitochondrial QC (subset nFeature_RNA 200–2500 and percent.mt < 5), NormalizeData, FindVariableFeatures(nfeatures=2000), ScaleData, RunPCA, FindNeighbors(dims 1:10), FindClusters(resolution=0.5) — then subset the Seurat object to drop the annotated tumor-cell clusters, retaining normal cells. Rationale: EasyCellType queries CellMarker for tissue/cell-type labels, so tumor clusters with aberrant marker profiles would otherwise contaminate per-cluster FindAllMarkers output (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) and mislead the Entrez-ID-scored input used by easyct(). After filtering, re-run marker detection on the retained clusters and annotate them with easyct(db="cellmarker", test="GSEA", p_cut=0.3), plus SingleR and scCATCH for comparison, integrating the three calls into a per-cluster table (tnbc_cluster_annotations.csv).

## Related tools

- **Seurat** (Clusters cells and provides subset() to drop annotated tumor-cell clusters while retaining normal cells)
- **EasyCellType** (Performs marker-based annotation of the retained non-malignant clusters via easyct() against CellMarker (GSEA, p_cut=0.3)) — rx-li/EasyCellType
- **scCATCH** (Independent reference-based annotation (CellMatch) of the filtered clusters for comparison) — ZJUFanLab/scCATCH
- **SingleR** (Reference-based annotation of the filtered clusters used as the third comparison call)

## Examples

```
pbmc_normal <- subset(pbmc, idents = setdiff(levels(pbmc), tumor_cluster_ids))
```

## Evaluation signals

- All cluster barcodes from the Wu et al. 2021a tumor annotation are absent from the filtered object; no malignant cluster survives
- Retained clusters resolve to the expected five non-malignant labels: CAFs, T cells, PVL, epithelial, endothelial
- EasyCellType annotation of the filtered clusters agrees with SingleR and scCATCH (reported: slightly better than scCATCH, comparable to SingleR)
- easyct() runs on a well-formed input data frame with columns Entrez ID, cluster and score, sorted by descending score within each cluster
- Final labels are written to tnbc_cluster_annotations.csv with one row per cluster and no missing cluster assignments

## Limitations

- Tumor-cluster identification relies on existing dataset annotations (here the Wu et al. 2021a TNBC labels); without prior malignant labels the filtering decision is not grounded
- Tissue/cancer scope terms for downstream annotation must be inspected explicitly (e.g. via data(cellmarker_tissue) for blood-immune and breast tissue terms); only three public marker databases are referenced and tissue database names are not fully enumerated
- Marker-based annotation can be unreliable when gene ranking is ignored — 'simply overlapping the two lists without considering gene ranking might lead to unreliable results'

## Evidence

- [other] workflow step 1 (source card): [evidence span withheld — anchor status: not_found]
- [other] TNBC application finding (source card): [evidence span withheld — anchor status: not_found]
- [methods] Seurat QC subsetting (article methods): "pbmc <- subset(pbmc, subset = nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)"
- [methods] Per-cluster marker detection (article methods): "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [readme] scCATCH Seurat-object input (README): [evidence span withheld — anchor status: external_doc]
