---
name: single-cell-simulation-benchmark-evaluation
description: Use when comparing annotation accuracy of marker-database-based cell-type
  annotation methods on data with known ground-truth labels (e.g. a simulated PBMC
  dataset), or when deciding which marker database (CellMarker, PanglaoDB, Clustermole)
  to trust for easyct-based annotation.
when_to_use_negative:
- Input is real scRNA-seq data without known ground-truth cluster labels — accuracy
  cannot be scored against simulated cell-type labels
- The goal is annotating a new dataset rather than benchmarking — directly run easyct
  on the marker list instead of running the full six-run database comparison
- Simple list-overlap annotation is sufficient — note the article's warning that 'simply
  overlapping the two lists without considering gene ranking might lead to unreliable
  results', so ranked-input methods are preferred over naive overlap benchmarks
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: Benchmarked annotation package; easyct() runs GSEA and modified Fisher's exact
    test per database with plot_dot/plot_bar visualization
  repo: https://github.com/rx-li/EasyCellType
- name: clusterProfiler
  role: Performs the enrichment scoring underlying easyct's GSEA test option
- name: Seurat
  role: Generates the differential-expression marker lists (FindAllMarkers with only.pos=TRUE,
    min.pct=0.25, logfc.threshold=0.25) that feed the benchmark
- name: scCATCH
  role: Alternative marker-database-based annotation tool (CellMatch database) cited
    as a comparable automatic annotation approach
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:7a0e957cf93fbfcf66759a09c3e1a788243b68640b92295fefe75e7d9ce6f4e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Seurat
  - clusterProfiler
  - devtools
  - rTensor
  - scCATCH
  - scRank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-simulation-benchmarking
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# single-cell-simulation-benchmark-evaluation

## Summary

Benchmark cell-type annotation accuracy by running EasyCellType's easyct GSEA and modified Fisher's exact test against multiple marker databases (CellMarker, PanglaoDB, Clustermole) on a simulated 2400-cell PBMC dataset with known cluster-level ground-truth labels, then scoring per-database accuracy.

## When to use

Use when comparing annotation accuracy of marker-database-based cell-type annotation methods on data with known ground-truth labels (e.g. a simulated PBMC dataset), or when deciding which marker database (CellMarker, PanglaoDB, Clustermole) to trust for easyct-based annotation.

## When NOT to use

- Input is real scRNA-seq data without known ground-truth cluster labels — accuracy cannot be scored against simulated cell-type labels
- The goal is annotating a new dataset rather than benchmarking — directly run easyct on the marker list instead of running the full six-run database comparison
- Simple list-overlap annotation is sufficient — note the article's warning that 'simply overlapping the two lists without considering gene ranking might lead to unreliable results', so ranked-input methods are preferred over naive overlap benchmarks

## Inputs

- Simulated 2400-cell PBMC marker dataset with known cluster-level cell-type labels (Section 3.4 simulation design)
- EasyCellType input data frame: Entrez IDs, cluster labels, expression scores, sorted by score within each cluster
- Tissue keyword tables: cellmarker_tissue, panglao_tissue, clustermole_tissue (package data objects)
- Upstream Seurat markers from FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25) mapped to Entrez IDs via mapIds(org.Hs.eg.db, ...)

## Outputs

- annotation_accuracy_comparison.csv: accuracy per marker database per test (GSEA vs modified Fisher's exact test)
- Per-cluster annotation dot plots (plot_dot) and bar plots (plot_bar) for each database/test run
- Ranked database recommendation: CellMarker > Clustermole > PanglaoDB under both tests

## How to apply

Load or regenerate the simulated 2400-cell PBMC marker dataset per the paper's Section 3.4 design, and format it as an EasyCellType input data frame of Entrez IDs, cluster labels and expression scores, sorting genes by score within each cluster. Inspect candidate tissue keywords via data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue), and apply the Section 3.4 tissue filters for each database. Run easyct six times: with test='GSEA' and test='fisher' (clusterProfiler-backed enrichment and the modified Fisher's exact test), species='Human', p_cut=0.3, once per db option (cellmarker plus the corresponding PanglaoDB and Clustermole options). Summarize per-cluster recommendations with plot_dot and plot_bar, then score accuracy of each database × test combination against the known simulated cell-type labels and write annotation_accuracy_comparison.csv. Validation requires reproducing the paper's ordering: CellMarker highest and PanglaoDB lowest accuracy under both tests.

## Related tools

- **EasyCellType** (Benchmarked annotation package; easyct() runs GSEA and modified Fisher's exact test per database with plot_dot/plot_bar visualization) — https://github.com/rx-li/EasyCellType
- **clusterProfiler** (Performs the enrichment scoring underlying easyct's GSEA test option)
- **Seurat** (Generates the differential-expression marker lists (FindAllMarkers with only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) that feed the benchmark)
- **scCATCH** (Alternative marker-database-based annotation tool (CellMatch database) cited as a comparable automatic annotation approach) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", test="GSEA", p_cut=0.3)
```

## Evaluation signals

- annotation_accuracy_comparison.csv reproduces the paper's ordering: CellMarker attains the highest and PanglaoDB the lowest accuracy under both test='GSEA' and test='fisher'
- All six easyct runs use species='Human' and p_cut=0.3, with GSEA enrichment scored via clusterProfiler
- Input data frame uses Entrez IDs and is sorted by expression score within each cluster before calling easyct
- Tissue filters are applied per database after inspecting data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue)
- Accuracy is computed per database and per test against the known simulated cluster-level cell-type labels, not against a manual annotation

## Limitations

- The accuracy ordering is established on the Section 3.4 simulation design (2400-cell PBMC) and may not generalize to other tissues, cell-type compositions, or simulation parameters
- Naive gene-list overlap benchmarks without gene ranking are unreliable — the article notes 'simply overlapping the two lists without considering gene ranking might lead to unreliable results'
- The article does not fully enumerate the tissue database names (only '3 public available marker gene data bases') and omits software version numbers, complicating exact reproducibility
- Results depend on p_cut=0.3 and the applied tissue keyword filters; different filtering choices may change the database ranking

## Evidence

- [abstract] The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] Simply overlapping gene lists without considering gene ranking might lead to unreliable results: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [methods] markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25): "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [readme] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance.: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance."
