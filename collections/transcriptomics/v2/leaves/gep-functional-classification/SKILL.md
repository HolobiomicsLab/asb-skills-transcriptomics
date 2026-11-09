---
name: gep-functional-classification
description: Use when you have gene co-expression modules (GEPs, e.g. MCL clusters
  of a GGM network) and need per-module GO/MP enrichment significance and a functional
  category split (identity vs. activity vs. morphogenesis).
when_to_use_negative:
- The gene set is not mouse, since ModuleEnrichmentAnalysis annotations are mouse-only
  (Ensembl BioMart GO and MGI MP) and only Ensembl gene IDs are supported
- Modules have fewer than 10 genes — the article restricts GEPs to modules with ≥10
  genes before enrichment
- You only need cell-type label transfer (CellGEP) rather than functional characterization
  of the programs themselves
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ModuleEnrichmentAnalysis
  role: Runs GO and MP term enrichment analysis on each GEP gene set against the network
    background genes
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Upstream generator of the GGM co-expression network whose MCL clusters form
    the GEPs being enriched
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:efe97366f54630ad388d20779edea8bc5e09a36caec6b5413b1a0c3c8068eae9
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
  - ModuleEnrichmentAnalysis
  - SingleCellGGM
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gep-functional-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functionally characterize gene expression programs (GEPs) identified by SingleCellGGM by running GO and MP term enrichment on each module (ModuleEnrichmentAnalysis) and classifying GEPs into cell-identity, activity, and morphogenesis programs.

## When to use

Use when you have gene co-expression modules (GEPs, e.g. MCL clusters of a GGM network) and need per-module GO/MP enrichment significance and a functional category split (identity vs. activity vs. morphogenesis).

## When NOT to use

- The gene set is not mouse, since ModuleEnrichmentAnalysis annotations are mouse-only (Ensembl BioMart GO and MGI MP) and only Ensembl gene IDs are supported
- Modules have fewer than 10 genes — the article restricts GEPs to modules with ≥10 genes before enrichment
- You only need cell-type label transfer (CellGEP) rather than functional characterization of the programs themselves

## Inputs

- GEP membership table (clusters.txt: tab-separated gene id + cluster id, Ensembl gene IDs)
- Background gene list (back_ground_genes.txt) of genes used in the co-expression network
- ModuleEnrichmentAnalysis annotation files (data/Mouse.GO.annotation.txt, data/Mouse.MP.annotation.txt, GO.names.txt, MP.names.txt, Mouse.gene.symbl.txt)

## Outputs

- Per-GEP GO term enrichment results with p-values (flagging significantly enriched GEPs, e.g. 135/158 at p ≤ 0.05)
- Per-GEP MP term enrichment results
- Functional category assignment of GEPs into cell-identity, activity, and morphogenesis programs
- Category summary counts (e.g. 116 cell-identity / 25 activity / 17 morphogenesis GEPs)

## How to apply

Take the GEP membership table (gene IDs clustered into modules, e.g. the 158 Mouse Brain GEPs B1–B158, each with ≥10 genes) and write it as clusters.txt (two tab-separated columns: gene id, cluster id, Ensembl IDs only) plus a back_ground_genes.txt list of the network's background genes. Run go_enrichment_analysis and mp_enrichment_analysis from ModuleEnrichmentAnalysis using the bundled Ensembl BioMart GO and MGI MP annotation files, calling enriched terms at p ≤ 0.05. Then assign each GO-enriched GEP to a category: cell-identity programs (marker-like, cell-type-specific), activity programs (functional state genes), or morphogenesis programs (developmental genes), verifying category counts against expected splits such as the reported 116/25/17. Enrichment levels can exceed 10-fold for genes required for the corresponding cell's function, which serves as a sanity check.

## Related tools

- **ModuleEnrichmentAnalysis** (Runs GO and MP term enrichment analysis on each GEP gene set against the network background genes) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Upstream generator of the GGM co-expression network whose MCL clusters form the GEPs being enriched) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
go = go_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', go_annotation_file='data/Mouse.GO.annotation.txt', go_name_file='data/GO.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- Reproducible count of enriched GEPs: 135 of 158 Mouse Brain GEPs with significant GO enrichment at p ≤ 0.05
- Category split matches the reported 116 cell-identity / 25 activity / 17 morphogenesis GEPs
- Enriched GEPs show high fold-enrichment for genes required for the corresponding cell's function, sometimes greater than 10-fold
- clusters.txt is schema-valid: exactly two tab-separated columns, Ensembl gene IDs only, every GEP has ≥10 genes
- Every GEP member gene appears in back_ground_genes.txt

## Limitations

- Enriched GO and MP terms for the Mouse Brain dataset were less compelling, offering limited insight into functional differences among neuronal cell subsets, attributed to lack of granularity in brain-related GO/MP gene annotations
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics
- Activity and morphogenesis GEPs have been largely overlooked by previous studies, so few reference annotations exist for cross-checking these categories

## Evidence

- [readme] ModuleEnrichmentAnalysis performs GO and MP enrichment for mouse gene sets: [evidence span withheld — anchor status: external_doc]
- [readme] clusters.txt input format specification: [evidence span withheld — anchor status: external_doc]
