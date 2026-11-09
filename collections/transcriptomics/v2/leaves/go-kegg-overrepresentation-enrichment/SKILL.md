---
name: go-kegg-overrepresentation-enrichment
description: Use when a scRNA-seq clustering pipeline has produced per-cluster marker
  gene lists (e.g. cluster-vs-rest Wilcoxon-style DE on PBMC3k) and you need to know
  which GO biological processes and KEGG pathways are enriched in each cluster.
when_to_use_negative:
- The input is raw count matrices or unclustered cells — enrichment requires previously
  identified clusters and their marker gene lists (markers identification must precede
  this step)
- The dataset is not human or mouse — SCALA currently supports analysis of only human
  and mouse datasets
- You need gene-set ranking (GSEA-style) on a full ranked gene list rather than overrepresentation
  testing on a discrete marker list
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: g:Profiler (gprofiler2)
  role: Performs GO biological process and KEGG pathway overrepresentation testing
    on each cluster's marker gene list; installed as a CRAN dependency of SCALA
- name: FLAME
  role: External application for deeper functional enrichment; accepts one gene list
    per cluster or up to 10 cluster gene lists simultaneously, with interactive UpSet
    plots and a per-database bubble plot
- name: decoupleR
  role: Complementary per-cluster TF activity inference scoring each cluster against
    a curated transcription-factor-to-target-gene resource
- name: SCALA
  role: Shiny web/desktop application that orchestrates the enrichment module downstream
    of marker identification in the scRNA-seq pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:3088a1cdf723a35db43a3746fc069bb803b18b04c7a3f03f43fb849543d27d5b
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
  - decoupleR
  - g:Profiler
  - g:Profiler (gprofiler2)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# GO/KEGG overrepresentation enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs overrepresentation testing of per-cluster up/down-regulated marker gene lists against Gene Ontology (biological process) and KEGG pathway terms, as implemented in SCALA's functional enrichment layer via g:Profiler, with deeper enrichment handled by the FLAME external application.

## When to use

Use when a scRNA-seq clustering pipeline has produced per-cluster marker gene lists (e.g. cluster-vs-rest Wilcoxon-style DE on PBMC3k) and you need to know which GO biological processes and KEGG pathways are enriched in each cluster.

## When NOT to use

- The input is raw count matrices or unclustered cells — enrichment requires previously identified clusters and their marker gene lists (markers identification must precede this step)
- The dataset is not human or mouse — SCALA currently supports analysis of only human and mouse datasets
- You need gene-set ranking (GSEA-style) on a full ranked gene list rather than overrepresentation testing on a discrete marker list

## Inputs

- Per-cluster up/down-regulated marker gene lists (one gene list per cluster, or up to 10 lists for simultaneous FLAME analysis)
- Cluster partition from the scRNA-seq pipeline (e.g. Seurat SF clusters on human or mouse data)

## Outputs

- Table of enriched GO/KEGG terms per cluster with term name, database, statistical significance, and gene overlap (functional_enrichment_results.csv)
- UpSet plots and a bubble plot summarizing enriched terms per database (from FLAME)
- Per-cluster transcription factor activity scores from decoupleR (optional complement)

## How to apply

Start from the up/down-regulated marker gene lists produced by the differential expression step (cluster-vs-rest testing after Seurat-based clustering, e.g. on the PBMC3k dataset). For each SF cluster (e.g. the S1–S5 clusters SCALA derived from 10 SF clusters on the hTNFtg synovial fibroblast data), test the marker list for enriched GO terms and KEGG pathways with g:Profiler (the gprofiler2 R package bundled with SCALA), recording per-term statistical significance and the overlap between the input gene list and each term. For deeper investigation, run FLAME, supplying one gene list per cluster or up to 10 cluster gene lists simultaneously, viewing interactive UpSet plots and generating a bubble plot summarizing enriched terms per database. SCALA supports only human and mouse datasets, so the marker symbols must match the corresponding organism's GO/KEGG annotation. Export enriched terms (term, database, statistical significance, gene overlap) per cluster to a results file such as functional_enrichment_results.csv; optionally complement the enrichment with decoupleR TF-activity scores inferred per cluster.

## Related tools

- **g:Profiler (gprofiler2)** (Performs GO biological process and KEGG pathway overrepresentation testing on each cluster's marker gene list; installed as a CRAN dependency of SCALA)
- **FLAME** (External application for deeper functional enrichment; accepts one gene list per cluster or up to 10 cluster gene lists simultaneously, with interactive UpSet plots and a per-database bubble plot)
- **decoupleR** (Complementary per-cluster TF activity inference scoring each cluster against a curated transcription-factor-to-target-gene resource)
- **SCALA** (Shiny web/desktop application that orchestrates the enrichment module downstream of marker identification in the scRNA-seq pipeline)

## Examples

```
library(gprofiler2); gost(cluster_markers$S1_up, organism="hsapiens", sources=c("GO:BP","KEGG"))
```

## Evaluation signals

- functional_enrichment_results.csv exists and contains one row per enriched term per input cluster with term, database, statistical significance, and gene-overlap fields
- Every input cluster has at least one enrichment record (or an explicitly empty result), and each record's gene overlap is a subset of that cluster's marker list
- Statistical significance values are valid p/FDR values within [0,1], and significance flags match the applied cut-offs
- The set of clusters in the enrichment output matches the cluster set from the marker identification step (e.g. S1–S5 SF clusters)
- The decoupleR TF-activity score column covers every input cluster when TF activity was requested

## Limitations

- Only human and mouse datasets are currently supported for both scRNA-seq and scATAC-seq modalities
- FLAME and enrichment accept at most 10 cluster gene lists simultaneously
- Online SCALA version restricts dataset size to < 2 GB and allocates no more than 2 CPU threads per session; for datasets larger than 50,000 cells some steps cannot be completed online and the desktop version is recommended

## Evidence

- [abstract] Abstract statement of SCALA's additional analysis modes including functional enrichment: "Additional analysis options enable functional enrichment, cellular trajectory inference, ligand-receptor analysis, and regulatory network reconstruction."
