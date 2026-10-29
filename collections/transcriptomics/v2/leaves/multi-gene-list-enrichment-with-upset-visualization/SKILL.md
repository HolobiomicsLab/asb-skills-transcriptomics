---
name: multi-gene-list-enrichment-with-upset-visualization
description: Use when you have up to 10 per-cluster up/down regulated gene lists from
  scRNA-seq differential expression and need to compare enriched GO/KEGG terms across
  clusters interactively.
when_to_use_negative:
- Input is raw count matrices or unclustered cells — marker gene lists must already
  exist from the differential expression step
- More than 10 gene lists need to be compared at once, exceeding FLAME's supported
  limit
- The goal is single-gene-list enrichment only without cross-cluster comparison —
  per-cluster g:Profiler GO enrichment suffices
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: SCALA
  role: Shiny web application orchestrating the enrichment workflow downstream of
    marker gene identification in the scRNA-seq pipeline
- name: FLAME
  role: External application invoked by SCALA for deeper functional enrichment of
    one or up to 10 cluster gene lists with interactive UpSet plots and per-database
    bubble plots
- name: g:Profiler
  role: R package (gprofiler2) used for per-cluster GO biological processes and KEGG
    pathway enrichment preceding the multi-list comparison
- name: R
  role: Implementation language of SCALA and its Shiny interface
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:f6031dbbf94edad610f9e29422c65a5b182f97aa27c2f873ddc2c5bf0e5f3bce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - FLAME
  - Flame
  - R
  - SCALA
  - g:Profiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multi-gene-list-enrichment-with-upset-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform functional enrichment on multiple cluster gene lists simultaneously using SCALA's FLAME external application, visualizing shared and unique enriched terms across up to 10 clusters with interactive UpSet plots and a bubble plot summarizing enrichment per database.

## When to use

Use when you have up to 10 per-cluster up/down regulated gene lists from scRNA-seq differential expression and need to compare enriched GO/KEGG terms across clusters interactively.

## When NOT to use

- Input is raw count matrices or unclustered cells — marker gene lists must already exist from the differential expression step
- More than 10 gene lists need to be compared at once, exceeding FLAME's supported limit
- The goal is single-gene-list enrichment only without cross-cluster comparison — per-cluster g:Profiler GO enrichment suffices

## Inputs

- Per-cluster up/down regulated marker gene lists from scRNA-seq differential expression (e.g. PBMC3k clusters)
- Up to 10 cluster gene lists supplied simultaneously to FLAME

## Outputs

- Interactive UpSet plots showing shared/unique enriched terms across cluster gene lists
- Bubble plot summarizing enriched terms per database (GO, KEGG)
- Rows in functional_enrichment_results.csv with term, database, statistical significance, and gene overlap

## How to apply

After marker gene identification (cluster-vs-rest Wilcoxon-style testing, e.g. on PBMC3k), load the per-cluster up/down regulated gene lists and run deeper functional enrichment with the FLAME external application, supplying one gene list per cluster or up to 10 cluster gene lists simultaneously. Inspect the interactive UpSet plots to identify which enriched terms are shared or unique across cluster combinations, and use the generated bubble plot to summarize enriched terms per database (GO biological processes, KEGG). This complements the per-cluster g:Profiler GO enrichment step, which records per-term statistical significance and the overlap between the input gene list and each term. Results, including term, database, significance, and gene overlap, are exported to functional_enrichment_results.csv alongside decoupleR TF-activity scores.

## Related tools

- **SCALA** (Shiny web application orchestrating the enrichment workflow downstream of marker gene identification in the scRNA-seq pipeline)
- **FLAME** (External application invoked by SCALA for deeper functional enrichment of one or up to 10 cluster gene lists with interactive UpSet plots and per-database bubble plots)
- **g:Profiler** (R package (gprofiler2) used for per-cluster GO biological processes and KEGG pathway enrichment preceding the multi-list comparison)
- **R** (Implementation language of SCALA and its Shiny interface)

## Evaluation signals

- UpSet plots are rendered interactively for the supplied set of cluster gene lists (1–10 lists)
- A bubble plot is produced summarizing enriched terms grouped per database
- functional_enrichment_results.csv exists and contains per-cluster enrichment rows with term, database, significance, and gene-overlap fields
- Each input cluster's up/down gene list yields at least one tested enriched term with recorded statistical significance and overlap count
- The number of gene lists consumed matches the number of SF clusters from the upstream clustering step (e.g. 9 merged clusters S1–S5)

## Limitations

- FLAME handles at most 10 gene lists simultaneously, so pipelines with more clusters must be run in batches
- Online SCALA version restricts dataset size to < 2 GB and 2 CPU threads, and some steps cannot be completed for datasets larger than 50,000 cells — prefer the desktop version for large datasets
- Analysis is supported only for human and mouse datasets

## Evidence

- [methods] Run deeper functional enrichment with the FLAME external application, handling one gene list per cluster or up to 10 cluster gene lists simultaneously with interactive UpSet plots, and generate a bubble plot summarizing enriched terms per database.: [evidence span withheld — anchor status: not_found]
- [results] up-regulated marker genes identified for each SF cluster are fed as input into functional enrichment analysis, where GO biological processes enrichment is performed per SF cluster using g:Profiler.: [evidence span withheld — anchor status: not_found]
