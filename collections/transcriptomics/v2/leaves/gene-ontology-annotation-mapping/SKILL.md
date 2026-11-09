---
name: gene-ontology-annotation-mapping
description: Use when you have mouse gene clusters/modules (e.g., co-expression GEPs)
  and need to determine which GO or MP terms are statistically enriched, with enrichment
  ratios and BH-adjusted p-values.
when_to_use_negative:
- Input gene identifiers are not Ensembl gene IDs (e.g., symbols without prior conversion
  via Mouse.gene.symbl.txt) — ModuleEnrichmentAnalysis only supports Ensembl gene
  IDs
- The gene set is from a non-mouse organism — annotations provided are mouse-specific
  (Ensembl BioMart GO and MGI MP)
- You need enrichment for modalities beyond GO/MP term ontology, or brain-focused
  functional annotation where GO/MP granularity is lacking
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: ModuleEnrichmentAnalysis
  role: Performs GO and MP term over-representation analysis (hypergeometric test,
    BH adjustment) on mouse gene clusters using bundled annotation files
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Upstream source of the gene co-expression modules (GEPs) being enriched, e.g.,
    the 137 MCA GEPs saved in data/mca.marker.geps.txt
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: CellGEP
  role: Distributes the marker GEP list (data/mca.marker.geps.txt) identified from
    the mouse MCA dataset that serves as input clusters for the enrichment analysis
  repo: https://github.com/MaShisongLab/cellgep
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:9524401980392aa93518769c263f7b3ee4a38e3a16305deb8455cabd4cff6289
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
  - CellGEP
  - ModuleEnrichmentAnalysis
  - SingleCellGGM
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-ontology-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps mouse gene sets (e.g., SingleCellGGM-derived GEPs) to GO and Mouse Phenotype (MP) terms via over-representation testing with the hypergeometric test and Benjamini-Hochberg correction, using ModuleEnrichmentAnalysis annotation files from Ensembl BioMart and MGI.

## When to use

Use when you have mouse gene clusters/modules (e.g., co-expression GEPs) and need to determine which GO or MP terms are statistically enriched, with enrichment ratios and BH-adjusted p-values.

## When NOT to use

- Input gene identifiers are not Ensembl gene IDs (e.g., symbols without prior conversion via Mouse.gene.symbl.txt) — ModuleEnrichmentAnalysis only supports Ensembl gene IDs
- The gene set is from a non-mouse organism — annotations provided are mouse-specific (Ensembl BioMart GO and MGI MP)
- You need enrichment for modalities beyond GO/MP term ontology, or brain-focused functional annotation where GO/MP granularity is lacking

## Inputs

- clusters.txt: tab-separated two-column file (gene id, cluster/GEP id) listing module membership
- back_ground_genes.txt: list of background genes used for the enrichment analysis (Ensembl gene IDs only)
- data/Mouse.GO.annotation.txt: GO term annotations for mouse genes (from Ensembl BioMart)
- data/Mouse.MP.annotation.txt: MP term annotations for mouse genes (from MGI)
- data/GO.names.txt and data/MP.names.txt: term name mappings
- data/Mouse.gene.symbl.txt: mouse gene symbol mapping

## Outputs

- Per-module enrichment table (go / mp objects in R) with term, enrichment ratio, raw p-value, and BH-adjusted p-value
- enrichment_results.csv with columns GEP id, term, ratio, p-value, BH-adjusted p-value

## How to apply

Prepare a clusters file (two tab-separated columns: gene id and cluster id) and a background gene list; note that ModuleEnrichmentAnalysis supports only Ensembl gene IDs, so convert symbols via the provided 'data/Mouse.gene.symbl.txt' mapping (e.g., use the 'mca.marker.symbl.geps.txt' variant if GEP genes are in symbol format). Run go_enrichment_analysis using 'data/Mouse.GO.annotation.txt' (annotations from Ensembl BioMart) and mp_enrichment_analysis using 'data/Mouse.MP.annotation.txt' (annotations from MGI), passing the background gene file in both cases. Each term is tested per module with a hypergeometric over-representation test and p-values are adjusted with the Benjamini-Hochberg method; record the enrichment ratio per significant term. Write the full table (GEP id, term, ratio, p-value, BH-adjusted p-value) to enrichment_results.csv; the MCA benchmark enriched GO terms in 123 of 137 GEPs at p <= 0.05, with MP terms strongly enriched (e.g., 'abnormal NK cell physiology' 50.7-fold in GEP M83, p = 3.75E-08; M72 ER = 157.5, p = 3.52E-21).

## Related tools

- **ModuleEnrichmentAnalysis** (Performs GO and MP term over-representation analysis (hypergeometric test, BH adjustment) on mouse gene clusters using bundled annotation files) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Upstream source of the gene co-expression modules (GEPs) being enriched, e.g., the 137 MCA GEPs saved in data/mca.marker.geps.txt) — https://github.com/MaShisongLab/SingleCellGGM
- **CellGEP** (Distributes the marker GEP list (data/mca.marker.geps.txt) identified from the mouse MCA dataset that serves as input clusters for the enrichment analysis) — https://github.com/MaShisongLab/cellgep

## Examples

```
go = go_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', go_annotation_file='data/Mouse.GO.annotation.txt', go_name_file='data/GO.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- GO enrichment is found for ~123 of the 137 MCA GEPs at BH-adjusted p <= 0.05 when reproducing the paper benchmark
- MP enrichment for GEP M83 reproduces 'abnormal NK cell physiology' at ER = 50.7, p = 3.75E-08
- MP enrichment for GEP M72 reproduces ER = 157.5, p = 3.52E-21
- Output table contains all five required columns (GEP id, term, ratio, p-value, BH-adjusted p-value) for every module
- All genes submitted are Ensembl gene IDs; no symbol-format entries leak into clusters.txt or back_ground_genes.txt

## Limitations

- Only Ensembl gene IDs are supported by ModuleEnrichmentAnalysis
- Brain-related GO and MP gene annotations lack granularity, so enriched terms for neuronal GEPs (e.g., from the Mouse Brain dataset) offer limited insight into functional differences among neuronal cell subsets
- Results depend on the chosen background gene list; the GEPs contain genes whose enrichment sometimes exceeds 10-fold but annotation coverage constrains interpretation

## Evidence

- [readme] ModuleEnrichmentAnalysis is an algorithm to perform enrichment analysis for mouse gene sets. The GO term annotations were obtained from Ensembl BioMart, while the MP term annotations were obtained from MGI: [evidence span withheld — anchor status: external_doc]
- [readme] Only Ensembl gene IDs are supported.: [evidence span withheld — anchor status: external_doc]
- [other] GO analysis revealed enrichment of GO terms in 123 of the 137 MCA GEPs (p ≤ 0.05, Benjamini-Hochberg adjusted): [evidence span withheld — anchor status: not_found]
- [other] MP terms strongly enriched, e.g., 'abnormal NK cell physiology' is 50.7-fold enriched in GEP M83 (p = 3.75E−08): [evidence span withheld — anchor status: not_found]
