---
name: modified-fisher-exact-test-annotation
description: Use when annotating clusters from scRNA-seq differential expression results
  (e.g. Seurat FindAllMarkers output) and you want a gene-overlap-based annotation
  alternative to GSEA that still respects expression significance rather than plain
  list overlap.
when_to_use_negative:
- When gene-ranking information is the priority or expected accuracy is highest —
  the paper reports GSEA with the CellMarker database attains the highest annotation
  accuracy, so prefer test='GSEA' for best performance
- When input is already a curated cell-type label per cluster and no marker-based
  scoring is needed
- When the marker input lacks expression scores/rankings, since plain list overlap
  without significance weighting is explicitly warned against
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EasyCellType
  role: Provides easyct(test='fisher'), the modified Fisher's exact test combining
    overlap with expression significance, plus plot_dot/plot_bar visualization
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Generates the input marker lists via FindAllMarkers(only.pos=TRUE, min.pct=0.25,
    logfc.threshold=0.25) on clustered PBMC data
- name: clusterProfiler
  role: Enrichment infrastructure used alongside the Fisher test path in the EasyCellType
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:ecacd18a36bd1c3a6ab3852e394408b034dcb4b6a9fbbb2b2e78f62d6bc6b149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Seurat
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Modified Fisher's exact test annotation

## Summary

EasyCellType's second annotation quantification approach: a Fisher's exact test that is modified by combining marker-list overlap with expression significance (gene ranking), run per cluster against CellMarker, PanglaoDB or Clustermole to recommend cell types.

## When to use

Use when annotating clusters from scRNA-seq differential expression results (e.g. Seurat FindAllMarkers output) and you want a gene-overlap-based annotation alternative to GSEA that still respects expression significance rather than plain list overlap.

## When NOT to use

- When gene-ranking information is the priority or expected accuracy is highest — the paper reports GSEA with the CellMarker database attains the highest annotation accuracy, so prefer test='GSEA' for best performance
- When input is already a curated cell-type label per cluster and no marker-based scoring is needed
- When the marker input lacks expression scores/rankings, since plain list overlap without significance weighting is explicitly warned against

## Inputs

- Marker list data frame of Entrez IDs, cluster labels and expression scores, sorted by score within each cluster (from Seurat FindAllMarkers mapped with mapIds/org.Hs.eg.db)
- Tissue keyword filter tables: cellmarker_tissue, panglao_tissue, clustermole_tissue (EasyCellType package data)

## Outputs

- Per-cluster cell-type annotation recommendations from easyct with test='fisher'
- Dot-plot and bar-plot summaries (plot_dot, plot_bar) of recommended annotations
- Optional annotation accuracy comparison table (e.g. annotation_accuracy_comparison.csv) when known labels are available

## How to apply

Format the marker input as a data frame of Entrez IDs, cluster labels and expression scores, sorting genes by score within each cluster (e.g. markers derived from Seurat's FindAllMarkers and mapped via mapIds(org.Hs.eg.db)). Run easyct with test='fisher', species='Human' and a permissive p_cut (0.3 in the paper's database comparison), once per marker database (db='cellmarker', plus the corresponding PanglaoDB and Clustermole options), applying tissue filters via the data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue) candidate keyword tables. The modification matters because the article warns that 'simply overlapping the two lists without considering gene ranking might lead to unreliable results' — the test combines Fisher overlap with expression significance. Summarize each run with plot_dot and plot_bar to inspect per-cluster annotation recommendations. For database-selection decisions, score accuracy against known labels and compare against the GSEA runs, since the paper found CellMarker with GSEA attains the highest accuracy (PanglaoDB the lowest) under both tests.

## Related tools

- **EasyCellType** (Provides easyct(test='fisher'), the modified Fisher's exact test combining overlap with expression significance, plus plot_dot/plot_bar visualization) — https://github.com/rx-li/EasyCellType
- **Seurat** (Generates the input marker lists via FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) on clustered PBMC data)
- **clusterProfiler** (Enrichment infrastructure used alongside the Fisher test path in the EasyCellType workflow)

## Examples

```
annot.fisher <- easyct(input.d, db="cellmarker", species="Human", test="fisher", p_cut=0.3)
```

## Evaluation signals

- easyct runs with test='fisher' return per-cluster annotation recommendations for every cluster without errors on Entrez-ID-formatted input
- With known labels, CellMarker attains higher and PanglaoDB the lowest annotation accuracy under the modified Fisher's exact test, matching the paper's reported ordering
- plot_dot and plot_bar outputs render per-cluster recommended cell types for each database run
- Tissue filters from data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue) are applied before scoring, restricting candidate cell types to the relevant tissue
- Results differ from plain gene-list overlap, confirming expression significance was incorporated in the test

## Limitations

- The paper reports that the GSEA approach outperforms the modified Fisher's exact test (CellMarker + GSEA gives the highest accuracy), so the Fisher path is the weaker of the two tests
- Database choice strongly affects accuracy; PanglaoDB was the lowest-performing of the three databases under both tests
- Software version numbers are not documented in the article's methods

## Evidence

- [intro] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [abstract] Simply overlapping gene lists without considering gene ranking might lead to unreliable results: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [readme] The package generates straight-forward visualization of the test results, which help the researchers to understand the results: "The package generates straight-forward visualization of the test results, which help the researchers to understand the results and pick the optimal annotations."
- [methods] The `EasyCellType` package was designed to examine an input marker list using the databases and provide annotation recommendations: "The `EasyCellType` package was designed to examine an input marker list using the databases and provide annotation recommendations in graphical outcomes."
