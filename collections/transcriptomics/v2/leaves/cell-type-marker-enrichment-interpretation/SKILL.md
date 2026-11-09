---
name: cell-type-marker-enrichment-interpretation
description: Use when you have a gene marker list selected from scRNA-seq phenotypes
  and need to verify it recovers the correct tissues and cell types by enrichment
  against a curated reference set such as HuBMAP_ASCTplusB_augmented_2022.
when_to_use_negative:
- The gene list has not been selected via phenotype cover (G-PC/CEM-PC) at the documented
  coverage threshold — the coverage-8 extraction rule is specific to this workflow
- The goal is deconvolution or classification accuracy rather than enrichment-based
  validation of marker specificity
- No curated reference gene set matching the tissues/cell types of the dataset exists
  (the analysis relied on gene marker lists that may be incomplete)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: GSEApy
  role: Python interface to submit the marker list to the Enrichr API and retrieve
    enrichment terms, overlaps, p values, and adjusted q values
- name: Enrichr API
  role: Backend gene set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022
    library
- name: phenotype-cover
  role: Produces the G-PC marker gene list (GreedyPC select at coverage 8) that is
    the input to the enrichment validation
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Numeric aggregation of q values across tissue/cell-type-pair, tissue, and
    cell-type levels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:7d9b617ea07b65adda9bb1c119f2ebc5bdcfa90a0d0df9524db4d1d83f0cd4ce
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
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-type-marker-enrichment-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a discriminative marker set (e.g., G-PC-selected HCA markers at coverage 8) by gene set enrichment analysis against the HuBMAP ASCT+B marker library via Enrichr/GSEApy, then aggregates adjusted q values at the tissue, cell-type, and tissue/cell-type-pair levels to compare marker-selection methods.

## When to use

Use when you have a gene marker list selected from scRNA-seq phenotypes and need to verify it recovers the correct tissues and cell types by enrichment against a curated reference set such as HuBMAP_ASCTplusB_augmented_2022.

## When NOT to use

- The gene list has not been selected via phenotype cover (G-PC/CEM-PC) at the documented coverage threshold — the coverage-8 extraction rule is specific to this workflow
- The goal is deconvolution or classification accuracy rather than enrichment-based validation of marker specificity
- No curated reference gene set matching the tissues/cell types of the dataset exists (the analysis relied on gene marker lists that may be incomplete)

## Inputs

- G-PC-selected HCA marker gene list (genes from solution S at coverage 8 satisfying M_i,s >= M_j,s + 1)
- HuBMAP_ASCTplusB_augmented_2022 Enrichr gene set library (via Enrichr API)
- Comparator marker sets (e.g., decision-tree-derived markers) for q-value comparison

## Outputs

- gsea_enrichment_results.csv with columns: term, overlap, p value, adjusted q value for each matched ASCT+B term
- Aggregated q values at the tissue/cell-type-pair level (42 pairs), tissue level (11 tissues), and cell-type level (7 cell types)
- Fraction of comparisons with lower q values versus the reference/comparator marker sets

## How to apply

Load the marker gene list produced by the phenotype-cover selection sub-task, i.e., genes s in the solution S for which some phenotype j satisfies M_i,s >= M_j,s + 1, extracted at a fixed coverage of 8. Submit the list through GSEApy's enrichr interface, which queries the Enrichr API using the HuBMAP_ASCTplusB_augmented_2022 gene set library. Retrieve the enrichment results (term, overlap, p value, adjusted q value) for all matched ASCT+B tissue and cell-type terms. Aggregate q values at three levels: 42 tissue/cell-type pairs, 11 tissues, and 7 cell types. Compare against comparator marker sets (e.g., decision trees, mRMR) by computing the fraction of comparisons where the G-PC markers obtain lower q values, then write the full enrichment table to gsea_enrichment_results.csv.

## Related tools

- **GSEApy** (Python interface to submit the marker list to the Enrichr API and retrieve enrichment terms, overlaps, p values, and adjusted q values)
- **Enrichr API** (Backend gene set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022 library)
- **phenotype-cover** (Produces the G-PC marker gene list (GreedyPC select at coverage 8) that is the input to the enrichment validation) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Numeric aggregation of q values across tissue/cell-type-pair, tissue, and cell-type levels)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=markers_pc_cov8, gene_sets='HuBMAP_ASCTplusB_augmented_2022', outdir=None); enr.results[['Term','Overlap','P-value','Adjusted P-value']].to_csv('gsea_enrichment_results.csv', index=False)
```

## Evaluation signals

- Reproduced fractions match the reported outcome: lower q values for 30/42 (71%) tissue/cell-type pairs, 6/11 (54%) tissues, and 3/7 (42%) cell types
- Enrichment table contains the required columns (term, overlap, p value, adjusted q value) for all matched ASCT+B tissue and cell-type terms
- Aggregation counts are consistent with the expected structure: 42 tissue/cell-type pairs, 11 tissues, 7 cell types
- gsea_enrichment_results.csv is written and q values are adjusted (not raw p values) before comparison
- Marker list is confirmed to contain genes s for which some phenotype j satisfies M_i,s >= M_j,s + 1 at coverage 8 before enrichment

## Limitations

- The biological analysis relied on computing the overlap with existing gene marker lists (HuBMAP ASCT+B) that may be incomplete
- G-PC does not provide an optimal solution, so the marker set fed to the enrichment step is an approximation
- Enrichment validation depends on coverage of the query markers by the reference library; unmatched terms provide no signal

## Evidence

- [results] GSEA enrichment against HuBMAP ASCT+B markers: "We fix a coverage of 8, and for every phenotype i, we select from the solution S all those genes s for which there exists some phenotype j satisfying Mi,s ≥ Mj,s + 1"
- [results] GSEA performed with HuBMAP ASCT+B marker set: "We next performed gene set enrichment analysis (GSEA)47 using the HuBMAP ASCT + B marker set48"
- [results] G-PC outperforms decision trees in GSEA validation: [evidence span withheld — anchor status: not_found]
- [discussion] Marker lists may be incomplete: "The biological analysis relied on computing the overlap with existing gene marker lists that may be incomplete."
- [readme] GreedyPC select at coverage: ">>> features = gpc.select(100)  # coverage of 100"
