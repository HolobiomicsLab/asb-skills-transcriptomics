---
name: enrichr-api-gene-list-query
description: Use when a ranked or selected gene marker list (e.g., G-PC-selected HCA
  markers at coverage 8) needs external biological validation against a tissue/cell-type
  marker library such as HuBMAP ASCT+B, with q-value-based comparison to a competing
  method.
when_to_use_negative:
- The input is a raw expression matrix rather than an already-selected marker gene
  list; marker selection via GreedyPC at the desired coverage must happen first
- The validation target is a general functional ontology (e.g., GO/KEGG) rather than
  a tissue/cell-type marker library like HuBMAP ASCT+B
- The goal is differential-expression testing per gene rather than gene-set enrichment
  for a curated marker list
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSEApy
  role: Provides the enrichr interface through which the marker list is submitted
    to the Enrichr API
- name: Enrichr API
  role: Backend gene-set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022
    library
- name: GreedyPC (phenotype-cover)
  role: Produces the upstream G-PC marker list at coverage 8 that is submitted for
    GSEA validation
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Dependency of phenotype-cover used in downstream aggregation/comparison of
    q values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:1830edbcafb5ce2bbf47ffb3bbdb6f572af30683062730a41b1e851453ece116
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
  - GreedyPC (phenotype-cover)
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# enrichr-api-gene-list-query

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates discriminative marker gene lists by submitting them to the Enrichr API via GSEApy against the HuBMAP_ASCTplusB_augmented_2022 library, then aggregating adjusted q values at tissue, cell-type, and tissue/cell-type-pair levels for method comparison.

## When to use

Use when a ranked or selected gene marker list (e.g., G-PC-selected HCA markers at coverage 8) needs external biological validation against a tissue/cell-type marker library such as HuBMAP ASCT+B, with q-value-based comparison to a competing method.

## When NOT to use

- The input is a raw expression matrix rather than an already-selected marker gene list; marker selection via GreedyPC at the desired coverage must happen first
- The validation target is a general functional ontology (e.g., GO/KEGG) rather than a tissue/cell-type marker library like HuBMAP ASCT+B
- The goal is differential-expression testing per gene rather than gene-set enrichment for a curated marker list

## Inputs

- G-PC-selected HCA marker gene list (genes extracted from solution S at coverage 8)
- HuBMAP_ASCTplusB_augmented_2022 gene set library (accessed via Enrichr)

## Outputs

- Enrichment table with term, overlap, p value, and adjusted q value for matched ASCT+B terms (written to gsea_enrichment_results.csv)
- Aggregated q values at tissue/cell-type-pair (42 pairs), tissue (11 tissues), and cell-type (7 cell types) levels
- Fraction of comparisons with lower q values versus decision trees

## How to apply

First load the marker gene list produced by the phenotype-cover marker-selection sub-task, taking all genes s in solution S for which some phenotype j satisfies M_i,s >= M_j,s + 1 at a fixed coverage of 8. Submit this list through GSEApy's enrichr interface to the Enrichr API, querying the HuBMAP_ASCTplusB_augmented_2022 gene set library. Retrieve per-term results (term, overlap, p value, adjusted q value) for all matched ASCT+B tissue and cell-type terms. Aggregate q values at three levels: 42 tissue/cell-type pairs, 11 tissues, and 7 cell types. Compare these q values against the comparator method (decision trees) and compute the fraction of comparisons with lower q values for the G-PC markers.

## Related tools

- **GSEApy** (Provides the enrichr interface through which the marker list is submitted to the Enrichr API)
- **Enrichr API** (Backend gene-set enrichment service queried with the HuBMAP_ASCTplusB_augmented_2022 library)
- **GreedyPC (phenotype-cover)** (Produces the upstream G-PC marker list at coverage 8 that is submitted for GSEA validation) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Dependency of phenotype-cover used in downstream aggregation/comparison of q values)

## Examples

```
import gsepy as gp; enr = gp.enrichr(gene_list=markers, gene_sets='HuBMAP_ASCTplusB_augmented_2022', outdir='gsea_enrichment_results')
```

## Evaluation signals

- Reproduced fraction of tissue/cell-type pairs with lower q values matches 30/42 (71%)
- Reproduced fractions for tissues and cell types match 6/11 (54%) and 3/7 (42%) respectively
- Enrichment output table contains the expected columns: term, overlap, p value, adjusted q value
- Q values are aggregated over exactly 42 tissue/cell-type pairs, 11 tissues, and 7 cell types
- Full enrichment table successfully written to gsea_enrichment_results.csv

## Limitations

- The biological analysis relied on computing overlap with existing gene marker lists that may be incomplete
- G-PC does not provide an optimal solution, so the marker list fed to Enrichr may not be the minimal achievable set

## Evidence

- [full_text] Reproduced GSEA validation outcome: [evidence span withheld — anchor status: not_found]
