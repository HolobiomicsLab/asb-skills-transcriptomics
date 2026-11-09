---
name: mammalian-phenotype-enrichment-analysis
description: Use when you have mouse gene clusters/modules (e.g., gene co-expression
  GEPs) and need to test which MP phenotype terms are significantly over-represented,
  reporting enrichment ratios and adjusted p-values per module.
when_to_use_negative:
- The gene identifiers are not Ensembl gene IDs and cannot be mapped — ModuleEnrichmentAnalysis
  supports only Ensembl gene IDs.
- The goal is cell-type label transfer or GEP identification from expression matrices
  rather than enrichment testing of pre-defined gene modules — use CellGEP/SingleCellGGM
  instead.
- The organism is not mouse — MP annotations here come from MGI mouse data, so non-mouse
  gene sets are out of scope.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: ModuleEnrichmentAnalysis
  role: R package providing mp_enrichment_analysis for hypergeometric MP term over-representation
    testing of mouse gene clusters with BH adjustment
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Upstream method that produces the gene co-expression modules (GEPs, e.g.,
    the 137 MCA GEPs) that serve as input clusters for enrichment testing
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:39adc4095d06bee1490ccf0e1b6c4b35bee967c7f1dc93646acab25a4864c83e
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mammalian-phenotype-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Over-representation analysis of mouse gene sets (e.g., SingleCellGGM-derived GEPs) against Mammalian Phenotype (MP) ontology terms using hypergeometric testing with Benjamini-Hochberg correction, as implemented in MaShisongLab/ModuleEnrichmentAnalysis.

## When to use

Use when you have mouse gene clusters/modules (e.g., gene co-expression GEPs) and need to test which MP phenotype terms are significantly over-represented, reporting enrichment ratios and adjusted p-values per module.

## When NOT to use

- The gene identifiers are not Ensembl gene IDs and cannot be mapped — ModuleEnrichmentAnalysis supports only Ensembl gene IDs.
- The goal is cell-type label transfer or GEP identification from expression matrices rather than enrichment testing of pre-defined gene modules — use CellGEP/SingleCellGGM instead.
- The organism is not mouse — MP annotations here come from MGI mouse data, so non-mouse gene sets are out of scope.

## Inputs

- clusters.txt: two-column tab-separated file of gene id and cluster/GEP id (Ensembl gene IDs only)
- back_ground_genes.txt: list of background genes used for the hypergeometric test
- data/Mouse.MP.annotation.txt: MP term annotations for mouse genes (from MGI)
- data/MP.names.txt: MP term names
- data/Mouse.gene.symbl.txt: mouse gene symbol mapping

## Outputs

- mp enrichment result object/table with per-cluster MP terms, enrichment ratios, raw p-values, and BH-adjusted p-values
- exported enrichment table (e.g., enrichment_results.csv) with GEP id, term, ratio, p-value, BH-adjusted p-value

## How to apply

Format the gene modules as a two-column tab-separated clusters.txt file (gene id, cluster id) plus a back_ground_genes.txt list; only Ensembl gene IDs are supported, so map gene symbols first (e.g., use the symbol-variant GEP files or the provided Mouse.gene.symbl.txt mapping). Run mp_enrichment_analysis in R with the MP annotation files data/Mouse.MP.annotation.txt and data/MP.names.txt, which were obtained from MGI. The function performs a hypergeometric over-representation test of each module against each MP term and adjusts p-values with Benjamini-Hochberg; compute enrichment ratios for significant terms and export the full table (GEP id, term, ratio, p-value, BH-adjusted p-value). Validate against known results, e.g., 'abnormal NK cell physiology' is 50.7-fold enriched in MCA GEP M83 (p = 3.75E-08) and M72 shows ER = 157.5 (p = 3.52E-21); in the source study, GO terms were enriched in 123 of 137 MCA GEPs at p ≤ 0.05, and MP terms showed strong enrichment.

## Related tools

- **ModuleEnrichmentAnalysis** (R package providing mp_enrichment_analysis for hypergeometric MP term over-representation testing of mouse gene clusters with BH adjustment) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Upstream method that produces the gene co-expression modules (GEPs, e.g., the 137 MCA GEPs) that serve as input clusters for enrichment testing) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
mp = mp_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', mp_annotation_file='data/Mouse.MP.annotation.txt', mp_name_file='data/MP.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- 123 of the 137 MCA GEPs show enriched GO terms at p ≤ 0.05 (BH-adjusted), indicating the enrichment pipeline is run consistently across modules
- MP term 'abnormal NK cell physiology' is reproduced at 50.7-fold enrichment in GEP M83 with p = 3.75E-08
- GEP M72 reproduces ER = 157.5 with p = 3.52E-21
- Output table contains the required columns: GEP id, term, enrichment ratio, p-value, BH-adjusted p-value
- All input gene IDs resolve against the Ensembl-based annotation files (no unmapped IDs silently dropped)

## Limitations

- MP/GO enrichment of brain-related modules was less compelling, attributed to lack of granularity in brain-related GO and MP gene annotations
- Only Ensembl gene IDs are supported for the cluster and background gene files
- Enrichment results depend on the chosen background gene list (back_ground_genes.txt)

## Evidence

- [readme] MP annotations sourced from MGI: [evidence span withheld — anchor status: external_doc]
- [readme] Only Ensembl gene IDs supported: [evidence span withheld — anchor status: external_doc]
- [other] Enrichment findings for MCA GEPs: [evidence span withheld — anchor status: not_found]
