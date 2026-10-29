---
name: go-enrichment-gene-module-analysis
description: Use when you have gene co-expression modules (GEPs, e.g., from MCL clustering
  of a SingleCellGGM network) as gene-id/cluster-id pairs and need to quantify functional
  enrichment via GO/MP terms in mouse.
when_to_use_negative:
- The gene modules come from a non-mouse organism — ModuleEnrichmentAnalysis only
  supports mouse Ensembl gene IDs with mouse GO/MP annotation files
- You have raw per-cell expression and have not yet clustered genes into modules —
  first construct the co-expression network and run MCL to obtain GEPs
- Gene identifiers are gene symbols or non-Ensembl IDs, since only Ensembl gene IDs
  are supported by the tool
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: ModuleEnrichmentAnalysis
  role: Performs GO and MP term enrichment analysis on the GEP gene-module clusters
    using mouse Ensembl BioMart GO and MGI MP annotations
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Constructs the GGM co-expression networks from which GEPs (the modules tested
    for enrichment) are identified via MCL
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: MCL
  role: Clusters the inferred co-expression networks into gene modules (GEPs) that
    are then input to enrichment analysis
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:009fe00c1802a191f4ded6136244399fe4cd8b1841a8f8b9b989d16f6246d1ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
  tools:
  - MCL
  - ModuleEnrichmentAnalysis
  - SILGGM
  - SingleCellGGM
  - ppcor
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# go-enrichment-gene-module-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs GO (and MP) term enrichment analysis on mouse gene co-expression modules (GEPs) derived from SingleCellGGM networks, using the ModuleEnrichmentAnalysis tool with Ensembl BioMart GO annotations and MGI MP annotations, to identify which modules are biologically significant gene expression programs.

## When to use

Use when you have gene co-expression modules (GEPs, e.g., from MCL clustering of a SingleCellGGM network) as gene-id/cluster-id pairs and need to quantify functional enrichment via GO/MP terms in mouse.

## When NOT to use

- The gene modules come from a non-mouse organism — ModuleEnrichmentAnalysis only supports mouse Ensembl gene IDs with mouse GO/MP annotation files
- You have raw per-cell expression and have not yet clustered genes into modules — first construct the co-expression network and run MCL to obtain GEPs
- Gene identifiers are gene symbols or non-Ensembl IDs, since only Ensembl gene IDs are supported by the tool

## Inputs

- clusters.txt: two-column tab-separated file of Ensembl gene ID and cluster (GEP) ID
- back_ground_genes.txt: list of background gene Ensembl IDs used for the enrichment analysis
- GEP membership tables from MCL clustering of SingleCellGGM/ppcor/SILGGM networks
- data annotation files shipped with ModuleEnrichmentAnalysis (Mouse.GO.annotation.txt, Mouse.MP.annotation.txt, GO.names.txt, MP.names.txt, Mouse.gene.symbl.txt)

## Outputs

- Per-module GO enrichment results (enriched terms and p-values) via go_enrichment_analysis
- Per-module MP enrichment results via mp_enrichment_analysis
- Counts of GO-enriched GEPs (e.g., 123 SingleCellGGM / 98 ppcor / 175 SILGGM on MCA) and significant GEPs under >= 50-gene and p <= 1E-20 criteria
- Benchmark comparison table (benchmark_comparison.csv) of module, enrichment, significance, runtime, and memory metrics per method

## How to apply

Prepare a tab-separated 'clusters.txt' mapping Ensembl gene IDs to module IDs and a 'back_ground_genes.txt' listing the background genes analyzed (only Ensembl gene IDs are supported). Run go_enrichment_analysis and/or mp_enrichment_analysis from MaShisongLab/ModuleEnrichmentAnalysis, which uses GO annotations from Ensembl BioMart and MP annotations from MGI. Count enriched GEPs and apply the paper's significance criteria: for the MCA method comparison, significant GEPs are those with >= 50 genes (SingleCellGGM: 5; ppcor: 64; SILGGM: 1) and those with GO enrichment p <= 1E-20 (SingleCellGGM: 6; ppcor: 32; SILGGM: 2); for the Mouse Brain dataset, GEPs with p <= 0.05 were considered significantly enriched (135 of 158). Only modules with >= 10 genes are designated GEPs before enrichment. Assemble enriched-module counts into a per-method comparison (e.g., benchmark_comparison.csv) for benchmarking.

## Related tools

- **ModuleEnrichmentAnalysis** (Performs GO and MP term enrichment analysis on the GEP gene-module clusters using mouse Ensembl BioMart GO and MGI MP annotations) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Constructs the GGM co-expression networks from which GEPs (the modules tested for enrichment) are identified via MCL) — https://github.com/MaShisongLab/SingleCellGGM
- **MCL** (Clusters the inferred co-expression networks into gene modules (GEPs) that are then input to enrichment analysis)

## Examples

```
go = go_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', go_annotation_file='data/Mouse.GO.annotation.txt', go_name_file='data/GO.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- The clusters.txt file contains two tab-separated columns: Ensembl gene id in the first column and cluster id in the second, with a back_ground_genes.txt file supplied
- On the MCA comparison, GO-enriched GEP counts match the reported values (98 for ppcor, 175 for SILGGM, 123 for SingleCellGGM)
- Significant-GEP counts match paper criteria: 64/1/5 GEPs with >= 50 genes and 32/2/6 GEPs at p <= 1E-20 for ppcor/SILGGM/SingleCellGGM
- On the Mouse Brain dataset, 135 of 158 GEPs (B1–B158) show GO enrichment at p <= 0.05, and the GEP split of 116 cell-identity / 25 activity / 17 morphogenesis programs is reproduced
- Enriched GEPs show substantial fold enrichment, sometimes at levels greater than 10-fold, as reported in the paper

## Limitations

- Enriched GO and MP terms for the Mouse Brain dataset were less compelling, with limited insight into functional differences among neuronal cell subsets, attributed to the lack of granularity in brain-related GO and MP gene annotations
- Only Ensembl mouse gene IDs are supported by ModuleEnrichmentAnalysis
- Activity and morphogenesis GEPs may be under-detected in interpretation because previous single-cell studies have mainly focused on cell-type-specific marker genes

## Evidence

- [readme] ModuleEnrichmentAnalysis is an algorithm to perform enrichment analysis for mouse gene sets. The GO term annotations were obtained from Ensembl BioMart, while the MP term annotations were obtained from MGI: [evidence span withheld — anchor status: not_found]
- [readme] The algorithm takes a list of clusters (saved in the file `clusters.txt`) and a list of background genes (saved in the file `back_ground_genes.txt`) as input. Only Ensembl gene IDs are supported.: [evidence span withheld — anchor status: not_found]
