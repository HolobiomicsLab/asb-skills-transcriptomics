---
name: macrophage-subtype-differential-pathway-analysis
description: Use when novel/rare macrophage subclusters have been identified in a
  COVID-19 scRNA-seq dataset (e.g., via scNovel on GSE171524) and you need to determine
  which GO pathways are significantly up- or downregulated in COVID versus healthy
  samples.
when_to_use_negative:
- When no cluster-level differential expression results exist yet — the novel macrophage
  clusters (3, 5, 12) and their COVID-vs-healthy DE statistics must be computed first
  via the scNovel/Scanpy pipeline
- When the biological question is novel rare cell discovery itself rather than downstream
  pathway interpretation of already-identified subtypes
- When the dataset is not macrophage-focused or lacks a COVID vs. non-COVID contrast
  to define the ranked gene lists
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scNovel
  role: Identifies the three novel macrophage subtypes whose DEGs feed the GO/GSEA
    pathway analysis
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Loads and processes the GSE171524 single-cell data (normalize_total to 1e4,
    log1p) and handles differential gene expression results for the macrophage clusters
- name: Python
  role: Implementation language for the enrichment pipeline and writing gsea_results.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:fe0b1bf0fe4c4b68c3e2840c731344122359b3039352c7560775fed7d957d0ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# macrophage-subtype-differential-pathway-analysis

## Summary

Performs GO enrichment with gene set enrichment analysis on ranked differential-expression gene lists from COVID-19 versus non-COVID macrophage clusters (clusters 3, 5, 12 identified by high CD163 expression in GSE171524), identifying which biological pathways differ by COVID status.

## When to use

Use when novel/rare macrophage subclusters have been identified in a COVID-19 scRNA-seq dataset (e.g., via scNovel on GSE171524) and you need to determine which GO pathways are significantly up- or downregulated in COVID versus healthy samples.

## When NOT to use

- When no cluster-level differential expression results exist yet — the novel macrophage clusters (3, 5, 12) and their COVID-vs-healthy DE statistics must be computed first via the scNovel/Scanpy pipeline
- When the biological question is novel rare cell discovery itself rather than downstream pathway interpretation of already-identified subtypes
- When the dataset is not macrophage-focused or lacks a COVID vs. non-COVID contrast to define the ranked gene lists

## Inputs

- Differential gene expression results table for macrophage clusters 3, 5, and 12 (GSE171524 COVID-19 dataset)
- Per-cluster ranked gene lists (genes sorted by COVID vs. healthy differential expression statistic)
- AnnData/expression matrices processed with Scanpy (normalize_total to target_sum=1e4 followed by log1p)

## Outputs

- gsea_results.csv containing the full enrichment statistics table
- Figure 6I-style pathway enrichment visualization showing significant pathways with direction of change in COVID samples
- Filtered list of significantly enriched GO pathways per macrophage cluster with up/down direction in COVID

## How to apply

Load the differential gene expression results for the three novel macrophage clusters (clusters 3, 5, and 12, identified by high CD163 expression) from the GSE171524 COVID-19 dataset using Python with Scanpy. Rank the genes in each cluster by their differential expression statistic between COVID-19 and healthy control samples. Run Gene Ontology analysis supplemented by gene set enrichment analysis on each ranked gene list, then filter the GO/GSEA results for significantly enriched pathways and assign each a direction of change (upregulated vs. downregulated in COVID). Rationale: the scNovel analysis of the clinical COVID-19 dataset identified three novel macrophage subtypes whose COVID-related differential genes show consistent expression patterns, so pathway-level testing should be run per cluster on the ranked lists rather than on a pooled gene set.

## Related tools

- **scNovel** (Identifies the three novel macrophage subtypes whose DEGs feed the GO/GSEA pathway analysis) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Loads and processes the GSE171524 single-cell data (normalize_total to 1e4, log1p) and handles differential gene expression results for the macrophage clusters)
- **Python** (Implementation language for the enrichment pipeline and writing gsea_results.csv)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.tl.rank_genes_groups(adata, 'cluster', method='wilcoxon')
```

## Evaluation signals

- Reproduces the reported result: COVID samples show significant upregulation particularly in neutrophil activation related to immune response, neutrophil degranulation, and neutrophil-mediated immunity pathways
- Analysis covers all three novel macrophage clusters (3, 5, 12) with high CD163 expression, not just one
- gsea_results.csv is produced and contains enrichment statistics with direction-of-change (upregulated vs. downregulated) per pathway
- Each cluster's genes are ranked by the COVID-vs-healthy differential expression statistic before enrichment, not analyzed as an unordered set
- Only significantly enriched pathways pass the filter before visualization

## Limitations

- Depends on upstream scNovel cluster calls; the paper acknowledges that a better score function could improve scNovel performance, which would propagate to the cluster definitions
- Analysis is restricted to transcriptomic information; the method does not incorporate other biology information such as single-cell chromatin accessibility data
- No version numbers are given for scNovel or Python in the article, and no accession-level detail beyond the GSE171524 dataset is provided in the discussion section

## Evidence

- [abstract] Applying scNovel on a clinical COVID-19 dataset, three potential novel subtypes of Macrophages are identified: "Applying scNovel on a clinical COVID-19 dataset, three potential novel subtypes of Macrophages are identified"
- [discussion] Pipeline for novel rare cells and differential gene expression: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [abstract] Consistent expression patterns of COVID-related differential genes: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [readme] Scanpy compatibility in scRare requirements: "- Scanpy (Compatible with all versions)"
- [readme] Scanpy normalization parameters used in the tutorial: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
