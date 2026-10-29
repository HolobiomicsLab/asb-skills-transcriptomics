---
name: kegg-pathway-enrichment-analysis
description: Use when raw and imputed versions of an scRNA-seq dataset (e.g., the
  adam h5 AD matrix vs GraphSCI/scGCL-imputed versions) must be compared at the DEG/pathway
  level rather than by clustering metrics.
when_to_use_negative:
- The goal is clustering or gene-imputation accuracy benchmarking rather than DEG/pathway-level
  comparison of imputation methods.
- The input is already an enrichment result or a pathway-level table rather than raw/imputed
  single-cell expression matrices.
- The dataset's gene identifiers cannot be mapped to KEGG entries, so no over-representation
  test is possible.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Loads the adam h5 and imputed matrices as AnnData and calls DEGs via rank_genes_groups
    Wilcoxon rank-sum test
- name: scGCL
  role: Produces the imputed AD expression matrix whose DEG KEGG enrichment is compared
    against raw and GraphSCI-imputed data
  repo: https://github.com/zehaoxiong123/scGCL
- name: GraphSCI
  role: Baseline imputation method whose imputed AD matrix provides the comparison
    DEG list
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:9dd9c60718582a86d511eebe470fe5b6c5e167fb46a4bf3a2935f27d318adecf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - GraphSCI
  - Python
  - scGCL
  - scanpy
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kegg-pathway-enrichment-analysis

## Summary

Calls Wilcoxon differential expression genes (scanpy rank_genes_groups) on the raw, GraphSCI-imputed, and scGCL-imputed Alzheimer's disease (adam) scRNA-seq matrices and runs KEGG pathway enrichment on each DEG list to judge which imputation best recovers disease pathways.

## When to use

Use when raw and imputed versions of an scRNA-seq dataset (e.g., the adam h5 AD matrix vs GraphSCI/scGCL-imputed versions) must be compared at the DEG/pathway level rather than by clustering metrics.

## When NOT to use

- The goal is clustering or gene-imputation accuracy benchmarking rather than DEG/pathway-level comparison of imputation methods.
- The input is already an enrichment result or a pathway-level table rather than raw/imputed single-cell expression matrices.
- The dataset's gene identifiers cannot be mapped to KEGG entries, so no over-representation test is possible.

## Inputs

- adam h5 file (scRNA-seq expression matrix with true labels, Alzheimer's disease dataset)
- GraphSCI-imputed AD expression matrix
- scGCL-imputed AD expression matrix
- Gene identifiers mappable to KEGG pathway entries

## Outputs

- Per-condition DEG lists (raw, GraphSCI-imputed, scGCL-imputed) from scanpy Wilcoxon rank_genes_groups
- Combined DEG table across the three conditions
- KEGG pathway enrichment results per DEG list
- deg_kegg_enrichment.csv containing DEG and KEGG enrichment results for all three conditions
- Enrichment figures comparable to Supplementary Figs S3-S5

## How to apply

Load the adam h5 file together with the GraphSCI- and scGCL-imputed AD expression matrices into scanpy AnnData objects. For each of the three matrices, run sc.tl.rank_genes_groups with the Wilcoxon rank-sum test to call differentially expressed genes, then pool the three DEG lists into a combined table. Run KEGG pathway enrichment independently on each DEG list to identify over-represented KEGG pathways. Write the DEG and KEGG enrichment results for all three conditions to deg_kegg_enrichment.csv and produce enrichment figures comparable to Supplementary Figs S3-S5. The comparison is correct if the scGCL-imputed DEGs most effectively recover disease pathways such as Alzheimer's disease and Huntington's disease relative to the raw and GraphSCI-imputed DEGs.

## Related tools

- **scanpy** (Loads the adam h5 and imputed matrices as AnnData and calls DEGs via rank_genes_groups Wilcoxon rank-sum test)
- **scGCL** (Produces the imputed AD expression matrix whose DEG KEGG enrichment is compared against raw and GraphSCI-imputed data) — https://github.com/zehaoxiong123/scGCL
- **GraphSCI** (Baseline imputation method whose imputed AD matrix provides the comparison DEG list)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby, method='wilcoxon'); sc.get.rank_genes_groups_df(adata, group=groupby).to_csv('degs_raw.csv')
```

## Evaluation signals

- deg_kegg_enrichment.csv contains DEG and KEGG enrichment rows for all three conditions (raw, GraphSCI-imputed, scGCL-imputed).
- Wilcoxon DEG calls are non-empty for each condition and generated with the same scanpy rank_genes_groups settings.
- KEGG enrichment of scGCL-imputed DEGs recovers disease pathways such as Alzheimer's disease and Huntington's disease more effectively than the other two conditions.
- Enrichment figures are directly comparable to Supplementary Figs S3-S5.
- scGCL was run with the documented settings (e.g., --dataset adam --epochs 300) so the imputed matrix matches the published setup.

## Limitations

- The pathway-level conclusion is demonstrated on the Alzheimer's disease (adam) dataset; it is a comparative claim, not a general guarantee of pathway recovery.
- scGCL depends on a pinned stack (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-cluster 1.6.0, torch-scatter 2.0.9, torch-sparse 0.6.13, faiss-cpu 1.7.3), so the imputed matrix may not reproduce without these versions.
- Enrichment results depend on the imputed matrix being regenerated (or supplied) exactly as the published imputation outputs; the h5 must contain expression values and true labels.

## Evidence

- [abstract] scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets: "scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
- [methods] Run scGCL from the command line: "$ python main.py --dataset adam --epochs 300"
- [results] Imputed matrices used for comparison: [evidence span withheld — anchor status: not_found]
- [readme] Dependencies for reproducing the imputed matrices: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
