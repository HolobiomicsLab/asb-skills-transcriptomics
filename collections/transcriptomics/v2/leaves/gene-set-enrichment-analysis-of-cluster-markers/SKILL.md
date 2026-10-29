---
name: gene-set-enrichment-analysis-of-cluster-markers
description: Use when you have Seurat FindAllMarkers output (only.pos=TRUE, min.pct=0.25,
  logfc.threshold=0.25) for scRNA-seq clusters and need reference-database cell-type
  annotations that respect gene ranking rather than simple list overlap.
when_to_use_negative:
- When a curated reference expression atlas and pre-labeled cell types are available
  — SingleR reference-based annotation may be more appropriate
- When gene ranking information is unavailable and only an unranked marker list exists,
  since ranking-aware GSEA is the method's key advantage over overlap-based tools
- When markers come from bulk differential expression rather than per-cluster scRNA-seq
  markers, as the input schema expects cluster-level marker lists
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EasyCellType
  role: Provides the easyct() GSEA test that checks the ranked marker list against
    the CellMarker database and returns annotations with visualization
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Generates the per-cluster positive marker lists via FindAllMarkers that serve
    as GSEA input
- name: org.Hs.eg.db
  role: Maps Seurat marker gene symbols to Entrez IDs with mapIds for the easyct()
    input
- name: CellMarker
  role: Reference marker-gene database queried by the GSEA test (db="cellmarker")
- name: scCATCH
  role: Independent marker-matching annotation tool (findmarkergene/findcelltype on
    CellMatch) used to benchmark the GSEA-based annotations
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:69bd80fac73dcce762475a1b59082ae8e90ad8ea12905b59053c2958fbf2b37e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Seurat
  - clusterProfiler
  - gene set enrichment analysis
  - org.Hs.eg.db
  - scCATCH
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gene-set-enrichment-analysis-of-cluster-markers

## Summary

Apply EasyCellType's GSEA-based test to rank-ordered per-cluster marker gene lists from Seurat differential expression to annotate single-cell clusters against CellMarker, as demonstrated on PBMC3k and TNBC normal-cell data.

## When to use

Use when you have Seurat FindAllMarkers output (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) for scRNA-seq clusters and need reference-database cell-type annotations that respect gene ranking rather than simple list overlap.

## When NOT to use

- When a curated reference expression atlas and pre-labeled cell types are available — SingleR reference-based annotation may be more appropriate
- When gene ranking information is unavailable and only an unranked marker list exists, since ranking-aware GSEA is the method's key advantage over overlap-based tools
- When markers come from bulk differential expression rather than per-cluster scRNA-seq markers, as the input schema expects cluster-level marker lists

## Inputs

- Seurat FindAllMarkers output table (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) with gene symbols, cluster, and score columns
- Seurat object processed through the standard QC/clustering workflow (e.g. pbmc3k filtered gene-barcode matrix from 10X Genomics)
- org.Hs.eg.db for symbol-to-Entrez-ID mapping

## Outputs

- Per-cluster cell-type annotation table from easyct() with candidate cell types, test statistics, and p-values at p_cut=0.3
- Graphical visualization of GSEA test results per cluster for manual review of optimal annotations
- Final per-cluster label assignments (e.g. tnbc_cluster_annotations.csv integrating EasyCellType, SingleR, and scCATCH calls)

## How to apply

Run the standard Seurat workflow (NormalizeData, FindVariableFeatures with nfeatures=2000, ScaleData, RunPCA, FindNeighbors on dims 1:10, FindClusters at resolution 0.5) and call FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25). Convert marker gene symbols to Entrez IDs with mapIds from org.Hs.eg.db, then build an input data frame with columns Entrez ID, cluster, and score, sorted by descending score within each cluster — ranking matters because the article warns that 'simply overlapping the two lists without considering gene ranking might lead to unreliable results'. Call easyct() with db="cellmarker", species="Human", a relevant tissue term, p_cut=0.3, and test="GSEA" to obtain per-cluster candidate cell types with graphical output that supports picking the optimal annotation. In the TNBC application (GSE176078, tumor clusters removed), this correctly annotated CAF, T cell, PVL, epithelial, and endothelial clusters, performing slightly better than scCATCH and comparably to SingleR.

## Related tools

- **EasyCellType** (Provides the easyct() GSEA test that checks the ranked marker list against the CellMarker database and returns annotations with visualization) — https://github.com/rx-li/EasyCellType
- **Seurat** (Generates the per-cluster positive marker lists via FindAllMarkers that serve as GSEA input)
- **org.Hs.eg.db** (Maps Seurat marker gene symbols to Entrez IDs with mapIds for the easyct() input)
- **CellMarker** (Reference marker-gene database queried by the GSEA test (db="cellmarker"))
- **scCATCH** (Independent marker-matching annotation tool (findmarkergene/findcelltype on CellMatch) used to benchmark the GSEA-based annotations) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue="blood-immune", p_cut=0.3, test="GSEA")
```

## Evaluation signals

- Markers passed to easyct() are Entrez-ID keyed, contain the required columns (Entrez ID, cluster, score), and are sorted by descending score within each cluster
- easyct() runs with the documented parameters: db="cellmarker", species="Human", p_cut=0.3, test="GSEA"
- Per-cluster annotations match expected labels in benchmark data — e.g. CAF, T cell, PVL, epithelial, and endothelial clusters in the TNBC normal-cell application
- Agreement with independent annotations (SingleR, scCATCH) is checked in a single per-cluster comparison table
- GSEA graphical output is inspected per cluster so that the chosen label is supported by the top-ranked enriched cell type

## Limitations

- Simply overlapping gene lists without considering gene ranking leads to unreliable results — the input must preserve score order
- The package refers to only 3 publicly available marker gene databases, so coverage is bounded by CellMarker and companion databases
- Tissue-term selection matters: candidate tissues must be inspected via data(cellmarker_tissue) before choosing the tissue argument

## Evidence

- [abstract] GSEA ranks markers against CellMarker: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] ranking matters: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [methods] Seurat marker parameters: "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [methods] easyct invocation: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [methods] Entrez mapping step: "markers$entrezid <- mapIds(org.Hs.eg.db,"
- [readme] scCATCH usage from README: [evidence span withheld — anchor status: external_doc]
