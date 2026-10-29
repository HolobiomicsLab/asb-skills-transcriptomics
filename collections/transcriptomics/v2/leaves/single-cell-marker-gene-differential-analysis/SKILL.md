---
name: single-cell-marker-gene-differential-analysis
description: Use when a scRNA-seq dataset has already been normalized, dimensionality-reduced
  and clustered (e.g. SF clusters on PBMC3k), and per-cluster marker genes are needed
  for cluster annotation or functional enrichment.
when_to_use_negative:
- Clusters have not yet been defined — markers are computed per cluster, so clustering
  must precede this step
- The goal is scATAC-seq peak/differential accessibility analysis rather than gene-level
  scRNA-seq markers
- The dataset is not human or mouse, since SCALA currently supports analysis of only
  human and mouse datasets
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCALA
  role: Interactive Shiny/R pipeline exposing the Markers' Identification module that
    performs cluster-vs-rest marker testing
- name: Seurat
  role: R framework whose normalized/clustering objects underlie marker identification
    in the scRNA-seq workflow
  repo: https://github.com/scottgigante/seurat
- name: presto
  role: Fast Wilcoxon rank-sum test and AUC computation (wilcoxauc) enabling whole-genome
    marker discovery across hundred-thousand-cell matrices
  repo: https://github.com/immunogenomics/presto
- name: g:Profiler
  role: Downstream consumer of per-cluster up-regulated marker lists for GO biological
    process and KEGG enrichment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:e5672f8a2345cfe865df427537a3ab5c2509c0cc5566c90ad69dafaa7da980f9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/scottgigante/seurat
  tools:
  - Flame
  - R
  - SCALA
  - Seurat
  - g:Profiler
  - presto
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-marker-gene-differential-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies up- and down-regulated marker genes per cell cluster from a normalized scRNA-seq object using cluster-vs-rest differential testing (Wilcoxon-style), as implemented in SCALA via Seurat and the presto package, producing per-cluster gene lists that feed downstream functional enrichment with g:Profiler, FLAME and decoupleR.

## When to use

Use when a scRNA-seq dataset has already been normalized, dimensionality-reduced and clustered (e.g. SF clusters on PBMC3k), and per-cluster marker genes are needed for cluster annotation or functional enrichment.

## When NOT to use

- Clusters have not yet been defined — markers are computed per cluster, so clustering must precede this step
- The goal is scATAC-seq peak/differential accessibility analysis rather than gene-level scRNA-seq markers
- The dataset is not human or mouse, since SCALA currently supports analysis of only human and mouse datasets

## Inputs

- Normalized scRNA-seq expression matrix or Seurat object with cluster assignments (e.g. PBMC3k dataset)
- Cluster labels from the SCALA/Seurat clustering step (e.g. 10 SF clusters merged into 9 designated S1, S2a, S2b, S2c, S2d, S3, S4a, S4b, S5)

## Outputs

- Per-cluster up- and down-regulated marker gene lists with test statistics and significance values
- Marker gene table consumed by the functional enrichment step (input to g:Profiler GO/KEGG testing and FLAME)

## How to apply

After clustering (e.g. the SF clustering that produced 10 clusters in the PBMC3k/Seurat vignette workflow), run a cluster-versus-rest differential expression test on the normalized expression matrix, ranking genes per cluster by significance and log-fold change; SCALA exposes this through its 'Markers' Identification' module, which can leverage presto's fast Wilcoxon rank-sum implementation (wilcoxauc) for whole-genome-by-thousands-of-cells matrices. Keep both up- and down-regulated gene lists per cluster, since SCALA's functional enrichment layer consumes the per-cluster up-regulated marker genes as input for GO biological process enrichment with g:Profiler and deeper analysis with FLAME. Record, for each marker gene, the cluster of origin, test statistic, p-value/adjusted significance and the overlap with annotated terms so results can be exported and validated. Confirm the marker table exists for every input cluster before proceeding to enrichment or TF-activity scoring with decoupleR.

## Related tools

- **SCALA** (Interactive Shiny/R pipeline exposing the Markers' Identification module that performs cluster-vs-rest marker testing)
- **Seurat** (R framework whose normalized/clustering objects underlie marker identification in the scRNA-seq workflow) — https://github.com/scottgigante/seurat
- **presto** (Fast Wilcoxon rank-sum test and AUC computation (wilcoxauc) enabling whole-genome marker discovery across hundred-thousand-cell matrices) — https://github.com/immunogenomics/presto
- **g:Profiler** (Downstream consumer of per-cluster up-regulated marker lists for GO biological process and KEGG enrichment)

## Examples

```
library(presto); markers <- wilcoxauc(seurat_obj, group_by = 'seurat_clusters')
```

## Evaluation signals

- A marker table exists for every input cluster with gene, statistic and significance columns (no empty clusters)
- Cluster-vs-rest testing recapitulates expected biology, e.g. PBMC3k clusters expressing canonical cell-type markers consistent with the Seurat vignette used by SCALA
- Exported marker lists contain both up- and down-regulated genes with per-term overlap information available for the enrichment step
- Number of distinct marker sets matches the number of clusters produced by the clustering step (e.g. 10 SF clusters before merging)
- Downstream validation: functional_enrichment_results.csv contains per-cluster enrichment rows derived from these marker lists plus a decoupleR TF-activity score column for every input cluster

## Limitations

- SCALA supports analysis of only human and mouse datasets for both scRNA-seq and scATAC-seq modalities
- In the online SCALA version, dataset size is restricted to < 2 GB with no more than 2 CPU threads per session, and some steps cannot be completed for datasets larger than 50,000 cells; the desktop version is suggested for larger datasets
- Execution times and RAM consumption for different input sizes are only documented in Supplementary Table 2, not benchmarked in the paper

## Evidence

- [methods] Markers' Identification: [evidence span withheld — anchor status: not_found]
- [results] SF clustering produced 10 clusters: [evidence span withheld — anchor status: not_found]
- [full_text] Markers feed g:Profiler enrichment: [evidence span withheld — anchor status: not_found]
- [readme] presto Wilcoxon scale: [evidence span withheld — anchor status: not_found]
