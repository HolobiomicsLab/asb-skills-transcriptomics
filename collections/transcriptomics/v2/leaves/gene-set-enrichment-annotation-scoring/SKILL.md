---
name: gene-set-enrichment-annotation-scoring
description: Use when cluster-level marker tables (gene symbol, avg_log2FC per Seurat
  cluster) must be converted into per-cluster cell-type recommendations, and when
  comparing GSEA vs Fisher exact test accuracy across marker databases (CellMarker,
  PanglaoDB, Clustermole).
when_to_use_negative:
- Input is a raw count matrix with no cluster identities — the skill expects ranked
  cluster markers, not clustering itself
- You need a single-reference annotator with its own taxonomy database (e.g. scCATCH's
  CellMatch with 353 cell types / 686 subtypes) rather than EasyCellType's three-database
  comparison
- Markers are unranked gene lists from plain overlap analysis — the article warns
  that overlapping lists without gene ranking leads to unreliable results
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: EasyCellType
  role: Provides easyct() with test='GSEA' and test='fisher', db/species/tissue/p_cut
    parameters, plus plot_dot and plot_bar visualization of recommendations
  repo: https://github.com/rx-li/EasyCellType
- name: clusterProfiler
  role: Performs the underlying gene set enrichment analysis invoked by easyct(test='GSEA')
- name: Seurat
  role: Generates the ranked cluster marker lists (FindAllMarkers under DESeq2/Wilcoxon/t-test
    backends) that feed the GSEA annotation
- name: org.Hs.eg.db
  role: Maps Seurat gene symbols to Entrez IDs via mapIds(multiVals='first') required
    by easyct input
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:ff8f8ecc64b4f2ad60c517694e5a0a94a8ee15dad33b46d7922e681bcef97e52
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
  - gene set enrichment analysis
  - ggplot2
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-set-enrichment-annotation-scoring

## Summary

Score cell-type annotation by running EasyCellType's easyct() GSEA test (via clusterProfiler) over cluster-ranked, Entrez-mapped marker lists from Seurat DE analysis, then visualize and judge per-cluster recommendations with plot_dot/plot_bar.

## When to use

Use when cluster-level marker tables (gene symbol, avg_log2FC per Seurat cluster) must be converted into per-cluster cell-type recommendations, and when comparing GSEA vs Fisher exact test accuracy across marker databases (CellMarker, PanglaoDB, Clustermole).

## When NOT to use

- Input is a raw count matrix with no cluster identities — the skill expects ranked cluster markers, not clustering itself
- You need a single-reference annotator with its own taxonomy database (e.g. scCATCH's CellMatch with 353 cell types / 686 subtypes) rather than EasyCellType's three-database comparison
- Markers are unranked gene lists from plain overlap analysis — the article warns that overlapping lists without gene ranking leads to unreliable results

## Inputs

- Seurat marker table from FindAllMarkers (gene, cluster, avg_log2FC), only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25
- EasyCellType input data frame: Entrez IDs, cluster labels and expression scores, sorted by score within each cluster
- PBMC 3k count matrix (pbmc_data / gene_pbmc from the EasyCellType package, or the 10x pbmc3k filtered gene-barcode matrices)
- Tissue keyword tables: cellmarker_tissue, panglao_tissue, clustermole_tissue

## Outputs

- Per-cluster cell-type annotation recommendations from easyct() under test='GSEA' and test='fisher'
- Dot plots (plot_dot) and bar plots (plot_bar) of per-cluster annotations
- Accuracy comparison tables, e.g. annotation_accuracy.csv / annotation_accuracy_comparison.csv scored against known simulated labels

## How to apply

Process the count matrix through the standard Seurat workflow (CreateSeuratObject min.cells=3, min.features=200; subset nFeature_RNA 200–2500 and percent.mt<5; NormalizeData; FindVariableFeatures vst nfeatures=2000; ScaleData; RunPCA; FindNeighbors dims=1:10; FindClusters resolution=0.5), then run FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25). Convert gene symbols to Entrez IDs with mapIds(org.Hs.eg.db, multiVals='first') and rank genes per cluster by avg_log2FC descending — ranking matters because simple gene-list overlap without rank information leads to unreliable results. Call easyct() with test='GSEA' (enrichment scoring via clusterProfiler GSEA), species='Human', p_cut=0.3, db set per database ('cellmarker', PanglaoDB, or Clustermole options), and apply tissue filters (e.g. tissue=c('Blood','Peripheral blood','Blood vessel','Umbilical cord blood','Venous blood') for PBMC, inspecting candidate keywords via data(cellmarker_tissue)/data(panglao_tissue)/data(clustermole_tissue)). Parallel-run the same input with test='fisher' (modified Fisher's exact test) as a baseline, then render recommendations with plot_dot and plot_bar and score accuracy against known cluster labels. Rationale from the paper: GSEA-based annotation outperforms the Fisher's exact test when markers come from DESeq2, and the CellMarker database attains the highest accuracy, especially under GSEA.

## Related tools

- **EasyCellType** (Provides easyct() with test='GSEA' and test='fisher', db/species/tissue/p_cut parameters, plus plot_dot and plot_bar visualization of recommendations) — https://github.com/rx-li/EasyCellType
- **clusterProfiler** (Performs the underlying gene set enrichment analysis invoked by easyct(test='GSEA'))
- **Seurat** (Generates the ranked cluster marker lists (FindAllMarkers under DESeq2/Wilcoxon/t-test backends) that feed the GSEA annotation)
- **org.Hs.eg.db** (Maps Seurat gene symbols to Entrez IDs via mapIds(multiVals='first') required by easyct input)

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue=c('Blood','Peripheral blood','Blood vessel','Umbilical cord blood','Venous blood'), test="GSEA", p_cut=0.3)
```

## Evaluation signals

- GSEA-based annotations recover the intended simulated cell types and outperform the modified Fisher's exact test when markers are DESeq2-derived
- Across the three databases on the 2400-cell PBMC simulation, CellMarker achieves the highest and PanglaoDB the lowest annotation accuracy under both tests
- Each marker table is sorted by score within each cluster and uses Entrez IDs (not symbols) before easyct()
- t-test-derived markers yield the weakest accuracy; the comparison table must reflect this ordering
- Every run uses p_cut=0.3 and the PBMC tissue filter c('Blood','Peripheral blood','Blood vessel','Umbilical cord blood','Venous blood')

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results
- The t-test is not recommended for detecting DE genes; markers from a t-test backend degrade annotation accuracy
- Candidate tissue keywords must be inspected per database (cellmarker_tissue, panglao_tissue, clustermole_tissue) and filtered, or annotations will be diluted by irrelevant tissues
- The article does not enumerate all tissue database names beyond '3 public available marker gene data bases', and no software version numbers are reported

## Evidence

- [methods] annot.GSEA <- easyct(input.d, db="cellmarker", species="Human",: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [intro] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [abstract] Simply overlapping the two lists without considering gene ranking might lead to unreliable results.: [evidence span withheld — anchor status: not_found]
- [methods] markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25): "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
