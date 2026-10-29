---
name: covid-patient-status-stratified-expression-comparison
description: Use when you have scNovel-detected (or otherwise annotated) cell clusters
  from a single-cell atlas with patient-level COVID/healthy status labels and need
  disease-status-stratified marker genes per cluster plus a cross-cluster consistent
  gene set.
when_to_use_negative:
- When the goal is novel rare cell detection/annotation itself — that is scNovel's
  query stage; this skill starts only after clusters have been detected.
- When comparing clusters against a reference cell-type atlas rather than comparing
  disease status within a fixed cluster — the stratification variable here is COVID
  vs healthy patient status.
- When patient metadata (healthy vs COVID labels) is missing, since rank_genes_groups
  cannot be stratified by status without it.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: Provides rank_genes_groups for COVID-vs-healthy differential expression within
    each Macrophage cluster, plus normalize_total/log1p preprocessing of the count
    matrix
  repo: github.com/chuanyang-Zheng/scNovel
- name: scNovel
  role: Upstream tool that detects the three novel Macrophage subtype clusters (3,
    5, 12) that define the strata for the differential analysis
  repo: github.com/chuanyang-Zheng/scNovel
- name: Python
  role: Implementation language for the differential expression pipeline built on
    the open-source scNovel/Scanpy stack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:56eb97189010757a30b821bd5b5a2d117a8257d6a54bb7f394e871d6c734ee86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# covid-patient-status-stratified-expression-comparison

## Summary

Stratifies differential gene expression by patient disease status (COVID vs healthy) within specific novel Macrophage subtype clusters, using Scanpy rank_genes_groups to find genes with consistent differential expression across dependent clusters (here clusters 3, 5, 12 from the GSE171524 bronchoalveolar atlas).

## When to use

Use when you have scNovel-detected (or otherwise annotated) cell clusters from a single-cell atlas with patient-level COVID/healthy status labels and need disease-status-stratified marker genes per cluster plus a cross-cluster consistent gene set.

## When NOT to use

- When the goal is novel rare cell detection/annotation itself — that is scNovel's query stage; this skill starts only after clusters have been detected.
- When comparing clusters against a reference cell-type atlas rather than comparing disease status within a fixed cluster — the stratification variable here is COVID vs healthy patient status.
- When patient metadata (healthy vs COVID labels) is missing, since rank_genes_groups cannot be stratified by status without it.

## Inputs

- AnnData / Scanpy object of scNovel-detected novel Macrophage clusters from GSE171524 (116,314 cells) with patient ID and COVID vs healthy status annotations
- Cluster labels identifying candidate novel Macrophage subtypes (clusters 3, 5, 12, defined by high CD163 expression)

## Outputs

- Per-cluster ranked gene lists from rank_genes_groups (top 8 biomarkers per cluster, Figure 6G style)
- Table of disease-status-differential genes consistent across clusters 3, 5, and 12 with expression direction labels: 7 up-regulated and 9 down-regulated genes (Figure 6H style)

## How to apply

Load the scNovel-detected novel cell clusters from the GSE171524 bronchoalveolar immune cell atlas (116,314 cells; 7 healthy controls, 20 COVID-19 samples) into Scanpy AnnData, retaining patient and COVID-status annotations, and preprocess with sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p as in the README tutorial. Select the candidate Macrophage subtype clusters (clusters 3, 5, and 12, identified by high CD163 expression) and run sc.tl.rank_genes_groups within each cluster, comparing COVID vs healthy samples and taking the top biomarkers per cluster (top 8 per cluster, as in Figure 6G). Then intersect/keep the genes satisfying the differential criteria in all three dependent clusters to reproduce the volcano-style gene set of Figure 6H. Finally compile a table of up-regulated (ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1) and down-regulated (ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC) genes with expression directions.

## Related tools

- **SCANPY** (Provides rank_genes_groups for COVID-vs-healthy differential expression within each Macrophage cluster, plus normalize_total/log1p preprocessing of the count matrix) — github.com/chuanyang-Zheng/scNovel
- **scNovel** (Upstream tool that detects the three novel Macrophage subtype clusters (3, 5, 12) that define the strata for the differential analysis) — github.com/chuanyang-Zheng/scNovel
- **Python** (Implementation language for the differential expression pipeline built on the open-source scNovel/Scanpy stack)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.tl.rank_genes_groups(adata[adata.obs['cluster'].isin([3,5,12])], groupby='covid_status', method='wilcoxon')
```

## Evaluation signals

- The final gene table contains exactly 7 genes labeled up-regulated: ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1 (contains_substring checks on gene names).
- The table contains exactly 9 genes labeled down-regulated: ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC.
- Every output gene has a field_present direction label (up-regulated vs down-regulated) matching the reported Figure 6H sets.
- The analysis was restricted to clusters 3, 5, and 12 (CD163-high Macrophage clusters) and rank_genes_groups returned top-8 biomarkers per cluster.
- Retained cells total 116,314 with the 7 healthy / 20 COVID-19 sample composition of GSE171524.

## Limitations

- The differential gene sets are specific to the scNovel-detected Macrophage clusters in the COVID-19 clinical dataset; other cell types or datasets require re-running the full detection-then-DE pipeline.
- Consistency across the three dependent clusters is used as the inclusion criterion, so genes differential in only one cluster are excluded even if strongly significant.
- The article acknowledges that a better score function could improve scNovel performance, which would change the cluster composition feeding this analysis.

## Evidence

- [abstract] three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [intro] We further apply scNovel on a clinical COVID-19 dataset, discover three potential novel subtypes of Macrophages and conduct a deeper analysis of the disease-related genes’ expression pattern: "We further apply scNovel on a clinical COVID-19 dataset, discover three potential novel subtypes of Macrophages and conduct a deeper analysis of the disease-related genes’ expression pattern"
- [discussion] We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression.: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [readme] sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata): "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
