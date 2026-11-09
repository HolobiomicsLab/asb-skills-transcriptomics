---
name: imputed-vs-raw-scrnaseq-comparison
description: Use when you have raw scRNA-seq counts plus one or more imputed versions
  of the same matrix (e.g. scGCL, GraphSCI h5/h5ad outputs) and want to judge imputation
  quality by DEG patterns and disease-pathway recovery.
when_to_use_negative:
- The goal is clustering- or gene-imputation-metric benchmarking across 14 datasets
  (the paper's main evaluation), not downstream DEG/pathway recovery on a disease
  dataset
- There is only one expression matrix with no raw counterpart to compare imputed versions
  against
- The genes cannot be mapped to KEGG identifiers, so pathway enrichment of the DEG
  lists cannot be computed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Produces the graph-contrastive-learning + ZINB-imputed AD matrix whose DEGs/pathways
    are compared against raw and GraphSCI-imputed data
  repo: https://github.com/zehaoxiong123/scGCL
- name: GraphSCI
  role: Baseline imputation method; its imputed AD matrix is the comparison arm against
    scGCL in the DEG/KEGG analysis
- name: Python
  role: Implementation environment for loading h5 matrices with scanpy, running rank_genes_groups
    Wilcoxon, and KEGG enrichment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:ed23188da915150b3ee129d320b1a55ca634a6f1f066745985de39c4b2bab158
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - GraphSCI
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# imputed-vs-raw-scrnaseq-comparison

## Summary

This skill benchmarks an scRNA-seq imputation method by running differential expression (scanpy Wilcoxon rank-sum) on raw vs. imputed expression matrices and comparing KEGG pathway enrichment of the DEGs, as done for scGCL versus GraphSCI on the Alzheimer's disease (adam) dataset.

## When to use

Use when you have raw scRNA-seq counts plus one or more imputed versions of the same matrix (e.g. scGCL, GraphSCI h5/h5ad outputs) and want to judge imputation quality by DEG patterns and disease-pathway recovery.

## When NOT to use

- The goal is clustering- or gene-imputation-metric benchmarking across 14 datasets (the paper's main evaluation), not downstream DEG/pathway recovery on a disease dataset
- There is only one expression matrix with no raw counterpart to compare imputed versions against
- The genes cannot be mapped to KEGG identifiers, so pathway enrichment of the DEG lists cannot be computed

## Inputs

- Raw scRNA-seq expression matrix in h5 format (e.g. the adam Alzheimer's disease h5 file) loadable with scanpy
- GraphSCI-imputed version of the AD expression matrix
- scGCL-imputed version of the AD expression matrix (e.g. produced by scGCL main.py)
- Cell type/group labels needed for the Wilcoxon rank-sum test

## Outputs

- DEG lists for raw, GraphSCI-imputed, and scGCL-imputed conditions (combined table)
- KEGG pathway enrichment results for each condition
- deg_kegg_enrichment.csv combining DEG and KEGG enrichment results for all three conditions
- KEGG enrichment figures comparable to Supplementary Figs S3–S5

## How to apply

Load the adam h5 file and the GraphSCI- and scGCL-imputed AD matrices into scanpy AnnData objects. For each of the three expression matrices (raw, GraphSCI-imputed, scGCL-imputed), run scanpy's rank_genes_groups with the Wilcoxon rank-sum test to call differentially expressed genes, then collect the three DEG lists into a combined table. Run KEGG pathway enrichment on each DEG list to identify over-represented pathways. The rationale is that a better imputation (scGCL, which integrates graph contrastive learning with a ZINB autoencoder) should yield clearer DEG patterns and recover disease-relevant pathways — in the paper, scGCL's DEGs most effectively surfaced Alzheimer's disease and Huntington's disease KEGG pathways. Write results to deg_kegg_enrichment.csv and produce enrichment figures comparable to Supplementary Figs S3–S5.

## Related tools

- **scGCL** (Produces the graph-contrastive-learning + ZINB-imputed AD matrix whose DEGs/pathways are compared against raw and GraphSCI-imputed data) — https://github.com/zehaoxiong123/scGCL
- **GraphSCI** (Baseline imputation method; its imputed AD matrix is the comparison arm against scGCL in the DEG/KEGG analysis)
- **Python** (Implementation environment for loading h5 matrices with scanpy, running rank_genes_groups Wilcoxon, and KEGG enrichment)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Three DEG lists (raw, GraphSCI-imputed, scGCL-imputed) are produced from the same adam dataset with the same scanpy rank_genes_groups Wilcoxon settings
- deg_kegg_enrichment.csv contains both DEG and KEGG enrichment columns for all three conditions
- KEGG enrichment of the scGCL-imputed DEGs recovers disease pathways such as Alzheimer's disease and Huntington's disease more effectively than the raw and GraphSCI arms
- Enrichment figures are directly comparable to Supplementary Figs S3–S5 (same three conditions, same pathway resources)

## Limitations

- The paper demonstrates this comparison only on the Alzheimer's disease (adam) dataset and against the GraphSCI baseline; behavior on other datasets or baselines is not established by this evidence
- The comparison inherits scGCL's environment requirements (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, faiss-cpu 1.7.3) when generating the imputed matrix
- Pathway recovery is judged only via KEGG enrichment of Wilcoxon DEGs, so pathway resources outside KEGG or other DEG tests are not covered

## Evidence

- [abstract] scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets: "scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
- [abstract] scGCL integrates graph contrastive learning and a ZINB distribution to estimate dropout values: "scGCL, which integrates graph contrastive learning and Zero-inflated Negative Binomial (ZINB) distribution to estimate dropout values"
- [readme] Run scGCL on the adam dataset from the command line: "$ python main.py --dataset adam --epochs 300"
- [readme] README confirms the AD finding in project documentation: "we find that scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets."
- [readme] Dataset argument is a h5 scRNA-seq expression file: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
