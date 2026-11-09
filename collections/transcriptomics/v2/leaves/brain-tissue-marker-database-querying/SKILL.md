---
name: brain-tissue-marker-database-querying
description: Use when annotating scRNA-seq/snRNA-seq clusters from brain or nervous-system
  tissue by querying differential-expression marker lists (e.g. Seurat FindAllMarkers,
  only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) against CellMarker.
when_to_use_negative:
- Dataset is from a non-brain tissue — the CellMarker query must be restricted to
  tissues matching the sample, so a generic brain list would miss the correct cell
  types
- Marker lists are to be compared as plain gene-set overlaps without ranking — the
  article warns that 'simply overlapping the two lists without considering gene ranking
  might lead to unreliable results', so use GSEA on ranked scores instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: R package that queries marker lists against CellMarker via GSEA or a modified
    Fisher's exact test and renders dot-plot visualizations of the annotations
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Standard workflow (NormalizeData, FindVariableFeatures, ScaleData, RunPCA,
    FindNeighbors, FindClusters) and FindAllMarkers to produce the input marker lists
- name: scCATCH
  role: Alternative cluster-annotation tool (CellMatch taxonomy reference) that EasyCellType
    is compared against
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:6d576e74f419eb21607e3fd363a9e19038c7c9b861ac1fb501a244303fb58d1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - AnnotationDbi
  - CellMarker
  - EasyCellType
  - Seurat
  - clusterProfiler
  - ggplot2
  - org.Hs.eg.db
  - scCATCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# brain-tissue-marker-database-querying

## Summary

Annotates clusters from brain single-nucleus RNA-seq by ranking Seurat-derived cluster markers (logFC cutoff 0.25) against the CellMarker database restricted to brain tissues, using EasyCellType's GSEA test with graphical dot-plot output.

## When to use

Use when annotating scRNA-seq/snRNA-seq clusters from brain or nervous-system tissue by querying differential-expression marker lists (e.g. Seurat FindAllMarkers, only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) against CellMarker.

## When NOT to use

- Dataset is from a non-brain tissue — the CellMarker query must be restricted to tissues matching the sample, so a generic brain list would miss the correct cell types
- Marker lists are to be compared as plain gene-set overlaps without ranking — the article warns that 'simply overlapping the two lists without considering gene ranking might lead to unreliable results', so use GSEA on ranked scores instead

## Inputs

- Seurat cluster marker table from FindAllMarkers (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) with gene, cluster and avg_log2FC columns
- Data frame of Entrez ID, cluster and expression score ranked by descending avg_log2FC per cluster (EasyCellType input format)
- Brain-restricted tissue list for the CellMarker database query

## Outputs

- easyct GSEA annotation results per cluster
- Dot plot of candidate cell types per cluster (plot_dot)
- Ranking table of top-five candidate cell types per cluster

## How to apply

Process the count matrix (e.g. Velmeshev 2019 ASD dataset from UCSC Cell Browser) with the standard Seurat workflow and call FindAllMarkers with only.pos=TRUE, min.pct=0.25 and logfc.threshold=0.25 to get per-cluster markers. Convert gene symbols to Entrez IDs with mapIds from org.Hs.eg.db via AnnotationDbi, dropping unmapped entries, then build an input data frame with columns Entrez ID, cluster and expression score (avg_log2FC), ranking genes by descending score within each cluster — ranking matters because simple list overlap without gene ordering can give unreliable results. Run easyct with db="cellmarker", species="Human", a brain-restricted tissue list, and test="GSEA" (the ranking-sensitive alternative to the modified Fisher's exact test). Inspect plot_dot(test="GSEA", results) and extract the top-five ranked candidate cell types per cluster into a ranking table for manual review.

## Related tools

- **EasyCellType** (R package that queries marker lists against CellMarker via GSEA or a modified Fisher's exact test and renders dot-plot visualizations of the annotations) — https://github.com/rx-li/EasyCellType
- **Seurat** (Standard workflow (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters) and FindAllMarkers to produce the input marker lists)
- **scCATCH** (Alternative cluster-annotation tool (CellMatch taxonomy reference) that EasyCellType is compared against) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue=brain_tissues, test="GSEA"); plot_dot(test="GSEA", annot.GSEA)
```

## Evaluation signals

- All input genes successfully mapped to Entrez IDs via org.Hs.eg.db, with unmapped entries removed before easyct
- Within each cluster, genes are ordered by descending avg_log2FC so GSEA can use the ranking
- Six clusters — ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes and OPC — are correctly annotated by the GSEA approach
- Remaining neuronal clusters receive candidate cell types in the top-five ranking table that are consistent with their true labels
- Dot plot (plot_dot(test="GSEA", results)) visually recovers the expected annotations for the six expected clusters

## Limitations

- EasyCellType only refers to 3 publicly available marker gene databases, so annotation quality depends on CellMarker coverage for the tissue of interest
- Simply overlapping gene lists without considering gene ranking can lead to unreliable results — use the GSEA test on ranked scores rather than unranked overlap
- Non-neuronal clusters (ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes, OPC) are exactly recovered, but neuronal clusters only receive candidates 'consistent with' their true labels rather than exact matches

## Evidence

- [abstract] which automatically checks an input marker list obtained by differential expression analysis against the databases and provides annotation recommendations in graphical outcomes: "which automatically checks an input marker list obtained by differential expression analysis against the databases and provides annotation recommendations in graphical outcomes"
- [methods] markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25): "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [methods] annot.GSEA <- easyct(input.d, db="cellmarker", species="Human",: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [readme] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance.: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance."
