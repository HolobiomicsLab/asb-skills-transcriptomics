---
name: qval-based-pathway-ranking
description: Use when comparing expression distributions of pathway gene sets across
  conditions (e.g. COVID vs healthy, 0/12/24 h T cell activation) and you need to
  rank which pathways are most perturbed.
when_to_use_negative:
- When the question is only gene-set over-representation of a fixed gene list (e.g.
  ORA of DE genes) rather than comparing pathway expression distributions across cell
  populations
- When input is bulk expression or an already-summarized enrichment table with no
  per-cell data — SCPA requires single-cell expression matrices or Seurat/SCE objects
- When fold change direction alone is sufficient and distributional shape change is
  not of interest
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCPA
  role: Runs compare_pathways()/compare_seurat() to compute the qval statistic ranking
    pathway distribution changes; qval is the primary output statistic
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Provides Seurat objects and SplitObject() to split by condition before extracting
    per-cell-type matrices with seurat_extract()
- name: msigdbr
  role: Pulls Hallmark pathway gene sets that format_pathways() converts into SCPA
    pathway input
- name: dplyr
  role: Joins and filters per-condition Pathway/qval columns into a single ranked
    table
- name: ggplot2
  role: Renders rank plots and supporting visualizations of qval-ranked pathway output
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:d69363cb3bef92b3757db31ac83b160ba1b896b747b6a5ac0d6ee73606132ef8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - R
  - SCPA
  - Seurat
  - dplyr
  - ggplot2
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# qval-based-pathway-ranking

## Summary

Rank single-cell pathway perturbations by the SCPA qval statistic — a distribution-free measure of change in a pathway's multivariate expression distribution — treating fold change/enrichment only as secondary information.

## When to use

Use when comparing expression distributions of pathway gene sets across conditions (e.g. COVID vs healthy, 0/12/24 h T cell activation) and you need to rank which pathways are most perturbed.

## When NOT to use

- When the question is only gene-set over-representation of a fixed gene list (e.g. ORA of DE genes) rather than comparing pathway expression distributions across cell populations
- When input is bulk expression or an already-summarized enrichment table with no per-cell data — SCPA requires single-cell expression matrices or Seurat/SCE objects
- When fold change direction alone is sufficient and distributional shape change is not of interest

## Inputs

- Per-population normalized expression matrices extracted via seurat_extract() from a Seurat object (e.g. covid_blood_atlas.rds, szabo_t_cell.rds, naive_cd4.rds) subset on metadata columns such as cell.type.fine, Status, or time point
- Pathway gene set file (e.g. combined_metabolic_pathways.csv or h_k_r_go_pid_reg_wik.csv, pathway name in column 1, genes in subsequent columns) or msigdbr Hallmark output formatted with format_pathways()

## Outputs

- Data frame with Pathway, pval, adjPval, and qval columns per pathway comparison
- Ranked qval table (e.g. scpa_pathway_qvals.csv, viral_pathway_sensitivity.csv) filtered to qval > 2
- Visualizations such as rank plots or ComplexHeatmap/plot_heatmap heatmaps of qvals per cell type or condition

## How to apply

Run SCPA's compare_pathways() on per-population expression matrices extracted with seurat_extract() (or compare_seurat()/compare_sce() directly on Seurat/SCE objects), supplying pathway gene sets from msigdbr formatted with format_pathways() or from the combined gene set file h_k_r_go_pid_reg_wik.csv (canonical, GO, and regulatory pathways from MSigDB). The output table contains Pathway, pval, adjPval, and qval columns; use the qval as the primary statistic for judging biological relevance, ranking pathways (e.g. interferon-response and metabolic Hallmark sets) by descending qval, and use pathway fold change/enrichment only as a secondary informative value. Filter to pathways with qval > 2 when summarizing perturbation breadth across conditions, as in the COVID-19 peripheral blood atlas benchmark where CD14+ monocytes showed the largest dysregulation. In multisample mode, compare_pathways() accepts lists of >2 samples (e.g. three activation time points) with downsampling to a default of 500 cells per population and optional parallel = TRUE with cores = x for speed. A large qval without matching fold change (e.g. arachidonic acid metabolism during T cell activation) is still biologically meaningful because SCPA detects distributional change independent of enrichment.

## Related tools

- **SCPA** (Runs compare_pathways()/compare_seurat() to compute the qval statistic ranking pathway distribution changes; qval is the primary output statistic) — https://github.com/jackbibby1/SCPA
- **Seurat** (Provides Seurat objects and SplitObject() to split by condition before extracting per-cell-type matrices with seurat_extract())
- **msigdbr** (Pulls Hallmark pathway gene sets that format_pathways() converts into SCPA pathway input)
- **dplyr** (Joins and filters per-condition Pathway/qval columns into a single ranked table)
- **ggplot2** (Renders rank plots and supporting visualizations of qval-ranked pathway output)

## Examples

```
scpa_out <- compare_pathways(samples = list(pop_0h, pop_12h, pop_24h), pathways = pathways, parallel = TRUE, cores = 4); scpa_out %>% arrange(desc(qval))
```

## Evaluation signals

- Output table contains Pathway and qval fields with ranked values, plus pval and adjPval
- In the COVID-19 benchmark, highly perturbed conditions such as CD14+ monocytes with large interferon/complement qvals rank at the top
- Pathways with qval > 2 in any comparison are retained when summarizing sensitivity
- Largest distribution changes fall in expected pathway classes (interferon-response and metabolic sets in the T cell activation analysis)
- Paths with high qval but low fold change (e.g. arachidonic acid metabolism) are still retained, confirming qval is used as the primary statistic

## Limitations

- compare_pathways() downsamples to a default of 500 cells per population, which may affect very small populations
- Min/max gene filters (min_genes, max_genes) restrict which pathways are tested
- Large qvals can occur without any fold change/enrichment, which may confuse users expecting concordant enrichment statistics

## Evidence
