---
name: enrichment-result-comparison-across-marker-methods
description: Use when you have marker gene lists from two or more selection methods
  and need to determine which method's markers better recover known tissue/cell-type
  biology via enrichment q values against a reference gene set library.
when_to_use_negative:
- You have only one marker set and no comparator — there is nothing to compare q values
  against, so enrichment alone is not this skill's purpose
- The target phenotypes have no meaningful overlap with the reference library's terms
  (e.g., mouse datasets like Tabula Muris against a HuBMAP human ASCT+B library)
- The goal is differential expression or deconvolution accuracy rather than external
  biological validation of marker discriminative power
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GSEApy
  role: Python interface to Enrichr used to submit the G-PC-selected HCA marker list
    and retrieve enrichment results (term, overlap, p value, q value)
- name: Enrichr API
  role: Backend enrichment service queried against the HuBMAP_ASCTplusB_augmented_2022
    gene set library
- name: phenotype-cover
  role: Produces the G-PC (GreedyPC) marker gene list at coverage 8 that serves as
    the enrichment input
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Used in Python to aggregate q values at pair/tissue/cell-type levels and compute
    the fraction of wins
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:0484c68ec1c3039a2c49f1ad6774f8314f3cacd3afa754fc22972f05e8fe61e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - Enrichr API
  - GSEApy
  - Python
  - numpy
  - phenotype-cover
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# enrichment-result-comparison-across-marker-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates marker sets selected by different algorithms (e.g., G-PC vs. decision trees) by running gene set enrichment analysis (GSEA) against an independent curated gene set library (HuBMAP ASCT+B) and comparing adjusted q values across methods at term, tissue, cell-type, and tissue/cell-type-pair levels.

## When to use

Use when you have marker gene lists from two or more selection methods and need to determine which method's markers better recover known tissue/cell-type biology via enrichment q values against a reference gene set library.

## When NOT to use

- You have only one marker set and no comparator — there is nothing to compare q values against, so enrichment alone is not this skill's purpose
- The target phenotypes have no meaningful overlap with the reference library's terms (e.g., mouse datasets like Tabula Muris against a HuBMAP human ASCT+B library)
- The goal is differential expression or deconvolution accuracy rather than external biological validation of marker discriminative power

## Inputs

- Marker gene list from G-PC selection at coverage 8 (HCA dataset), genes s with M_i,s >= M_j,s + 1 for some phenotype j
- Comparator marker gene lists (e.g., from decision trees)
- Reference gene set library: HuBMAP_ASCTplusB_augmented_2022 (via Enrichr)

## Outputs

- Enrichment table (term, overlap, p value, adjusted q value) written to gsea_enrichment_results.csv
- Aggregated q values per tissue/cell-type pair (42), tissue (11), and cell type (7)
- Fraction of comparisons where the focal method has lower q values (e.g., 71%, 54%, 42%)

## How to apply

Start from the marker list produced by the marker-selection sub-task — for G-PC this is, at a fixed coverage of 8, the set of genes s in solution S for which some phenotype j satisfies M_i,s >= M_j,s + 1. Submit the gene list to the Enrichr API through GSEApy's enrichr interface, querying the HuBMAP_ASCTplusB_augmented_2022 library, and retrieve term, overlap, p value, and adjusted q value for every matched ASCT+B tissue and cell-type term. Aggregate the q values at three levels: tissue/cell-type pairs (42 pairs), tissues (11), and cell types (7). For each comparison, count how often the focal method's q values are lower than the comparator's (e.g., decision trees); in the paper this yields 30/42 (71%) pairs, 6/11 (54%) tissues, and 3/7 (42%) cell types. Write the full enrichment table (e.g., gsea_enrichment_results.csv) so pairwise comparisons are reproducible.

## Related tools

- **GSEApy** (Python interface to Enrichr used to submit the G-PC-selected HCA marker list and retrieve enrichment results (term, overlap, p value, q value))
- **Enrichr API** (Backend enrichment service queried against the HuBMAP_ASCTplusB_augmented_2022 gene set library)
- **phenotype-cover** (Produces the G-PC (GreedyPC) marker gene list at coverage 8 that serves as the enrichment input) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Used in Python to aggregate q values at pair/tissue/cell-type levels and compute the fraction of wins)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=gpc_markers_cov8, gene_sets='HuBMAP_ASCTplusB_augmented_2022', outdir='.', cutoff=1.0); enr.results.to_csv('gsea_enrichment_results.csv')
```

## Evaluation signals

- Enrichment output contains adjusted q values for ASCT+B terms covering 42 tissue/cell-type pairs, 11 tissues, and 7 cell types
- Reproduced win fractions match the reported outcome: 30/42 (71%) pairs, 6/11 (54%) tissues, and 3/7 (42%) cell types with lower q values for G-PC
- Full enrichment table is written to gsea_enrichment_results.csv with columns term, overlap, p value, and adjusted q value
- Marker list input matches the coverage-8 extraction rule (genes s where M_i,s >= M_j,s + 1 for some phenotype j)
- All matched ASCT+B terms returned by the Enrichr API are retained before aggregation (no silent filtering of non-significant terms)

## Limitations

- The biological analysis relied on computing overlap with existing gene marker lists that may be incomplete
- G-PC does not provide an optimal solution, so the enrichment comparison validates a greedy approximation rather than a provably best marker set
- Enrichment comparison measures agreement with curated HuBMAP ASCT+B markers, not direct predictive accuracy of the marker sets

## Evidence

- [results] GSEA against HuBMAP ASCT+B: [evidence span withheld — anchor status: not_found]
