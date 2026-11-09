---
name: single-cell-differential-gene-expression-testing
description: 'Use when comparing scRNA-seq imputation methods on biological interpretability:
  you have raw and imputed expression matrices for the same cells and want to test
  whether imputation yields clearer DEG patterns and better recovery of disease-related
  KEGG pathways.'
when_to_use_negative:
- You only have a single expression matrix with no imputed counterpart to compare
  against — the skill's value comes from the three-condition (raw vs GraphSCI vs scGCL)
  comparison.
- The goal is clustering or imputation-accuracy benchmarking (e.g. on the 14 scRNA-seq
  datasets) rather than downstream differential-expression/pathway interpretability.
- Pathway databases other than KEGG are required — the workflow and the paper's findings
  are framed specifically around KEGG pathway enrichment.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGCL
  role: Provides the graph contrastive learning + ZINB autoencoder imputed expression
    matrix that is one of the three conditions tested for DEG/KEGG pathway recovery
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Environment for running scanpy (AnnData, rank_genes_groups Wilcoxon) and the
    KEGG enrichment steps; reference version 3.8.8
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:2948822f57c6143e855ed54f059d775cbb0e8204fe922b823a1530d3f0595a20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-differential-gene-expression-testing

## Summary

Runs scanpy's Wilcoxon rank-sum test (rank_genes_groups) on raw and imputed scRNA-seq expression matrices (e.g. the adam Alzheimer's disease h5 and GraphSCI/scGCL-imputed versions) to call DEGs, then performs KEGG pathway enrichment on each DEG list to judge which imputation recovers disease pathways most clearly.

## When to use

Use when comparing scRNA-seq imputation methods on biological interpretability: you have raw and imputed expression matrices for the same cells and want to test whether imputation yields clearer DEG patterns and better recovery of disease-related KEGG pathways.

## When NOT to use

- You only have a single expression matrix with no imputed counterpart to compare against — the skill's value comes from the three-condition (raw vs GraphSCI vs scGCL) comparison.
- The goal is clustering or imputation-accuracy benchmarking (e.g. on the 14 scRNA-seq datasets) rather than downstream differential-expression/pathway interpretability.
- Pathway databases other than KEGG are required — the workflow and the paper's findings are framed specifically around KEGG pathway enrichment.

## Inputs

- adam h5 file containing the raw scRNA-seq expression matrix (with true labels and other information)
- GraphSCI-imputed AD expression matrix
- scGCL-imputed AD expression matrix

## Outputs

- deg_kegg_enrichment.csv with DEG lists and KEGG enrichment results for raw, GraphSCI-imputed, and scGCL-imputed conditions
- KEGG pathway enrichment figures comparable to Supplementary Figs S3–S5

## How to apply

Load the adam h5 file and the GraphSCI- and scGCL-imputed AD matrices into scanpy AnnData objects so all three share a comparable gene/cell structure. For each of the three expression matrices, run sc.tl.rank_genes_groups with the Wilcoxon rank-sum test to call differentially expressed genes, then collect the three DEG lists into a combined table. Run KEGG pathway enrichment independently on each DEG list to identify over-represented pathways. The rationale (per the scGCL paper) is that a better imputation method should surface disease pathways such as Alzheimer's disease and Huntington's disease more effectively in the DEG-derived enrichment. Write the DEG and KEGG results for all three conditions to deg_kegg_enrichment.csv and produce enrichment figures comparable to Supplementary Figs S3–S5; scGCL-imputed data is expected to most effectively discover the potential disease pathways.

## Related tools

- **scGCL** (Provides the graph contrastive learning + ZINB autoencoder imputed expression matrix that is one of the three conditions tested for DEG/KEGG pathway recovery) — https://github.com/zehaoxiong123/scGCL
- **Python** (Environment for running scanpy (AnnData, rank_genes_groups Wilcoxon) and the KEGG enrichment steps; reference version 3.8.8)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- All three conditions (raw, GraphSCI-imputed, scGCL-imputed) yield DEG lists from scanpy Wilcoxon rank_genes_groups before enrichment.
- KEGG enrichment on scGCL-imputed DEGs recovers Alzheimer's disease and Huntington's disease pathways, matching the paper's finding.
- deg_kegg_enrichment.csv contains DEG and KEGG enrichment results for all three conditions in one combined table.
- Enrichment figures are directly comparable to Supplementary Figs S3–S5.

## Limitations

- Findings are demonstrated on the Alzheimer's disease (adam) dataset; pathway-recovery superiority of scGCL is claimed for this case study and 14 datasets for clustering/imputation metrics, not universally for all DEG/pathway analyses.
- Comparison requires the same imputation baseline (GraphSCI) and the scGCL imputed matrix produced with matching parameters (e.g. --dataset adam --epochs 300).

## Evidence

- [methods] Wilcoxon DEG workflow: [evidence span withheld — anchor status: not_found]
- [results] KEGG finding: [evidence span withheld — anchor status: not_found]
- [abstract] AD expression enhancement: "scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
- [readme] CLI invocation: "python main.py --dataset adam --epochs 300"
