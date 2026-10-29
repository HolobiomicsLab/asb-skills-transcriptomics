---
name: benjamini-hochberg-multiple-testing-correction
description: Use when hundreds of module-vs-term hypergeometric tests are performed
  (e.g., 137 MCA GEPs against GO/MP gene sets) and enriched terms must be reported
  with BH-adjusted p-values alongside raw p-values and enrichment ratios.
when_to_use_negative:
- You are controlling FDR for the SingleCellGGM network edges themselves, where the
  paper uses the permutation-based fdr_control function on permutated edges rather
  than BH on enrichment p-values
- Gene sets are few and pre-specified such that no large family of simultaneous tests
  exists, making BH adjustment unnecessary or overly conservative
- Input genes are not Ensembl mouse gene IDs; ModuleEnrichmentAnalysis supports only
  Ensembl gene IDs unless the symbol-variant GEP file is used
tools:
- name: ModuleEnrichmentAnalysis
  role: Performs the GO and MP over-representation (hypergeometric) tests on GEP gene
    sets whose p-values are then BH-adjusted
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- name: SingleCellGGM
  role: Produces the GEPs (from the MCA dataset) and provides a separate permutation-based
    fdr_control function for FDR control of network edges at different pcor cutoffs
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:305f46572665ed662a80328d514316841b16a84b489bcd397d29a1cabb0b54bf
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# benjamini-hochberg-multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply Benjamini-Hochberg (BH) false discovery rate adjustment to hypergeometric over-representation p-values when testing many gene co-expression modules (GEPs) against GO and Mouse Phenotype (MP) gene sets, so that enriched terms (e.g., 'abnormal NK cell physiology' in GEP M83) are called at a controlled FDR rather than raw p <= 0.05.

## When to use

Use when hundreds of module-vs-term hypergeometric tests are performed (e.g., 137 MCA GEPs against GO/MP gene sets) and enriched terms must be reported with BH-adjusted p-values alongside raw p-values and enrichment ratios.

## When NOT to use

- You are controlling FDR for the SingleCellGGM network edges themselves, where the paper uses the permutation-based fdr_control function on permutated edges rather than BH on enrichment p-values
- Gene sets are few and pre-specified such that no large family of simultaneous tests exists, making BH adjustment unnecessary or overly conservative
- Input genes are not Ensembl mouse gene IDs; ModuleEnrichmentAnalysis supports only Ensembl gene IDs unless the symbol-variant GEP file is used

## Inputs

- Hypergeometric over-representation p-values for each GEP x term combination (GO terms from Ensembl BioMart; MP terms from MGI)
- GEP gene lists (e.g., 'data/mca.marker.geps.txt' or 'mca.marker.symbl.geps.txt' for gene-symbol IDs)
- Background gene list ('back_ground_genes.txt', Ensembl gene IDs only)
- GO/MP annotation files: 'data/Mouse.GO.annotation.txt', 'data/Mouse.MP.annotation.txt' with name files 'data/GO.names.txt', 'data/MP.names.txt'

## Outputs

- Enrichment table (enrichment_results.csv) with columns: GEP id, term, enrichment ratio, raw p-value, BH-adjusted p-value
- Count of GEPs with significant enriched terms (e.g., 123 of 137 MCA GEPs at p <= 0.05)

## How to apply

For each of the GEPs (e.g., the 137 MCA marker GEPs loaded from 'data/mca.marker.geps.txt'), run an over-representation test of every GO and MP gene set using the hypergeometric test, taking the GEP members as the query set and the network/background genes as the tested universe. Compute an enrichment ratio (observed/expected gene overlap) for each term alongside the raw p-value. Then apply the Benjamini-Hochberg procedure across all tested terms (per module or per test family) to obtain adjusted p-values, and call terms significant at BH-adjusted p <= 0.05. Record GEP id, term, ratio, raw p-value, and BH-adjusted p-value per row (e.g., M83 ER = 50.7, p = 3.75E-08; M72 ER = 157.5, p = 3.52E-21) and write the table to enrichment_results.csv. Rationale: with 137 modules and thousands of GO/MP terms, raw p <= 0.05 would yield many false positives; BH adjustment is what lets the paper claim GO enrichment in 123 of 137 MCA GEPs.

## Related tools

- **ModuleEnrichmentAnalysis** (Performs the GO and MP over-representation (hypergeometric) tests on GEP gene sets whose p-values are then BH-adjusted) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
- **SingleCellGGM** (Produces the GEPs (from the MCA dataset) and provides a separate permutation-based fdr_control function for FDR control of network edges at different pcor cutoffs) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
go = go_enrichment_analysis(cluster_file='clusters.txt', bk_gene_file='back_ground_genes.txt', go_annotation_file='data/Mouse.GO.annotation.txt', go_name_file='data/GO.names.txt', gene_symbl_file='data/Mouse.gene.symbl.txt')
```

## Evaluation signals

- 123 of the 137 MCA GEPs have enriched GO terms at p <= 0.05 (BH-adjusted) in the output table
- Reported MP enrichment results are reproduced: M83 ER = 50.7 with p = 3.75E-08, and M72 ER = 157.5 with p = 3.52E-21
- Every output row contains all five fields (GEP id, term, enrichment ratio, p-value, BH-adjusted p-value) with BH-adjusted p >= raw p
- Significant terms are called at BH-adjusted p <= 0.05, not raw p <= 0.05
- Background gene universe matches the genes actually tested (network genes with Ensembl IDs) so enrichment ratios are computed against the correct denominator

## Limitations

- Enriched GO and MP terms can be less compelling for some datasets (e.g., Mouse Brain), offering limited insight into functional differences among neuronal cell subsets
- This limitation may stem from the lack of granularity in brain-related GO and MP gene annotations of the genes present in the GEPs, indicating a need for further functional studies
- ModuleEnrichmentAnalysis supports only Ensembl gene IDs for the cluster and background gene files

## Evidence

- [readme] Input annotation sources for the tested term families: [evidence span withheld — anchor status: external_doc]
- [readme] Input format for the enrichment tests: [evidence span withheld — anchor status: external_doc]
- [discussion] Annotation granularity limits interpretation of enrichment results: "This limitation may stem from the lack of granularity in the brain-related GO and MP annotations of the genes present in the GEPs"
- [readme] Separate permutation-based FDR for network edges in SingleCellGGM: [evidence span withheld — anchor status: external_doc]
