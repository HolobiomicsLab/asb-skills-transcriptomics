---
name: covid-19-differential-gene-interpretation
description: Use when novel cell subtypes (e.g. the three CD163-high macrophage clusters
  3, 5, 12 from the GSE171524 COVID-19 dataset) have been identified and you need
  to know which GO biological pathways are significantly up- or downregulated in disease
  vs. control samples.
when_to_use_negative:
- The goal is discovering the novel rare cell subtypes themselves rather than interpreting
  their differential genes — use scNovel's novel rare cell detection instead.
- Input is an unannotated single-cell matrix without prior cluster-level differential
  expression statistics — the skill operates on DE results of already-identified clusters.
- The comparison is not between disease and control conditions (e.g. no COVID vs.
  non-COVID contrast), so directional pathway interpretation is not defined.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: Identifies the three novel macrophage subtypes whose COVID-related differential
    genes are subsequently analyzed for consistent expression patterns
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Loads and preprocesses (normalize_total target_sum=1e4, log1p) the single-cell
    data and supports differential expression analysis of the macrophage clusters
- name: Python
  role: Language in which the enrichment pipeline, ranked gene lists, and gsea_results.csv
    output are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:fdfab2626159f0c99bbc4ede05a0fbc10678d7bf5207fc84a62bbc1ae7c35d18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# covid-19-differential-gene-interpretation

## Summary

Interprets differential expression of COVID-associated genes in scNovel-discovered novel macrophage subtypes by ranking cluster-level DE statistics and running GO enrichment with gene set enrichment analysis (GSEA) to identify directionally shifted pathways between COVID and non-COVID samples.

## When to use

Use when novel cell subtypes (e.g. the three CD163-high macrophage clusters 3, 5, 12 from the GSE171524 COVID-19 dataset) have been identified and you need to know which GO biological pathways are significantly up- or downregulated in disease vs. control samples.

## When NOT to use

- The goal is discovering the novel rare cell subtypes themselves rather than interpreting their differential genes — use scNovel's novel rare cell detection instead.
- Input is an unannotated single-cell matrix without prior cluster-level differential expression statistics — the skill operates on DE results of already-identified clusters.
- The comparison is not between disease and control conditions (e.g. no COVID vs. non-COVID contrast), so directional pathway interpretation is not defined.

## Inputs

- Differential gene expression results (DE statistics) for macrophage clusters 3, 5, and 12 from the GSE171524 COVID-19 dataset
- Ranked gene lists per macrophage cluster ordered by COVID-vs-control differential expression statistic
- Single-cell count matrices processed with Scanpy normalize_total(target_sum=1e4) and log1p

## Outputs

- Filtered list of significantly enriched GO pathways with direction of change (upregulated vs. downregulated) in COVID samples
- gsea_results.csv containing the full enrichment statistics table
- Figure 6I-style pathway enrichment visualization

## How to apply

Load the differential gene expression results for the three novel macrophage clusters (clusters 3, 5, and 12, identified by high CD163 expression) from the GSE171524 COVID-19 dataset using Python with Scanpy, after applying Scanpy's normalize_total (target_sum=1e4) and log1p preprocessing as in the scNovel pipeline. Rank the genes in each macrophage cluster by their differential expression statistic between COVID-19 and healthy control samples. Conduct Gene Ontology (GO) analysis supplemented by gene set enrichment analysis on the ranked gene lists, then filter the GO/GSEA results for significantly enriched pathways and assign each a direction of change (upregulated vs. downregulated) in COVID samples. The rationale is that scNovel's deeper analysis showed COVID-related differential genes have consistent expression patterns, so pathway-level enrichment of cluster DEGs reveals the functional program distinguishing the novel macrophage subtypes.

## Related tools

- **scNovel** (Identifies the three novel macrophage subtypes whose COVID-related differential genes are subsequently analyzed for consistent expression patterns) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Loads and preprocesses (normalize_total target_sum=1e4, log1p) the single-cell data and supports differential expression analysis of the macrophage clusters)
- **Python** (Language in which the enrichment pipeline, ranked gene lists, and gsea_results.csv output are implemented)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.tl.rank_genes_groups(adata, 'covid_status', method='wilcoxon')
```

## Evaluation signals

- The enrichment output reproduces the reported result that COVID samples show significant upregulation in neutrophil activation related to immune response, neutrophil degranulation, and neutrophil-mediated immunity pathways.
- gsea_results.csv is written and contains full enrichment statistics for each tested GO pathway.
- Each significant pathway carries an assigned direction of change (upregulated vs. downregulated) in COVID samples.
- Input clusters correspond to the three novel subtypes (clusters 3, 5, 12) marked by high CD163 expression in the GSE171524 dataset.
- The ranked gene lists used for GSEA reflect per-cluster COVID-vs-control differential expression statistics, not raw expression.

## Limitations

- Pathway interpretation depends on scNovel's preliminary finding; scNovel's authors note that a better score function could improve novel cell detection performance, which would propagate to downstream DEG interpretation.
- The current method does not incorporate other biology information such as single-cell chromatin accessibility data, so regulatory interpretation of the enriched pathways is limited.
- No version numbers for scNovel or Python are specified in the article context, so reproducibility of the enrichment environment must be pinned separately.

## Evidence

- [results] GO enrichment of three macrophage clusters: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [discussion] GSE171524 clusters: "Within the context of the profoundly imbalanced clinical COVID-19 dataset, a comprehensive analysis has unveiled the existence of three promising novel subtypes of macrophages."
- [readme] Scanpy preprocessing: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [discussion] DE pipeline: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
