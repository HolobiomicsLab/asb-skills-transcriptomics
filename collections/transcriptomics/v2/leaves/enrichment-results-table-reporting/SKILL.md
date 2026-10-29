---
name: enrichment-results-table-reporting
description: Use when per-cluster up/down-regulated marker gene lists (cluster-vs-rest
  Wilcoxon-style testing, e.g. PBMC3k) from SCALA's differential expression step are
  available and per-cluster GO/KEGG enrichment plus TF activity must be computed and
  exported as a table.
when_to_use_negative:
- The dataset is from an organism other than human or mouse — SCALA currently supports
  only human and mouse for both scRNA-seq and scATAC-seq
- The input is already an enrichment result table (e.g. a pre-computed term/significance
  table) rather than marker gene lists, so re-running g:Profiler/FLAME is redundant
- A non-single-cell bulk expression dataset is provided and cluster-vs-rest marker
  identification has not been performed
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: g:Profiler
  role: Tests each cluster's up/down regulated gene list for enriched GO biological
    process terms and KEGG pathways, recording per-term significance and gene overlap;
    provided in R via the gprofiler2 CRAN package
- name: Flame
  role: External application for deeper functional enrichment, handling one gene list
    per cluster or up to 10 cluster gene lists simultaneously with interactive UpSet
    plots and a per-database bubble plot
- name: decoupleR
  role: Scores each cluster against a curated TF-to-target-gene interaction resource
    to infer cluster-level TF activity
- name: SCALA
  role: Hosts the enrichment module downstream of marker gene identification in its
    Shiny/R scRNA-seq pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:38765cf6bb6f7b6c2fdb62f9ded295e47cc0f1a7591e0bcd11a1bcc8f4ab6a88
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - Flame
  - R
  - SCALA
  - decoupleR
  - g:Profiler
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# enrichment-results-table-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Produces and reports the per-cluster functional enrichment table downstream of marker-gene identification in SCALA's scRNA-seq pipeline: up/down-regulated marker genes per SF cluster are tested for GO biological process and KEGG enrichment with g:Profiler (gprofiler2), deeper enrichment is run with FLAME, and cluster-level TF activity is scored with decoupleR.

## When to use

Use when per-cluster up/down-regulated marker gene lists (cluster-vs-rest Wilcoxon-style testing, e.g. PBMC3k) from SCALA's differential expression step are available and per-cluster GO/KEGG enrichment plus TF activity must be computed and exported as a table.

## When NOT to use

- The dataset is from an organism other than human or mouse — SCALA currently supports only human and mouse for both scRNA-seq and scATAC-seq
- The input is already an enrichment result table (e.g. a pre-computed term/significance table) rather than marker gene lists, so re-running g:Profiler/FLAME is redundant
- A non-single-cell bulk expression dataset is provided and cluster-vs-rest marker identification has not been performed

## Inputs

- Per-cluster up/down-regulated marker gene lists from scRNA-seq cluster-vs-rest differential expression (e.g. PBMC3k clusters)
- Curated transcription-factor-to-target-gene interaction resource for decoupleR
- Gene lists for up to 10 clusters passed simultaneously to FLAME

## Outputs

- Per-term enrichment table with term, database, statistical significance, and gene overlap for GO biological processes and KEGG pathways
- Bubble plot summarizing enriched terms per database
- Interactive UpSet plots from FLAME for multi-cluster gene-list comparison
- Per-cluster decoupleR TF activity scores
- functional_enrichment_results.csv combining enrichment rows and TF activity per cluster

## How to apply

Load the per-cluster up/down regulated marker-gene lists produced by SCALA's differential expression step (e.g. on the PBMC3k dataset after SF clustering yielded 10 SF clusters). Test each cluster's gene list for enriched Gene Ontology (GO) biological process terms and KEGG pathways with the g:Profiler package (gprofiler2 is a required CRAN dependency of SCALA), recording per-term statistical significance and the overlap between the input gene list and each term. Run deeper functional enrichment with the FLAME external application — handling one gene list per cluster or up to 10 cluster gene lists simultaneously with interactive UpSet plots — and generate a bubble plot summarizing enriched terms per database. Infer cluster-level transcription factor activity with decoupleR, scoring each cluster against a curated TF-to-target-gene resource. Export the enriched terms (term, database, statistical significance, gene overlap) and per-cluster TF activity scores to functional_enrichment_results.csv, then validate the report schema.

## Related tools

- **g:Profiler** (Tests each cluster's up/down regulated gene list for enriched GO biological process terms and KEGG pathways, recording per-term significance and gene overlap; provided in R via the gprofiler2 CRAN package)
- **Flame** (External application for deeper functional enrichment, handling one gene list per cluster or up to 10 cluster gene lists simultaneously with interactive UpSet plots and a per-database bubble plot)
- **decoupleR** (Scores each cluster against a curated TF-to-target-gene interaction resource to infer cluster-level TF activity)
- **SCALA** (Hosts the enrichment module downstream of marker gene identification in its Shiny/R scRNA-seq pipeline)

## Examples

```
gost(cluster_markers, organism = "hsapiens", sources = c("GO:BP", "KEGG"))
```

## Evaluation signals

- functional_enrichment_results.csv exists and contains per-cluster enrichment rows with significance and gene-overlap fields
- The output CSV includes a decoupleR TF-activity score column for every input cluster
- Each enrichment row records the overlap between the input gene list and the term (non-empty intersection where enrichment is significant)
- Enriched terms are attributed to a database (GO biological process or KEGG) with per-term statistical significance values
- FLAME output covers the submitted cluster gene lists (1–10 lists) with an accompanying bubble plot of enriched terms per database

## Limitations

- Only human and mouse datasets are currently supported for both scRNA-seq and scATAC-seq
- The online SCALA version restricts dataset size to < 2 GB with no more than 2 CPU threads per session, and datasets > 50,000 cells may not complete all steps online (desktop version recommended)
- FLAME handles up to 10 cluster gene lists simultaneously, limiting multi-cluster comparisons to that count per run
- Benchmarking of execution times and RAM consumption is deferred to Supplementary Table 2 rather than reported in the paper

## Evidence

- [other] Export the enriched terms (term, database, statistical significance, gene overlap) and per-cluster TF activity scores to functional_enrichment_results.csv.: [evidence span withheld — anchor status: not_found]
- [other] confirm the output CSV exists and contains per-cluster enrichment rows with significance and gene-overlap fields plus a decoupleR TF-activity score column for every input cluster.: [evidence span withheld — anchor status: not_found]
- [intro] SCALA supports additional analysis modes such as automatic cluster annotation, functional enrichment analysis, ligand-receptor analysis, trajectory inference, and reconstruction of GRNs: "SCALA supports additional analysis modes such as automatic cluster annotation, functional enrichment analysis, ligand-receptor analysis, trajectory inference, and reconstruction of GRNs"
- [readme] gprofiler2: "- gprofiler2
- heatmaply
- Seurat
- SeuratObject"
- [readme] A web tool that handles the analysis of scRNAseq data, from quality control and normalization, to dimensionality reduction, differential expression analysis, clustering and visualization.: "A web tool that handles the analysis of scRNAseq data, from quality control and normalization, to dimensionality reduction, differential expression analysis, clustering and visualization."
