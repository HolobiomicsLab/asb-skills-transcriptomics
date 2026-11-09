---
name: novel-cell-cluster-biomarker-selection
description: Use when a novel/rare cell cluster discovery step (e.g., scNovel) has
  produced candidate clusters and you need disease-vs-control differential genes within
  those clusters, or consistent biomarkers across related novel subtypes.
when_to_use_negative:
- When the clusters being tested are known, annotated cell types in a reference set
  rather than novel rare-cell candidates — standard marker analysis suffices
- When the dataset lacks matched disease/condition labels and patient metadata, since
  the test is a between-condition comparison within each cluster
- When the goal is to discover the novel clusters themselves rather than characterize
  them — use scNovel's detection modes first
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scNovel
  role: Upstream tool that detects the novel rare Macrophage subtype clusters whose
    biomarkers are then analyzed
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Provides rank_genes_groups for per-cluster differential expression between
    COVID and healthy cells and general data handling (normalize_total, log1p)
- name: Python
  role: Implementation language for the scNovel package and the Scanpy-based differential
    expression pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:bbc8b73e1a6d4e7cfb3d390a2a27bcc98b8dfef59bdff39919aee9e4be2ea705
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# novel-cell-cluster-biomarker-selection

## Summary

Differential gene expression analysis between disease and control conditions within novel rare-cell clusters (e.g., scNovel-detected Macrophage subtypes), selecting biomarkers with consistent differential expression across dependent clusters to characterize newly discovered cell subtypes.

## When to use

Use when a novel/rare cell cluster discovery step (e.g., scNovel) has produced candidate clusters and you need disease-vs-control differential genes within those clusters, or consistent biomarkers across related novel subtypes.

## When NOT to use

- When the clusters being tested are known, annotated cell types in a reference set rather than novel rare-cell candidates — standard marker analysis suffices
- When the dataset lacks matched disease/condition labels and patient metadata, since the test is a between-condition comparison within each cluster
- When the goal is to discover the novel clusters themselves rather than characterize them — use scNovel's detection modes first

## Inputs

- Annotated single-cell RNA-seq AnnData object (e.g., GSE171524, 116,314 bronchoalveolar immune cells) with patient ID and COVID/healthy status in obs
- Cluster labels from novel rare cell detection (clusters 3, 5, 12; Macrophage subtype candidates identified by CD163 expression)
- Condition annotation (COVID vs healthy sample labels)

## Outputs

- Per-cluster ranked biomarker lists from rank_genes_groups (e.g., top 8 genes per cluster)
- Intersection table of genes consistently differentially expressed across all three novel Macrophage clusters with up/down-regulation direction labels
- Volcano-style gene set (Figure 6H): 7 up-regulated and 9 down-regulated genes

## How to apply

Load the annotated single-cell atlas (e.g., GSE171524 bronchoalveolar immune cells, 116,314 cells from 7 healthy controls and 20 COVID-19 samples) into Scanpy, retaining patient and condition annotations. Select the dependent novel clusters of interest — here the three Macrophage subtype clusters (3, 5, and 12) identified by high CD163 expression. Run differential expression with Scanpy's rank_genes_groups between COVID and healthy cells within each cluster (e.g., top 8 biomarkers per cluster as in Figure 6G). Intersect the per-cluster biomarker lists to keep genes with consistent differential expression across all three dependent clusters, reproducing the volcano-style gene set of Figure 6H. Compile a final table of direction labels: expect 7 up-regulated genes (ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1) and 9 down-regulated genes (ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC).

## Related tools

- **scNovel** (Upstream tool that detects the novel rare Macrophage subtype clusters whose biomarkers are then analyzed) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Provides rank_genes_groups for per-cluster differential expression between COVID and healthy cells and general data handling (normalize_total, log1p))
- **Python** (Implementation language for the scNovel package and the Scanpy-based differential expression pipeline)

## Examples

```
sc.tl.rank_genes_groups(adata[adata.obs['cluster'].isin(['3','5','12'])], groupby='COVID_status')
```

## Evaluation signals

- Output table contains exactly 7 genes labeled up-regulated (ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1)
- Output table contains exactly 9 genes labeled down-regulated (ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC)
- Every reported gene appears as a top biomarker (e.g., top 8) in all three Macrophage clusters (3, 5, 12), confirming consistent expression patterns
- Each gene row carries a valid direction label (up-regulated or down-regulated) via field_present checks
- Cluster selection is validated by high CD163 expression in the candidate Macrophage clusters

## Limitations

- Consistency filtering across three dependent clusters may discard condition-relevant genes that are differential in only a subset of the novel subtypes
- Only top-ranked biomarkers per cluster (top 8 in Figure 6G) are considered, so weaker but significant DE genes can be missed
- Upstream results depend on scNovel's score function, which the authors note could be improved ('a better score function could improve scNovel performance')

## Evidence

- [abstract] three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [discussion] We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression.: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [intro] We further apply scNovel on a clinical COVID-19 dataset, discover three potential novel subtypes of Macrophages and conduct a deeper analysis of the disease-related genes’ expression pattern: "We further apply scNovel on a clinical COVID-19 dataset, discover three potential novel subtypes of Macrophages and conduct a deeper analysis of the disease-related genes’ expression pattern"
- [readme] sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata): "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
