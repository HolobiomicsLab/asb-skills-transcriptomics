---
name: cell-type-annotation-by-gsea
description: Use when you have Seurat cluster marker lists (only.pos=TRUE, min.pct=0.25,
  logfc.threshold=0.25) from scRNA-seq data and need automated, database-backed cell-type
  annotation for each cluster.
when_to_use_negative:
- Input is already a curated cell-type feature/reference table rather than differential-expression
  marker lists from clustering — no Seurat-derived markers to query
- Working with species or tissues outside the supported marker databases (EasyCellType/CellMarker
  coverage is human/mouse-oriented; scCATCH's CellMatch covers human and mouse references)
- Only a single-cell expression matrix is available with no cluster identities — annotation
  is per-cluster, so clustering and marker calling must precede this skill
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: Runs the GSEA-based annotation of cluster marker lists against CellMarker
    and visualizes results
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Standard scRNA-seq workflow (QC, normalization, clustering) and FindAllMarkers
    to produce the input marker lists
- name: AnnotationDbi / org.Hs.eg.db
  role: Converts gene symbols to Entrez IDs with mapIds before querying the database
- name: CellMarker
  role: Marker gene database queried by easyct (db="cellmarker", species="Human")
- name: scCATCH
  role: Alternative cluster-annotation tool compared against EasyCellType; matches
    marker genes against CellMatch tissue-specific taxonomy
  repo: https://github.com/ZJUFanLab/scCATCH
- name: shiny
  role: Interactive GUI application for users who prefer graphical annotation instead
    of scripted easyct calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:9e524b9fde9e936edaed0c00420a9fb62d608648a54163f3274244a01632c536
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - AnnotationDbi
  - AnnotationDbi / org.Hs.eg.db
  - CellMarker
  - EasyCellType
  - Seurat
  - clusterProfiler
  - ggplot2
  - org.Hs.eg.db
  - scCATCH
  - shiny
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 25
---

# cell-type-annotation-by-gsea

## Summary

Annotate scRNA-seq clusters by running EasyCellType's GSEA test on Seurat-derived cluster markers (FindAllMarkers with logfc.threshold=0.25) against the CellMarker database, using gene ranking rather than simple list overlap to recover biologically consistent cell-type labels.

## When to use

Use when you have Seurat cluster marker lists (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) from scRNA-seq data and need automated, database-backed cell-type annotation for each cluster.

## When NOT to use

- Input is already a curated cell-type feature/reference table rather than differential-expression marker lists from clustering — no Seurat-derived markers to query
- Working with species or tissues outside the supported marker databases (EasyCellType/CellMarker coverage is human/mouse-oriented; scCATCH's CellMatch covers human and mouse references)
- Only a single-cell expression matrix is available with no cluster identities — annotation is per-cluster, so clustering and marker calling must precede this skill

## Inputs

- scRNA-seq or single-nucleus count matrix (e.g. Velmeshev ASD dataset from UCSC Cell Browser, or pbmc3k from 10X Genomics)
- Seurat object with clusters assigned via FindNeighbors/FindClusters
- FindAllMarkers output (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25)
- Entrez ID mapping via org.Hs.eg.db

## Outputs

- EasyCellType GSEA annotation results per cluster (easyct output object)
- Dot plot of candidate cell types per cluster (plot_dot)
- Top-five ranked candidate cell types per cluster (ranking table)

## How to apply

Process the count matrix with the standard Seurat workflow (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters) and call FindAllMarkers(pbmc, only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) to obtain cluster-specific markers. Convert gene symbols to Entrez IDs with mapIds from org.Hs.eg.db (AnnotationDbi) and drop unmapped entries, then build the EasyCellType input data frame with columns Entrez ID, cluster and expression score (avg_log2FC), ranking genes by descending score within each cluster — ranking matters because simple gene-list overlap without ranking can give unreliable results. Run easyct with db="cellmarker", species="Human", a tissue-restricted list (e.g. brain tissues), and test="GSEA". Inspect plot_dot(test="GSEA", results) and extract the top-five ranked candidate cell types per cluster into a ranking table for manual review. Validation: confirm that known clusters (e.g. Microglia, Oligodendrocytes, OPC, Endothelial in a brain dataset) are recovered as the top annotation and that remaining clusters receive candidates consistent with their true labels.

## Related tools

- **EasyCellType** (Runs the GSEA-based annotation of cluster marker lists against CellMarker and visualizes results) — https://github.com/rx-li/EasyCellType
- **Seurat** (Standard scRNA-seq workflow (QC, normalization, clustering) and FindAllMarkers to produce the input marker lists)
- **AnnotationDbi / org.Hs.eg.db** (Converts gene symbols to Entrez IDs with mapIds before querying the database)
- **CellMarker** (Marker gene database queried by easyct (db="cellmarker", species="Human"))
- **scCATCH** (Alternative cluster-annotation tool compared against EasyCellType; matches marker genes against CellMatch tissue-specific taxonomy) — https://github.com/ZJUFanLab/scCATCH
- **shiny** (Interactive GUI application for users who prefer graphical annotation instead of scripted easyct calls)

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue=c("Brain"), test="GSEA"); plot_dot(test="GSEA", annot.GSEA)
```

## Evaluation signals

- Known clusters are recovered as top annotations: e.g. ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes and OPC correctly annotated in the Velmeshev ASD brain dataset
- Remaining (neuronal) clusters receive top-five candidate cell types consistent with their true labels, even if not the single top hit
- Input data frame has exactly columns Entrez ID, cluster and expression score (avg_log2FC), with genes ranked by descending score within each cluster
- All unmapped gene symbols were removed after mapIds conversion before running easyct
- Dot plot generated by plot_dot(test="GSEA", results) renders per-cluster candidate types ranked by GSEA significance

## Limitations

- Gene ranking matters: simply overlapping marker lists without considering ranking might lead to unreliable results
- Gene symbols without an Entrez mapping must be dropped, losing those markers from the query
- Article's method text does not fully enumerate the supported marker databases ('3 public available marker gene data bases') nor report software version numbers

## Evidence

- [methods] markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25): "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [methods] easyct call: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [methods] mapIds conversion: "markers$entrezid <- mapIds(org.Hs.eg.db,
                           keys=markers$gene"
- [abstract] ranking caveat: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [intro] two tests offered: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [readme] interactive GUI: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance."
