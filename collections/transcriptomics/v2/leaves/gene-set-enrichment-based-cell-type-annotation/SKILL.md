---
name: gene-set-enrichment-based-cell-type-annotation
description: Use when you have a Seurat object with clusters and a per-cluster positive
  marker list (e.g. FindAllMarkers output mapped to Entrez IDs) and need automatic,
  ranking-aware cell-type annotation instead of manual marker matching.
when_to_use_negative:
- When you lack a ranked marker list — merely overlapping an unranked gene set against
  marker databases may lead to unreliable results per the article
- When reference-based annotation with a well-matched curated reference is preferred
  and the query dataset deviates strongly from the tissue terms available in CellMarker
- When the tissue of origin has no matching entries in the cell marker databases (check
  data(cellmarker_tissue) first)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: EasyCellType
  role: Core package that runs GSEA and modified Fisher's exact test of the marker
    list against cell marker databases and renders annotation visualizations
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Produces the clustered object and per-cluster positive markers (FindAllMarkers)
    that feed the annotation input
- name: org.Hs.eg.db
  role: AnnotationDbi mapIds source for converting marker gene SYMBOLs to Entrez IDs
    required by easyct()
- name: clusterProfiler
  role: Supports the gene set enrichment analysis machinery used for annotation scoring
- name: scCATCH
  role: Baseline cluster-annotation tool (findmarkergene/findcelltype over CellMatch)
    used for comparison in benchmarks
  repo: https://github.com/ZJUFanLab/scCATCH
- name: SingleR
  role: Reference-based baseline annotator used to benchmark EasyCellType on the TNBC
    dataset
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:57896f6aef82aa574113e0c762b0dd6284ba2bc739ad0904846c12643d83d99c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - AnnotationDbi
  - CellMarker
  - EasyCellType
  - Seurat
  - SingleR
  - clusterProfiler
  - dplyr
  - gene set enrichment analysis
  - org.Hs.eg.db
  - scCATCH
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# gene-set-enrichment-based-cell-type-annotation

## Summary

Annotate scRNA-seq clusters by running GSEA (or a modified Fisher's exact test) on ranked per-cluster marker gene lists against curated cell marker databases, as implemented in the EasyCellType R package, producing per-cluster cell-type candidate scores and visualizations.

## When to use

Use when you have a Seurat object with clusters and a per-cluster positive marker list (e.g. FindAllMarkers output mapped to Entrez IDs) and need automatic, ranking-aware cell-type annotation instead of manual marker matching.

## When NOT to use

- When you lack a ranked marker list — merely overlapping an unranked gene set against marker databases may lead to unreliable results per the article
- When reference-based annotation with a well-matched curated reference is preferred and the query dataset deviates strongly from the tissue terms available in CellMarker
- When the tissue of origin has no matching entries in the cell marker databases (check data(cellmarker_tissue) first)

## Inputs

- Per-cluster positive marker data frame with columns entrezid, cluster, score (avg_log2FC), sorted by descending score within each cluster
- Seurat object from a QC-filtered and clustered scRNA-seq matrix (e.g. 10X filtered gene-barcode matrices such as pbmc3k)
- CellMarker tissue terms appropriate to the sample (e.g. Blood, Peripheral blood, Blood vessel, Umbilical cord blood, Venous blood)

## Outputs

- Per-cluster cell-type annotation recommendations from easyct() for each test variant (GSEA / modified Fisher's exact test, hard and soft)
- Graphical visualization of test results per cluster to guide optimal annotation selection
- Annotation accuracy table/figure when compared against baseline tools (CellAssign, scCATCH, SCINA, SCSA, SingleR)

## How to apply

Process the scRNA-seq data with the standard Seurat workflow (CreateSeuratObject with min.cells=3, min.features=200; QC subset with nFeature_RNA 200–2500 and percent.mt < 5; NormalizeData; FindVariableFeatures with vst and nfeatures=2000; ScaleData; RunPCA; FindNeighbors on dims 1:10; FindClusters at resolution 0.5), then call FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25). Convert marker gene symbols to Entrez IDs with mapIds(org.Hs.eg.db, keytype='SYMBOL', multiVals='first'), build an input data frame with entrezid, cluster and score (avg_log2FC) sorted by descending expression score within each cluster — ranking matters because the article warns that simply overlapping gene lists without considering gene ranking can yield unreliable results. Run easyct() choosing db='cellmarker' (species='Human', tissue terms matched to your sample, p_cut=0.3) and one of four variants: test='GSEA' or test='fisher', each with hard or soft classification. Inspect candidate tissues via data(cellmarker_tissue), review the generated visualizations, and optionally compare against SingleR and scCATCH baselines; in the article the hard GSEA variant outperformed CellAssign, scCATCH, SCINA and SCSA at 4000 simulated PBMC cells and beat SCSA and scCATCH at 2400 and 1600 cells.

## Related tools

- **EasyCellType** (Core package that runs GSEA and modified Fisher's exact test of the marker list against cell marker databases and renders annotation visualizations) — https://github.com/rx-li/EasyCellType
- **Seurat** (Produces the clustered object and per-cluster positive markers (FindAllMarkers) that feed the annotation input)
- **org.Hs.eg.db** (AnnotationDbi mapIds source for converting marker gene SYMBOLs to Entrez IDs required by easyct())
- **clusterProfiler** (Supports the gene set enrichment analysis machinery used for annotation scoring)
- **scCATCH** (Baseline cluster-annotation tool (findmarkergene/findcelltype over CellMatch) used for comparison in benchmarks) — https://github.com/ZJUFanLab/scCATCH
- **SingleR** (Reference-based baseline annotator used to benchmark EasyCellType on the TNBC dataset)

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue=c("Blood","Peripheral blood","Blood vessel","Umbilical cord blood","Venous blood"), p_cut=0.3, test="GSEA")
```

## Evaluation signals

- Annotation of known PBMC clusters (e.g. CD4/CD8 T cells, B cells, monocytes) matches expected markers after easyct() with test='GSEA'
- Hard GSEA variant reproduces the benchmark result of highest accuracy over CellAssign, scCATCH, SCINA and SCSA on 4000-cell simulated PBMC data, exceeding SCSA and scCATCH at 2400 and 1600 cells
- Input data frame passes schema checks: entrezid/cluster/score columns present, scores sorted descending within each cluster
- On the TNBC dataset (GSE176078), EasyCellType labels CAF, T cell, PVL, epithelial and endothelial clusters correctly and performs comparably to SingleR and slightly better than scCATCH
- easyct() returns candidates within the p_cut=0.3 threshold and the chosen tissue subset (verified via data(cellmarker_tissue))

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results
- Only 3 publicly available marker gene databases are referenced, and the exact tissue database enumeration is not fully documented in the article
- Accuracy degrades at smaller dataset sizes: hard GSEA only exceeded SCSA and scCATCH (not all baselines) at 2400 and 1600 cells
- Shiny application provides the GUI route, but version numbers of the software are not reported in the article

## Evidence

- [intro] Two quantification approaches for cell type annotation: GSEA and a Fisher exact test combined with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [abstract] GSEA and modified Fisher's exact test: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [intro] Marker list input from Seurat: "automatically examine the input marker lists obtained from existing software such as Seurat over the cell marker databases"
- [methods] FindAllMarkers parameters: "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [methods] easyct call with cellmarker db: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [abstract] Ranking caveat: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [intro] Visualization rationale: "The package generates straight-forward visualization of the test results, which help the researchers to understand the results and pick the optimal annotations"
- [readme] scCATCH workflow from README: [evidence span withheld — anchor status: external_doc]
