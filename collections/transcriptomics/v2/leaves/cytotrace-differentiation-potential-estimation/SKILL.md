---
name: cytotrace-differentiation-potential-estimation
description: Use when a normalized scRNA-seq count matrix (genes as rows, cells as
  columns) with cell-type annotations must be scored for stemness/differentiation
  potential per cell, e.g. to identify stem/progenitor populations or benchmark against
  ORIGINS activity and LandSCENT SR scores.
when_to_use_negative:
- The input is not a raw/normalized single-cell expression matrix but an already-aggregated
  feature table or bulk-RNA profile, since CytoTRACE scores are computed per cell.
- The goal is to score the activity of a PPIN associated with a biological process
  other than differentiation — the article points to ORIGINS2/PyOrigins with build_ppin()
  for custom gene-set PPINs instead.
- 'Memory is severely constrained: the article reports CytoTRACE requires more RAM
  memory than ORIGINS.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CytoTRACE
  role: Computes per-cell differentiation-potential scores used as a benchmark comparator
    to LandSCENT SR and ORIGINS activity across the breast, colon, hematopoietic and
    lung datasets
- name: ORIGINS
  role: Provides activity(expression_matrix, differentiation_edges) and the bundled
    differentiation PPIN used to compute the comparator pluripotency activity scores
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Provides the Signal Rank (SR) pluripotency score compared against CytoTRACE
    on the same expression matrix
- name: R
  role: Runtime environment in which ORIGINS, LandSCENT and CytoTRACE scoring, and
    per-cell-type aggregation are executed
- name: dplyr
  role: Used to aggregate mean scores per annotated cell type and rank cell types
    under each score
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:d9c561c8e418340b051559f52a9fd3241f84ffc7acb27e59024efaa55bf14f3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - dplyr
  - remotes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cytotrace-differentiation-potential-estimation

## Summary

Benchmark/apply CytoTRACE, a published differentiation-potential scoring tool (Gulati et al., 2020), on a gene-by-cell scRNA-seq expression matrix to rank cell types by pluripotency, as done in the ORIGINS paper where CytoTRACE served as a comparator to LandSCENT SR and ORIGINS differentiation activity scores.

## When to use

Use when a normalized scRNA-seq count matrix (genes as rows, cells as columns) with cell-type annotations must be scored for stemness/differentiation potential per cell, e.g. to identify stem/progenitor populations or benchmark against ORIGINS activity and LandSCENT SR scores.

## When NOT to use

- The input is not a raw/normalized single-cell expression matrix but an already-aggregated feature table or bulk-RNA profile, since CytoTRACE scores are computed per cell.
- The goal is to score the activity of a PPIN associated with a biological process other than differentiation — the article points to ORIGINS2/PyOrigins with build_ppin() for custom gene-set PPINs instead.
- Memory is severely constrained: the article reports CytoTRACE requires more RAM memory than ORIGINS.

## Inputs

- Normalized scRNA-seq expression matrix (genes as rows, cells as columns), e.g. GEO GSE161383 lung dataset
- Per-cell cell-type annotations (e.g. ~30 lung cell types including AT2/club-like cells)
- Optionally, comparator per-cell scores: LandSCENT Signal Rank (SR) and ORIGINS diff_activity from activity(expression_matrix, differentiation_edges)

## Outputs

- Per-cell CytoTRACE differentiation-potential scores
- Mean score per cell type with ranking across cell types (e.g. results table such as lung_stemness_scores.csv)
- Cross-method rank comparison (CytoTRACE vs LandSCENT SR vs ORIGINS activity)

## How to apply

Run CytoTRACE on the same expression matrix used for the comparator scores: a normalized gene-by-cell matrix from a dataset such as GEO GSE161383 (lung), formatted with genes as rows and cells as columns, together with per-cell cell-type annotations. Compute per-cell CytoTRACE scores alongside LandSCENT Signal Rank (SR) and ORIGINS differentiation activity (computed via activity(expression_matrix, differentiation_edges) with the PPIN bundled in the ORIGINS package data directory). Aggregate mean scores per annotated cell type (e.g. with dplyr across ~30 lung cell types), rank cell types under each score, and compare the orderings — in the lung sample, AT2/club-like cells held the highest mean SR and ORIGINS activity, and cells of interest fell within the top 8 mean ranks for SR, CytoTRACE and activity. Treat CytoTRACE's ranking as one of several pluripotency estimates; agreement among methods on top-ranked stem/progenitor types is the correctness criterion, and the article notes CytoTRACE requires more RAM memory than ORIGINS.

## Related tools

- **CytoTRACE** (Computes per-cell differentiation-potential scores used as a benchmark comparator to LandSCENT SR and ORIGINS activity across the breast, colon, hematopoietic and lung datasets)
- **ORIGINS** (Provides activity(expression_matrix, differentiation_edges) and the bundled differentiation PPIN used to compute the comparator pluripotency activity scores) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Provides the Signal Rank (SR) pluripotency score compared against CytoTRACE on the same expression matrix)
- **R** (Runtime environment in which ORIGINS, LandSCENT and CytoTRACE scoring, and per-cell-type aggregation are executed)
- **dplyr** (Used to aggregate mean scores per annotated cell type and rank cell types under each score)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- Per-cell scores are produced for every column of the gene-by-cell expression matrix (one score per cell).
- Mean per-cell-type ranks place expected stem/progenitor types within the top 8 mean ranks for SR, CytoTRACE and ORIGINS activity (as observed on the GSE161383 lung sample).
- The top-ranked cell type by mean score is biologically coherent — in the lung sample, AT2/club-like cells attained the highest mean SR and ORIGINS activity scores.
- Results table (e.g. lung_stemness_scores.csv) contains one ranked row per annotated cell type for all computed scores.
- Cross-method comparison confirms ORIGINS activity orders cell types by expected pluripotency while a mismatched comparator (HVF-derived activity) fails to do so.

## Limitations

- CytoTRACE requires more RAM memory than ORIGINS and is less efficient than the ORIGINS implementation.
- Score orderings are method-dependent: computing activity over an HVF-derived network failed to order cell types by expected pluripotency, so network/score choice affects results.
- Scores are validated on healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); behavior on diseased or non-human tissue is not established here.

## Evidence

- [abstract] CytoTRACE (Gulati, et al., 2020): [evidence span withheld — anchor status: not_found]
- [abstract] benchmark comparison: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [abstract] datasets benchmarked: [evidence span withheld — anchor status: not_found]
- [readme] activity call: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [intro] differentiation PPIN bundled: "The differentiation PPIN is loaded within the package in the data directory."
- [supplementary] lung dataset accession: "GSE161383 for lung cells"
- [intro] purpose statement: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
