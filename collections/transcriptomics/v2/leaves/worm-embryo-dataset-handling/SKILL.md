---
name: worm-embryo-dataset-handling
description: Use when the input is the CEED (GSE126954) C. elegans single-cell embryogenesis
  dataset (or another time-series worm scRNA-seq matrix) and the goal is to identify
  time-varying differential genes and cell subclusters with scTITANS.
when_to_use_negative:
- 'When automatic cell type annotation is required: the scTITANS `sp` parameter accepts
  only Human or Mouse, so C. elegans data must run with annotation=''F'' and manual
  marker-gene annotation'
- 'When the input is single-bulk or non-time-series data: scTITANS requires time-series
  scRNA-seq data where trajectory-based pseudotime correction of cell asynchrony is
  meaningful'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTITANS
  role: Identifies differential genes and cell subclusters from the CEED time-series
    data by combining trajectory inference (Monocle3 pseudotime) with a time-dependent
    covariate, run at fdr 0.01
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Reduces dimensionality, learns the trajectory graph, and orders CEED cells
    by pseudotime prior to scTITANS differential testing
- name: scran
  role: Normalizes the CEED matrix using the default pool-and-deconvolute normalization
    algorithm
- name: ToppGene
  role: Performs GO biological-process enrichment on the top scTITANS CEED DEGs to
    validate biological meaningfulness
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:427d2bbcdfdc3a76ffdd500110b4fdfc510e7608ea04d73aed95e52801fe8fe3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - ToppGene
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# worm-embryo-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A workflow for analyzing the CEED (GSE126954) C. elegans embryogenesis time-series scRNA-seq dataset with scTITANS: preprocessing the raw digital gene expression matrix, clustering and ordering cells by pseudotime with Monocle3, and identifying differential genes at fdr level 0.01 for downstream GO enrichment validation.

## When to use

Use when the input is the CEED (GSE126954) C. elegans single-cell embryogenesis dataset (or another time-series worm scRNA-seq matrix) and the goal is to identify time-varying differential genes and cell subclusters with scTITANS.

## When NOT to use

- When automatic cell type annotation is required: the scTITANS `sp` parameter accepts only Human or Mouse, so C. elegans data must run with annotation='F' and manual marker-gene annotation
- When the input is single-bulk or non-time-series data: scTITANS requires time-series scRNA-seq data where trajectory-based pseudotime correction of cell asynchrony is meaningful

## Inputs

- Raw or preprocessed digital gene expression matrix of CEED (GSE126954) C. elegans embryo cells (genes as rows, cells as columns)
- Optional cell metadata (clusters/cell types) and gene metadata with gene_short_name column for 'partial' mode

## Outputs

- SigGenes.fdr0.01.txt — significant differential genes at fdr level 0.01
- SigClusters.fdr0.01.txt — significant cell subclusters at fdr level 0.01
- Cell type annotations for the 27 Louvain-clustered CEED cell types based on literature marker genes
- Downstream GO BP enrichment table (toppgene_go_enrichment.csv) with term IDs, names, p-values, and FDR for the top DEG list

## How to apply

Obtain the CEED (GSE126954) single-cell transcriptome matrix of C. elegans embryos and preprocess it by filtering (discard genes without counts in any cell; a gene present with two or more transcripts in at least 10 cells is 'detected'; discard cells with detected-gene counts outside the 5th–95th percentile or with >10% UMIs assigned to mitochondrial genes), then normalize with the scran pool-and-deconvolute algorithm. Because scTITANS annotation only supports Human or Mouse species, run the CEED data in 'full' mode with annotation='F' so cells are clustered with the Louvain algorithm and annotated manually against literature-reported marker genes (the paper annotates 27 cell types), rather than via scCATCH. Use Monocle3 to reduce dimensionality, learn the trajectory graph, and order cells by pseudotime; scTITANS then introduces a time-dependent covariate to evaluate gene and cell-number fluctuations along pseudotime and calls differential genes and significant subclusters at the paper's fdr level of 0.01. The run returns SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt, which can be submitted to ToppGene (GO biological-process terms) to confirm biological meaningfulness — the paper confirmed 11 of the top 12 CEED DEGs by literature and found typical developmental processes (e.g., ascending aorta morphogenesis) enriched among the top 20 DEGs.

## Related tools

- **scTITANS** (Identifies differential genes and cell subclusters from the CEED time-series data by combining trajectory inference (Monocle3 pseudotime) with a time-dependent covariate, run at fdr 0.01) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Reduces dimensionality, learns the trajectory graph, and orders CEED cells by pseudotime prior to scTITANS differential testing)
- **scran** (Normalizes the CEED matrix using the default pool-and-deconvolute normalization algorithm)
- **ToppGene** (Performs GO biological-process enrichment on the top scTITANS CEED DEGs to validate biological meaningfulness)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- scTITANS run completes and produces both SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt; a displayed error message indicates significant-cluster identification failed
- The DEG list corresponds to the paper's fdr level of 0.01 and the CEED cells are grouped into the expected 27 annotated cell types from Louvain clustering
- The downstream enrichment table toppgene_go_enrichment.csv parses as a valid table and contains GO BP term annotations plus p-value and FDR statistics
- Literature cross-check: roughly 11 of the top 12 CEED DEGs should be confirmable via literature survey, with enriched GO BP terms reflecting typical developmental processes

## Limitations

- scTITANS automatic annotation supports only Human or Mouse species, so for C. elegans the annotation parameter must be 'F' and cell types must be assigned manually from marker genes
- No software version numbers are stated for scTITANS, Monocle3, or scran in the article, and no changelog is available for the scTITANS repository
- If something goes wrong in identifying significant clusters, scTITANS only displays an error message rather than returning partial results

## Evidence
