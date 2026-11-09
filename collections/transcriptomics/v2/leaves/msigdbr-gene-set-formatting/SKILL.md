---
name: msigdbr-gene-set-formatting
description: 'Use when preparing pathway inputs for compare_pathways()/compare_seurat()
  SCPA analyses: you have a Seurat/SCE object or expression matrices and need MSigDB-derived
  gene sets formatted as pathway-name-plus-genes structures.'
when_to_use_negative:
- Your gene sets already come as SCPA-formatted CSVs from the jackbibby1/SCPA gene_sets
  repository or a prior run — no reformatting needed
- You are doing over-representation/enrichment analysis (e.g., standard GSEA) rather
  than SCPA multivariate distribution testing — format_pathways targets SCPA's expected
  input
- You need non-Homo sapiens gene sets — the workflows shown here pull msigdbr Homo
  sapiens output only
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: msigdbr
  role: Queries MSigDB (Hallmark, KEGG, Reactome, GO, PID, WikiPathways) within R
    as the raw source of pathway gene sets before formatting
  repo: https://www.gsea-msigdb.org/gsea/msigdb
- name: SCPA
  role: Provides format_pathways() to reshape msigdbr output into the pathway/gene
    structure consumed by compare_pathways(), plus get_paths() for gmt inputs
  repo: https://github.com/jackbibby1/SCPA
- name: tidyverse
  role: Used with grepl/dplyr filtering of msigdbr gs_subcat columns to select pathway
    collections (e.g., 'kegg', 'reactome', 'pid')
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:d7290e5f1a5d85e01a1093a020ccdac7f755fe034681a6f0176bd8b1b74a3452
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SCPA
  - dplyr
  - ggplot2
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# msigdbr-gene-set-formatting

## Summary

Defines pathway gene sets for SCPA single-cell pathway analysis by pulling MSigDB collections with the msigdbr R package (e.g., Hallmark, KEGG, Reactome subsets) and normalizing them with format_pathways, or by loading pre-built gene set CSV files distributed with the SCPA package.

## When to use

Use when preparing pathway inputs for compare_pathways()/compare_seurat() SCPA analyses: you have a Seurat/SCE object or expression matrices and need MSigDB-derived gene sets formatted as pathway-name-plus-genes structures.

## When NOT to use

- Your gene sets already come as SCPA-formatted CSVs from the jackbibby1/SCPA gene_sets repository or a prior run — no reformatting needed
- You are doing over-representation/enrichment analysis (e.g., standard GSEA) rather than SCPA multivariate distribution testing — format_pathways targets SCPA's expected input
- You need non-Homo sapiens gene sets — the workflows shown here pull msigdbr Homo sapiens output only

## Inputs

- msigdbr Homo sapiens query output (data frame with gs_name, gene_symbol, gs_subcat columns)
- Gene set CSV files (pathway name in column 1, genes in subsequent columns), e.g. combined_metabolic_pathways.csv or h_k_r_go_pid_reg_wik.csv
- gmt files (one or multiple) loadable via get_paths()

## Outputs

- Formatted pathway list compatible with compare_pathways()/compare_seurat() (pathway name plus associated genes)
- Ranked SCPA result table downstream (Pathway, pval, adjPval, qval columns) once compared

## How to apply

Load msigdbr alongside SCPA (library(SCPA); library(msigdbr)) and query Homo sapiens gene sets, filtering by collection or gs_subcat — e.g., grep for 'kegg', 'reactome', 'biocarta', 'wiki', 'pid' plus all HALLMARK sets to build a ~3,000-pathway panel, or take category H for Hallmark only. Pass the msigdbr data frame to format_pathways(), which reshapes it into the pathway/gene structure SCPA expects; alternatively read a pre-built CSV such as combined_metabolic_pathways.csv (pathway name in column 1, genes in subsequent columns), h_k_r_go_pid_reg_wik.csv, or files from the SCPA GitHub gene_sets directory. The rationale is that SCPA's compare_pathways() requires pathway definitions in a consistent format regardless of whether they come from msigdbr, gmt files, or custom gene sets, and min_genes/max_genes thresholds in compare_pathways (defaults enforced downstream) assume this structure. Feed the formatted object directly as the pathways argument to compare_pathways(samples, pathways, ...).

## Related tools

- **msigdbr** (Queries MSigDB (Hallmark, KEGG, Reactome, GO, PID, WikiPathways) within R as the raw source of pathway gene sets before formatting) — https://www.gsea-msigdb.org/gsea/msigdb
- **SCPA** (Provides format_pathways() to reshape msigdbr output into the pathway/gene structure consumed by compare_pathways(), plus get_paths() for gmt inputs) — https://github.com/jackbibby1/SCPA
- **tidyverse** (Used with grepl/dplyr filtering of msigdbr gs_subcat columns to select pathway collections (e.g., 'kegg', 'reactome', 'pid'))

## Examples

```
pathways <- format_pathways(msigdbr(species = "Homo sapiens", category = "H"))
```

## Evaluation signals

- format_pathways() returns a structure with a pathway-name column and gene lists/rows consumable by compare_pathways() without errors
- Filtered Hallmark-plus-subcategory panel contains roughly ~3,000 gene sets when grepl-filtering kegg/reactome/biocarta/wiki/pid plus all HALLMARK sets, per the tissue-comparison workflow
- Each pathway retains gene membership (compare_pathways enforces min_genes/max_genes; pathways outside bounds are dropped)
- Downstream compare_pathways output table has Pathway, pval, adjPval, qval columns with qval interpretable as the primary statistic
- Custom CSV inputs parse such that pathway name is in column 1 with genes in subsequent columns

## Limitations

- Gene set files such as combined_metabolic_pathways.csv and h_k_r_go_pid_reg_wik.csv are distributed at github.com/jackbibby1/SCPA/tree/main/gene_sets and must match the expected column layout; a malformed CSV silently changes pathway membership
- compare_pathways applies min_genes/max_genes filters and a default downsample of 500 cells per population, so very small or very large gene sets may be excluded regardless of formatting
- Version differences in msigdbr/MSigDB releases can shift gene set membership and thus qval rankings between runs

## Evidence

- [methods] Pre-built gene sets hosted in the repository: [evidence span withheld — anchor status: not_found]
