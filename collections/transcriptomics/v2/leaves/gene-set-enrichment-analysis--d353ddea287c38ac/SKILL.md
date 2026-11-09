---
name: gene-set-enrichment-analysis
description: Use when a list of discriminatively selected marker genes needs functional/annotation
  validation against a curated tissue and cell-type gene-set library such as HuBMAP
  ASCT+B, or when comparing marker-selection methods by enrichment q values.
when_to_use_negative:
- Input is raw expression data rather than an already-selected marker gene list —
  marker selection (e.g., GreedyPC at coverage 8) must be performed first.
- The target annotation library is not a tissue/cell-type marker set compatible with
  Enrichr libraries (e.g., no ASCT+B-equivalent terms to aggregate q values over).
- The goal is deconvolution of pseudo-bulk mixtures rather than annotation-level validation
  of marker specificity.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: GSEApy
  role: Python interface that submits the marker list to the Enrichr API and retrieves
    enrichment terms, overlaps, p values, and adjusted q values
- name: Enrichr API
  role: Backend gene-set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022
    library
- name: phenotype-cover
  role: Produces the G-PC (GreedyPC) marker gene list at coverage 8 that serves as
    the enrichment input
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Numeric aggregation of q values at pair, tissue, and cell-type levels and
    fraction computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:72aefafe9bae44838f6e8ec2b12c027ee4ac937a7c2c06533f8530abd7cec88d
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-set-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates G-PC-selected marker genes by submitting the coverage-8 marker list to Enrichr (via GSEApy) against the HuBMAP ASCT+B marker library, then aggregates adjusted q values at tissue/cell-type-pair, tissue, and cell-type levels to benchmark against decision-tree markers.

## When to use

Use when a list of discriminatively selected marker genes needs functional/annotation validation against a curated tissue and cell-type gene-set library such as HuBMAP ASCT+B, or when comparing marker-selection methods by enrichment q values.

## When NOT to use

- Input is raw expression data rather than an already-selected marker gene list — marker selection (e.g., GreedyPC at coverage 8) must be performed first.
- The target annotation library is not a tissue/cell-type marker set compatible with Enrichr libraries (e.g., no ASCT+B-equivalent terms to aggregate q values over).
- The goal is deconvolution of pseudo-bulk mixtures rather than annotation-level validation of marker specificity.

## Inputs

- G-PC-selected HCA marker gene list (genes s in solution S with some phenotype j satisfying M_i,s >= M_j,s + 1 at coverage 8)
- Comparator marker gene lists (e.g., decision-tree-derived markers) for benchmarking
- Enrichr gene-set library HuBMAP_ASCTplusB_augmented_2022 (accessed via Enrichr API)

## Outputs

- Full enrichment table (term, overlap, p value, adjusted q value) written to gsea_enrichment_results.csv
- Aggregated q values per tissue/cell-type pair (42), per tissue (11), and per cell type (7)
- Fractions of comparisons where G-PC q values are lower than comparator q values

## How to apply

Start from the G-PC-selected HCA marker genes: at a fixed coverage of 8, for every phenotype i select from solution S all genes s for which there exists a phenotype j satisfying M_i,s >= M_j,s + 1. Submit this ranked marker list through GSEApy's enrichr interface to the Enrichr API, querying the HuBMAP_ASCTplusB_augmented_2022 gene-set library, and retrieve term, overlap, p value, and adjusted q value for all matched ASCT+B tissue and cell-type terms. Aggregate q values hierarchically — 42 tissue/cell-type pairs across 11 tissues and 7 cell types — and compare against q values from the decision-tree/comparator marker sets, counting the fraction of comparisons where G-PC achieves a lower q value. Rationale: GSEA against an independent, curated annotation library tests whether the discriminative markers recovered by the phenotype cover formulation are biologically specific rather than merely separable in the training data.

## Related tools

- **GSEApy** (Python interface that submits the marker list to the Enrichr API and retrieves enrichment terms, overlaps, p values, and adjusted q values)
- **Enrichr API** (Backend gene-set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022 library)
- **phenotype-cover** (Produces the G-PC (GreedyPC) marker gene list at coverage 8 that serves as the enrichment input) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Numeric aggregation of q values at pair, tissue, and cell-type levels and fraction computation)

## Examples

```
import gseapy as gp; enr = gp.enrichr(gene_list=marker_genes, gene_sets='HuBMAP_ASCTplusB_augmented_2022', outdir=None); enr.results.to_csv('gsea_enrichment_results.csv')
```

## Evaluation signals

- Reproduced fractions match the reported outcome: lower q values for 30/42 (71%) tissue/cell-type pairs, 6/11 (54%) tissues, and 3/7 (42%) cell types.
- The enrichment table contains term, overlap, p value, and adjusted q value columns for all matched ASCT+B terms and is written to gsea_enrichment_results.csv.
- Aggregation counts are consistent: 42 tissue/cell-type pairs map to exactly 11 tissues and 7 cell types.
- Every returned term is traceable to the HuBMAP_ASCTplusB_augmented_2022 library queried through GSEApy's enrichr interface.

## Limitations

- The biological analysis relied on computing the overlap with existing gene marker lists that may be incomplete.
- A more thorough biological analysis of the selected genes and their relation to each cell type was not performed in the paper.
- G-PC does not provide an optimal solution, so the marker list feeding the enrichment is an approximation of the phenotype cover optimum.

## Evidence

- [results] G-PC outperforms decision trees: [evidence span withheld — anchor status: not_found]
