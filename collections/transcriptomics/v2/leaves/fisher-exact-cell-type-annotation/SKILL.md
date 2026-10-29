---
name: fisher-exact-cell-type-annotation
description: Use when annotating Seurat-derived cluster marker lists (positive markers
  with avg_log2FC scores, Entrez-mapped) against CellMarker via easyct(test='fisher'),
  or when comparing Fisher vs GSEA annotation accuracy.
when_to_use_negative:
- When the analysis goal is maximum annotation accuracy on larger simulated PBMC datasets
  — the GSEA variant is preferred (hard GSEA outperformed all four unsupervised tools
  at 4000 cells, while Fisher exceeded only SCSA and scCATCH at 2400/1600 cells)
- When markers come from a t-test DE backend — the paper's DE-test comparison found
  t-test markers give the weakest accuracy and t-test is not recommended for detecting
  DE genes
- When you need organism or tissue coverage outside CellMarker's human blood/tissue
  references; check available db/tissue options first
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: EasyCellType
  role: Provides easyct(test='fisher'), the modified Fisher's exact test classifier
    combined with expression significance, plus plot_dot/plot_bar visualization
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Generates the input marker list via FindAllMarkers(only.pos=TRUE, min.pct=0.25,
    logfc.threshold=0.25) after clustering
- name: AnnotationDbi
  role: mapIds with org.Hs.eg.db converts marker gene symbols to Entrez IDs (multiVals='first')
    required by easyct
- name: clusterProfiler
  role: Underpins the enrichment testing machinery; GSEA alternative backend to the
    Fisher test
- name: scCATCH
  role: Baseline unsupervised annotation tool used in the accuracy comparison against
    the Fisher classifier
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:c842a1dd4426c55d861037d42d398443b46d2875ff0a84bc3d1dafea02d4598f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - AnnotationDbi
  - EasyCellType
  - Seurat
  - clusterProfiler
  - dplyr
  - ggplot2
  - org.Hs.eg.db
  - scCATCH
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# fisher-exact-cell-type-annotation

## Summary

Annotate scRNA-seq clusters with EasyCellType's modified Fisher's exact test classifier, which combines gene-set overlap against CellMarker with expression significance (avg_log2FC ranking) rather than raw list overlap alone.

## When to use

Use when annotating Seurat-derived cluster marker lists (positive markers with avg_log2FC scores, Entrez-mapped) against CellMarker via easyct(test='fisher'), or when comparing Fisher vs GSEA annotation accuracy.

## When NOT to use

- When the analysis goal is maximum annotation accuracy on larger simulated PBMC datasets — the GSEA variant is preferred (hard GSEA outperformed all four unsupervised tools at 4000 cells, while Fisher exceeded only SCSA and scCATCH at 2400/1600 cells)
- When markers come from a t-test DE backend — the paper's DE-test comparison found t-test markers give the weakest accuracy and t-test is not recommended for detecting DE genes
- When you need organism or tissue coverage outside CellMarker's human blood/tissue references; check available db/tissue options first

## Inputs

- Data frame of positive cluster markers with columns entrezid, cluster, score (avg_log2FC), sorted by descending score per cluster (e.g. from FindAllMarkers on a Seurat object)
- Seurat object or PBMC 3k filtered gene-barcode matrix (pbmc3k from cf.10xgenomics.com or pbmc_data shipped in the EasyCellType package)
- org.Hs.eg.db annotation for SYMBOL-to-Entrez mapping

## Outputs

- Per-cluster cell-type annotation recommendations from easyct(test='fisher') against CellMarker
- Dot plots and bar plots via plot_dot/plot_bar summarizing per-cluster test results
- Annotation accuracy table (e.g. annotation_accuracy.csv) when benchmarked against simulated ground-truth labels

## How to apply

Run the standard Seurat PBMC workflow (CreateSeuratObject min.cells=3/min.features=200, percent.mt<5 via '^MT-', nFeature_RNA 200–2500, NormalizeData, vst 2000 variable features, PCA, FindNeighbors dims=1:10, FindClusters resolution=0.5), then FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25). Map gene symbols to Entrez IDs with mapIds(org.Hs.eg.db, multiVals='first'), rank by descending avg_log2FC within each cluster, and call easyct(input.d, db='cellmarker', species='Human', tissue=c('Blood','Peripheral blood','Blood vessel','Umbilical cord blood','Venous blood'), p_cut=0.3, test='fisher'). The modified Fisher's exact test differs from naive list overlap by incorporating expression significance (gene ranking), which the paper flags as critical since 'simply overlapping the two lists without considering gene ranking might lead to unreliable results'. Render results with plot_dot and plot_bar, and score soft/hard classification accuracy against known labels when benchmarking (task_001: Fisher variant trails the GSEA variant, which outperforms CellAssign, scCATCH, SCINA and SCSA at 4000 simulated cells).

## Related tools

- **EasyCellType** (Provides easyct(test='fisher'), the modified Fisher's exact test classifier combined with expression significance, plus plot_dot/plot_bar visualization) — https://github.com/rx-li/EasyCellType
- **Seurat** (Generates the input marker list via FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) after clustering)
- **AnnotationDbi** (mapIds with org.Hs.eg.db converts marker gene symbols to Entrez IDs (multiVals='first') required by easyct)
- **clusterProfiler** (Underpins the enrichment testing machinery; GSEA alternative backend to the Fisher test)
- **scCATCH** (Baseline unsupervised annotation tool used in the accuracy comparison against the Fisher classifier) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
annot <- easyct(input.d, db="cellmarker", species="Human", tissue=c('Blood','Peripheral blood','Blood vessel','Umbilical cord blood','Venous blood'), p_cut=0.3, test="fisher")
```

## Evaluation signals

- easyct runs without error on the entrezid/cluster/score input frame and returns per-cluster recommendations for db='cellmarker', p_cut=0.3
- All marker genes in the input have non-NA Entrez IDs after mapIds(multiVals='first')
- Reproduced accuracy comparison shows the GSEA variant outperforming the Fisher variant (per task_001: hard GSEA beats CellAssign, scCATCH, SCINA, SCSA at 4000 cells; Fisher exceeds only SCSA and scCATCH at 2400/1600)
- Per the DE-test comparison (task_006), Fisher accuracy is weakest when markers come from t-test DE detection, and GSEA beats Fisher when markers come from DESeq2
- plot_dot/plot_bar render recommendations for every Seurat cluster produced by FindClusters(resolution=0.5)

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results — the Fisher test must combine overlap with expression significance
- The hard Fisher classifier underperforms the hard GSEA classifier against CellAssign, scCATCH, SCINA and SCSA at 4000 simulated PBMC cells
- The article names '3 public available marker gene data bases' without fully enumerating tissue database coverage; software version numbers are not documented

## Evidence

- [abstract] The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher's exact test: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] simply overlapping the two lists without considering gene ranking might lead to unreliable results: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [intro] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [methods] FindAllMarkers with only.pos, min.pct, logfc.threshold parameters: "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [methods] easyct cellmarker Human call from the article methods: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [readme] scCATCH functions findmarkergene and findcelltype: [evidence span withheld — anchor status: external_doc]
