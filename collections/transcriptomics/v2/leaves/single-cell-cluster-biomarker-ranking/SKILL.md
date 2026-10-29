---
name: single-cell-cluster-biomarker-ranking
description: Use when a scRNA-seq dataset has been clustered into cell subtypes (e.g.
  CD163-high macrophage clusters identified by scNovel) and you need pathway-level
  (GO/GSEA) interpretation of condition-dependent differential expression per cluster.
when_to_use_negative:
- The single-cell data have not yet been clustered or novel subtypes identified —
  scNovel's detection step (barcode preprocessing, query-stage thresholding) must
  precede this ranking step.
- The goal is cell-type annotation or novel-cell detection rather than interpreting
  cluster biology through pathway enrichment.
- You have only bulk RNA-seq or pre-aggregated gene signatures without per-cell cluster
  structure — the skill depends on cluster-resolved ranked gene lists.
edam_operation: http://edamontology.org/operation_3940
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Loads and preprocesses the single-cell data (sc.pp.normalize_total with target_sum=1e4,
    sc.pp.log1p), supports ranking cluster genes by differential expression statistic,
    and is the analysis backbone for the enrichment workflow
  repo: github.com/chuanyang-Zheng/scNovel
- name: scNovel
  role: 'Upstream step: identifies the novel macrophage subtypes (clusters 3, 5, 12
    in the COVID dataset) whose DE genes are then ranked and enriched; installed via
    pip install scNovel==1.1.0'
  repo: github.com/chuanyang-Zheng/scNovel
- name: Python
  role: Runtime language for the scNovel package and the Scanpy-based differential
    expression and enrichment pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:9fdd5db9c89e5f74b2da0c4c88e68c64e3e435860142704585d719c91143bd00
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3940
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-cluster-biomarker-ranking

## Summary

Ranks genes within single-cell RNA-seq clusters by their differential expression statistic between conditions (here COVID-19 vs healthy macrophage clusters 3, 5, and 12 from GSE171524) and converts the ranked lists into GO and gene set enrichment results that reveal which biological pathways distinguish COVID from non-COVID cells.

## When to use

Use when a scRNA-seq dataset has been clustered into cell subtypes (e.g. CD163-high macrophage clusters identified by scNovel) and you need pathway-level (GO/GSEA) interpretation of condition-dependent differential expression per cluster.

## When NOT to use

- The single-cell data have not yet been clustered or novel subtypes identified — scNovel's detection step (barcode preprocessing, query-stage thresholding) must precede this ranking step.
- The goal is cell-type annotation or novel-cell detection rather than interpreting cluster biology through pathway enrichment.
- You have only bulk RNA-seq or pre-aggregated gene signatures without per-cell cluster structure — the skill depends on cluster-resolved ranked gene lists.

## Inputs

- Differential gene expression results (per-cluster DE statistics) for the clusters of interest, e.g. COVID-19 vs healthy control statistics for macrophage clusters 3, 5, 12
- Single-cell count matrices for query and reference samples (CSV or AnnData/Scanpy-compatible format)
- Cluster labels (e.g. a label CSV in pandas structure, or cluster assignments such as CD163-high macrophage subtypes)
- Ranked gene lists per cluster (genes ordered by DE statistic)
- GO gene sets / pathway gene set collections for enrichment testing

## Outputs

- gsea_results.csv — full enrichment statistics table per pathway
- Figure 6I-style pathway enrichment visualization showing significant pathways and their direction of change in COVID samples
- Filtered list of significantly enriched GO pathways per cluster with upregulated/downregulated direction in COVID samples

## How to apply

Start from the differential gene expression results of the target clusters — in the scNovel COVID-19 case study these are the three novel macrophage clusters 3, 5, and 12, identified by high CD163 expression in the GSE171524 dataset. Preprocess the count matrices with Scanpy (sc.pp.normalize_total with target_sum=1e4, then sc.pp.log1p) before any ranking. Rank the genes in each cluster by their differential expression statistic between COVID-19 and healthy control samples, then run Gene Ontology analysis supplemented by gene set enrichment analysis on the ranked gene lists. Filter the GO/GSEA output for significantly enriched pathways and record each pathway's direction of change (upregulated vs downregulated) in COVID samples. Rationale: scNovel's authors pair novel rare-cell discovery with a downstream differential-expression pipeline, so the enrichment step validates that the discovered subtypes carry disease-relevant biology rather than annotation artifacts.

## Related tools

- **SCANPY** (Loads and preprocesses the single-cell data (sc.pp.normalize_total with target_sum=1e4, sc.pp.log1p), supports ranking cluster genes by differential expression statistic, and is the analysis backbone for the enrichment workflow) — github.com/chuanyang-Zheng/scNovel
- **scNovel** (Upstream step: identifies the novel macrophage subtypes (clusters 3, 5, 12 in the COVID dataset) whose DE genes are then ranked and enriched; installed via pip install scNovel==1.1.0) — github.com/chuanyang-Zheng/scNovel
- **Python** (Runtime language for the scNovel package and the Scanpy-based differential expression and enrichment pipeline)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(train_adata, target_sum=1e4); sc.pp.log1p(train_adata); sc.tl.rank_genes_groups(train_adata, 'cluster', method='wilcoxon')
```

## Evaluation signals

- Reproduces the reported result: COVID samples show significant upregulation particularly in neutrophil activation related to immune response, neutrophil degranulation, and neutrophil-mediated immunity pathways.
- Each significant pathway carries a direction-of-change call (upregulated vs downregulated in COVID samples), not just an enrichment score.
- gsea_results.csv contains full enrichment statistics (pathway, significance, direction) for all three macrophage clusters, not only top hits.
- Ranked gene lists are derived from clusters independently annotated as novel (CD163-high macrophage clusters 3, 5, 12), so enrichment signals track the correct cell population.
- Preprocessing invariants hold: expression matrices normalized with target_sum=1e4 and log1p-transformed before ranking.

## Limitations

- scNovel's authors note a better score function could improve performance, so the novel subtypes feeding this ranking may shift if detection scores change.
- The current pipeline does not incorporate additional biology such as single-cell chromatin accessibility data, limiting pathway interpretation to the transcriptome.
- No version numbers were specified for Python or the analysis dependencies in the article beyond pip install scNovel==1.1.0; version drift could affect enrichment reproducibility.
- The clinical COVID-19 dataset is profoundly imbalanced (few healthy controls vs many COVID cells), which can affect the stability of the DE statistics feeding the ranked lists.

## Evidence

- [abstract] three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [discussion] We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression.: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [readme] sc.pp.normalize_total(train_adata, target_sum=1e4): "sc.pp.normalize_total(train_adata, target_sum=1e4)"
- [results] GO analysis with gene set enrichment of the three macrophage clusters shows significant upregulation in COVID samples, particularly in neutrophil activation related to immune response, neutrophil degranulation, and neutrophil-mediated immunity.: [evidence span withheld — anchor status: not_found]
