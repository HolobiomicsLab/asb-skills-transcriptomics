---
name: alzheimer-disease-pathway-interpretation
description: Use when comparing scRNA-seq imputation methods on a disease dataset
  and you need biological evidence that one imputed matrix preserves or clarifies
  disease-relevant DEG patterns and KEGG pathways.
when_to_use_negative:
- Your goal is a quantitative clustering or imputation-error benchmark rather than
  biological pathway interpretation of DEGs.
- The dataset is not a disease-annotated scRNA-seq study (e.g. the AD/adam dataset);
  KEGG disease-pathway recovery will not be interpretable.
- You lack imputed variants of the same matrix — the skill requires side-by-side raw
  vs. imputed DEG comparisons.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: Provides the graph contrastive learning + ZINB autoencoder imputed AD expression
    matrix whose DEGs and KEGG enrichment are compared against raw and GraphSCI-imputed
    data
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Environment (Python 3.8.8) for loading h5/AnnData, running scanpy rank_genes_groups
    Wilcoxon tests, and KEGG enrichment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:cff88b3359341e183d32f9ca07310f78f42d1b91945c2e654fdd71a38618902e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# alzheimer-disease-pathway-interpretation

## Summary

Interpret imputation quality biologically by running Wilcoxon differential expression (scanpy rank_genes_groups) on raw, GraphSCI-imputed, and scGCL-imputed Alzheimer's disease (adam) expression matrices, then running KEGG pathway enrichment on each DEG list to see which imputation best recovers disease pathways.

## When to use

Use when comparing scRNA-seq imputation methods on a disease dataset and you need biological evidence that one imputed matrix preserves or clarifies disease-relevant DEG patterns and KEGG pathways.

## When NOT to use

- Your goal is a quantitative clustering or imputation-error benchmark rather than biological pathway interpretation of DEGs.
- The dataset is not a disease-annotated scRNA-seq study (e.g. the AD/adam dataset); KEGG disease-pathway recovery will not be interpretable.
- You lack imputed variants of the same matrix — the skill requires side-by-side raw vs. imputed DEG comparisons.

## Inputs

- adam h5 file containing the raw AD scRNA-seq expression matrix, true labels, and other information
- GraphSCI-imputed Alzheimer's disease expression matrix
- scGCL-imputed Alzheimer's disease expression matrix

## Outputs

- Combined DEG table for raw, GraphSCI-imputed, and scGCL-imputed conditions
- KEGG pathway enrichment results per DEG list, written to deg_kegg_enrichment.csv
- Enrichment figures comparable to Supplementary Figs S3–S5

## How to apply

Load the adam h5 file together with the GraphSCI- and scGCL-imputed AD matrices into scanpy AnnData objects. For each of the three expression conditions (raw, GraphSCI-imputed, scGCL-imputed), run scanpy's rank_genes_groups Wilcoxon rank-sum test to call differentially expressed genes. Collect the three DEG lists into a combined table, then run KEGG pathway enrichment on each list to identify over-represented pathways. Judge success by whether the scGCL-imputed condition most effectively surfaces disease pathways such as Alzheimer's disease and Huntington's disease relative to raw and GraphSCI-imputed data, and write results to deg_kegg_enrichment.csv with figures comparable to Supplementary Figs S3–S5.

## Related tools

- **scGCL** (Provides the graph contrastive learning + ZINB autoencoder imputed AD expression matrix whose DEGs and KEGG enrichment are compared against raw and GraphSCI-imputed data) — https://github.com/zehaoxiong123/scGCL
- **Python** (Environment (Python 3.8.8) for loading h5/AnnData, running scanpy rank_genes_groups Wilcoxon tests, and KEGG enrichment)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('adam.h5ad'); sc.tl.rank_genes_groups(adata, groupby='celltype', method='wilcoxon')
```

## Evaluation signals

- Wilcoxon DEG lists are successfully generated for all three conditions (raw, GraphSCI, scGCL) and merged into one comparable table.
- KEGG enrichment of scGCL-derived DEGs surfaces disease pathways such as Alzheimer's disease and Huntington's disease more clearly than raw or GraphSCI conditions.
- deg_kegg_enrichment.csv contains DEG and enrichment results for all three conditions.
- Resulting figures are comparable to Supplementary Figs S3–S5 of the paper.

## Limitations

- Evidence is limited to the Alzheimer's disease dataset; pathway-recovery superiority of scGCL is claimed specifically for that case study.
- The comparison is against raw and GraphSCI-imputed matrices; other imputation baselines are not part of this pathway-interpretation workflow.
- Requires the exact adam h5 file with expression values, true labels, and other information as documented in the scGCL README.

## Evidence

- [results] finding: KEGG enrichment surfaces disease pathways: [evidence span withheld — anchor status: not_found]
- [other] workflow: rank_genes_groups on three conditions: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: enhanced expression patterns in AD datasets: "scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
- [readme] readme: adam h5 file description: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [intro] intro: outperformance on real datasets: "we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
