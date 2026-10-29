---
name: single-cell-pathway-distribution-testing
description: Use when comparing pathway activity between scRNA-seq conditions (e.g.
  resting vs 24h-activated naive CD4+ T cells) and you need distribution-level perturbation
  signals, not gene-set over-representation.
when_to_use_negative:
- Your question is strictly gene-set over-representation/enrichment testing, not multivariate
  distributional change in gene expression
- You have no per-cell expression data — SCPA requires total normalized count matrices,
  not precomputed bulk or aggregate profiles
- You need a two-sample-only statistic and rely on fold change as the primary readout;
  SCPA's fold change/enrichment is secondary and is not produced for multisample (>2)
  comparisons
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCPA
  role: Core R package providing seurat_extract/sce_extract to subset expression matrices
    by metadata and compare_pathways to run the graph-based nonparametric multisample
    distribution test
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Supplies the scRNA-seq object (e.g. naive_cd4.rds) whose metadata (Cell_Type,
    Hour) is used to split resting vs activated populations; SCPA auto-detects Seurat
    v5 layer objects
- name: crossmatch
  role: R dependency (pinned v1.3.1) underlying the graph-based distribution statistics
    used by SCPA
- name: multicross
  role: R dependency (pinned v2.1.0) underlying the multisample graph-based distribution
    statistics used by SCPA
- name: msigdbr
  role: Source of Hallmark and other pathway gene sets, formatted for SCPA with format_pathways
- name: tidyverse
  role: Data wrangling and joining of package-vs-standalone result tables (e.g. joining
    by Pathway for qval comparison)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:cec4e7b7adfa89e08ec3f032ce867c763b06cd2e9a367d9dec939e82b1180072
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - Seurat
  - crossmatch
  - dplyr
  - msigdbr
  - multicross
  - tidyverse
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# single-cell-pathway-distribution-testing

## Summary

Detect pathway perturbations in single-cell RNA-seq using SCPA, a graph-based nonparametric framework that defines pathway activity as a change in the multivariate distribution of a pathway's genes across two or more conditions, yielding pval/adjusted pval/qval statistics rather than enrichment scores.

## When to use

Use when comparing pathway activity between scRNA-seq conditions (e.g. resting vs 24h-activated naive CD4+ T cells) and you need distribution-level perturbation signals, not gene-set over-representation.

## When NOT to use

- Your question is strictly gene-set over-representation/enrichment testing, not multivariate distributional change in gene expression
- You have no per-cell expression data — SCPA requires total normalized count matrices, not precomputed bulk or aggregate profiles
- You need a two-sample-only statistic and rely on fold change as the primary readout; SCPA's fold change/enrichment is secondary and is not produced for multisample (>2) comparisons

## Inputs

- Total normalized count matrices from two or more conditions (extracted via seurat_extract from a Seurat object, or sce_extract from a SingleCellExperiment object)
- Seurat object, e.g. naive CD4+ T cell dataset (naive_cd4.rds), subsettable by metadata columns (Cell_Type, Hour, Status)
- Pathway gene sets: CSV (combined_metabolic_pathways.csv, h_k_r_go_pid_reg_wik.csv), gmt file(s), or msigdbr output formatted with format_pathways
- Optional parameters: downsample (default 500 cells), min_genes, max_genes, parallel, cores

## Outputs

- Per-pathway results table with Pathway, pval, adjusted pval, and qval statistics
- Fold change / enrichment score per pathway (two-sample comparisons only)
- Visualizations via plot_rank and plot_heatmap (rank plots, heatmaps of SCPA output)

## How to apply

Load the Seurat object (e.g. naive_cd4.rds) and curated pathway gene sets (e.g. combined_metabolic_pathways.csv combining canonical, GO, and regulatory pathways from MSigDB) with SCPA, Seurat, and tidyverse in R. Extract condition-specific expression matrices with seurat_extract, subsetting by metadata columns (e.g. meta1 = 'Cell_Type', meta2 = 'Hour'), then call compare_pathways with a list of sample matrices and the pathway table; SCPA builds pathway-specific matrices, downsamples to a default of 500 cells per population, and applies a graph-based multisample distribution test (underpinned by the crossmatch and multicross R packages). Report the qval as the primary statistic of biological relevance, and use the fold change/enrichment score (computed only in the two-sample case) as a secondary, informative value. Pathways with high qval but low fold change (e.g. arachidonic acid metabolism during T cell activation) are still meaningfully perturbed, since transcriptional change independent of enrichment is part of the signal. For standalone reimplementations, mirror this design and validate by correlating your Qvals against package output across the same pathways and inputs.

## Related tools

- **SCPA** (Core R package providing seurat_extract/sce_extract to subset expression matrices by metadata and compare_pathways to run the graph-based nonparametric multisample distribution test) — https://github.com/jackbibby1/SCPA
- **Seurat** (Supplies the scRNA-seq object (e.g. naive_cd4.rds) whose metadata (Cell_Type, Hour) is used to split resting vs activated populations; SCPA auto-detects Seurat v5 layer objects)
- **crossmatch** (R dependency (pinned v1.3.1) underlying the graph-based distribution statistics used by SCPA)
- **multicross** (R dependency (pinned v2.1.0) underlying the multisample graph-based distribution statistics used by SCPA)
- **msigdbr** (Source of Hallmark and other pathway gene sets, formatted for SCPA with format_pathways)
- **tidyverse** (Data wrangling and joining of package-vs-standalone result tables (e.g. joining by Pathway for qval comparison))

## Examples

```
scpa_out <- compare_pathways(samples = list(resting, activated), pathways = pathways)
```

## Evaluation signals

- The comparison runs without error on two-sample input (e.g. resting Hour 0 vs activated Hour 24 naive CD4+ T cells) and returns a table of Pathway, pval, adjusted pval, qval
- Standalone-reimplementation Qvals show strong agreement (high rank/linear correlation) with compare_pathways output across the same gene sets, e.g. written to qval_comparison.csv
- Downsampling behavior matches defaults (500 cells per population) when reproducing results
- Fold change/enrichment values are present for two-sample runs and the qval is treated as the primary statistic when judging biological relevance
- Known biological pattern is recovered: pathways such as arachidonic acid metabolism show large qvals without proportional fold change

## Limitations

- Fold change/enrichment scores are only produced for the two-sample case; multisample comparisons yield distribution statistics only
- Large qval with low fold change (e.g. arachidonic acid metabolism) must be interpreted as real perturbation, which can surprise users expecting enrichment-only logic
- Installation depends on pinned versions of crossmatch (1.3.1) and multicross (2.1.0); users may need to manually install dependencies if errors arise
- Interpretation guidance says the pathway fold change/enrichment should only be a secondary informative value, not a filtering statistic

## Evidence

- [readme] README: multivariate distribution testing rather than enrichment: [evidence span withheld — anchor status: external_doc]
- [readme] README: multisample testing across conditions/pseudotime: [evidence span withheld — anchor status: external_doc]
- [readme] High qval without enrichment is still meaningful: [evidence span withheld — anchor status: external_doc]
