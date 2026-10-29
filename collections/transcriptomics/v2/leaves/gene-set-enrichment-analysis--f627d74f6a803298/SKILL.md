---
name: gene-set-enrichment-analysis
description: Use when you have a list of mouse gene sets (e.g., co-expression modules
  or GEPs from SingleCellGGM) and need to test which GO or MP terms are statistically
  enriched at p <= 0.05.
when_to_use_negative:
- Gene IDs are not Ensembl gene IDs and cannot be mapped — ModuleEnrichmentAnalysis
  states 'Only Ensembl gene IDs are supported.'
- Analyzing non-mouse gene sets — the bundled GO (Ensembl BioMart) and MP (MGI) annotations
  are mouse-specific.
- The task is cell-type annotation or label transfer with the GEPs (use CellGEP/cellgep
  instead of enrichment testing).
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ModuleEnrichmentAnalysis
  role: R package providing go_enrichment_analysis and mp_enrichment_analysis functions
    that perform hypergeometric GO/MP over-representation testing with Benjamini-Hochberg
    correction on mouse gene sets
  repo: MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Produces the gene co-expression modules (GEPs, e.g., 137 MCA GEPs saved in
    data/mca.marker.geps.txt) that serve as the gene sets tested for enrichment
  repo: MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:9a754c34886a3a963870bd65c50e94ca41602a6d4028df70cb9f835b5172845d
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-set-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform over-representation testing of GO and Mouse Phenotype (MP) terms for mouse gene co-expression modules (e.g., SingleCellGGM-derived GEPs) using the ModuleEnrichmentAnalysis R functions, which apply a hypergeometric test with Benjamini-Hochberg correction and report enrichment ratios.

## When to use

Use when you have a list of mouse gene sets (e.g., co-expression modules or GEPs from SingleCellGGM) and need to test which GO or MP terms are statistically enriched at p <= 0.05.

## When NOT to use

- Gene IDs are not Ensembl gene IDs and cannot be mapped — ModuleEnrichmentAnalysis states 'Only Ensembl gene IDs are supported.'
- Analyzing non-mouse gene sets — the bundled GO (Ensembl BioMart) and MP (MGI) annotations are mouse-specific.
- The task is cell-type annotation or label transfer with the GEPs (use CellGEP/cellgep instead of enrichment testing).

## Inputs

- clusters.txt: two-column tab-separated file of Ensembl gene IDs and cluster/GEP IDs (137 MCA GEPs, e.g., from data/mca.marker.geps.txt)
- back_ground_genes.txt: list of background gene IDs used for the hypergeometric null
- data/Mouse.GO.annotation.txt and data/GO.names.txt: GO term annotations from Ensembl BioMart
- data/Mouse.MP.annotation.txt and data/MP.names.txt: MP term annotations from MGI
- data/Mouse.gene.symbl.txt: mouse gene symbol mapping file

## Outputs

- go object/table: per-GEP enriched GO terms with enrichment ratio, p-value, and BH-adjusted p-value
- mp object/table: per-GEP enriched MP terms with enrichment ratio, p-value, and BH-adjusted p-value
- enrichment_results.csv: full table of GEP id, term, ratio, p-value, BH-adjusted p-value

## How to apply

Format each gene module (GEP) as a two-column tab-separated clusters.txt file (gene ID in column 1, cluster/GEP id in column 2) and provide a back_ground_genes.txt list of background genes; only Ensembl gene IDs are supported, so use the symbol-mapping files (data/Mouse.gene.symbl.txt) or symbol-variant inputs if genes are in symbol format. Run go_enrichment_analysis using GO annotations downloaded from Ensembl BioMart (data/Mouse.GO.annotation.txt, data/GO.names.txt) and mp_enrichment_analysis using MP annotations from MGI (data/Mouse.MP.annotation.txt, data/MP.names.txt). Both functions test over-representation of each term per module with the hypergeometric test and adjust p-values with Benjamini-Hochberg. Record enrichment ratio, raw p-value, and BH-adjusted p-value per GEP-term pair (e.g., M83 'abnormal NK cell physiology' ER = 50.7, p = 3.75E-08; M72 ER = 157.5, p = 3.52E-21) and write the full table to enrichment_results.csv. Modules whose enrichment is significant at BH-adjusted p <= 0.05 validate the biological relevance of the GEPs.

## Related tools

- **ModuleEnrichmentAnalysis** (R package providing go_enrichment_analysis and mp_enrichment_analysis functions that perform hypergeometric GO/MP over-representation testing with Benjamini-Hochberg correction on mouse gene sets) — MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Produces the gene co-expression modules (GEPs, e.g., 137 MCA GEPs saved in data/mca.marker.geps.txt) that serve as the gene sets tested for enrichment) — MaShisongLab/SingleCellGGM

## Examples

```
mp = mp_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', mp_annotation_file='data/Mouse.MP.annotation.txt', mp_name_file='data/MP.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- 123 of the 137 MCA GEPs have significantly enriched GO terms at BH-adjusted p <= 0.05
- MP term 'abnormal NK cell physiology' shows 50.7-fold enrichment in GEP M83 with p = 3.75E-08
- GEP M72 shows an enrichment ratio of 157.5 with p = 3.52E-21 for its top MP term
- Output table schema contains GEP id, term, enrichment ratio, raw p-value, and BH-adjusted p-value columns
- Reported enrichment ratios exceed 10-fold for many GEPs, consistent with the paper's claim of strong functional enrichment

## Limitations

- Brain-related GO and MP annotations lack granularity, so enriched terms for neuronal subsets (e.g., Mouse Brain dataset GEPs) offer limited functional insight; further functional studies are needed.
- Only Ensembl gene IDs are supported by ModuleEnrichmentAnalysis inputs.
- SingleCellGGM GEPs rely exclusively on transcriptome data, so enrichment reflects transcriptome-based co-expression only.

## Evidence

- [readme] ModuleEnrichmentAnalysis is an algorithm to perform enrichment analysis for mouse gene sets. The GO term annotations were obtained from Ensembl BioMart, while the MP term annotations were obtained from MGI: [evidence span withheld — anchor status: external_doc]
- [readme] Only Ensembl gene IDs are supported.: [evidence span withheld — anchor status: external_doc]
- [results] GO analysis revealed enrichment of GO terms in 123 of the 137 MCA GEPs (p ≤ 0.05, Benjamini-Hochberg adjusted): [evidence span withheld — anchor status: not_found]
- [methods] For each of the 137 GEPs, perform over-representation testing of each gene set with the hypergeometric test and adjust p-values with the Benjamini-Hochberg (BH) method.: [evidence span withheld — anchor status: not_found]
