---
name: landscent-signal-rank-computation
description: Use when scRNA-seq count/normalized expression matrices (genes as rows,
  cells as columns) need per-cell stemness/pluripotency ranking, e.g. to select stem/progenitor
  origins for trajectory inference or to benchmark against ORIGINS activity scores.
when_to_use_negative:
- When the goal is to score PPIN activity for a biological process other than differentiation
  — the extended ORIGINS2 package or PyOrigins with a custom build_ppin network is
  the appropriate tool
- 'When RAM memory is severely constrained: the article reports ORIGINS requires less
  RAM memory than LandSCENT and CytoTRACE'
- When only a highly-variable-feature (HVF)-derived network is available as scaffold
  — this comparator failed to order lung cell types by expected pluripotency
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LandSCENT
  role: Computes the Signal Rank (SR) pluripotency score per cell from the expression
    matrix; benchmarked against ORIGINS activity
- name: CytoTRACE
  role: Provides a third per-cell pluripotency score for cross-method comparison of
    cell-type rankings
- name: ORIGINS
  role: Computes the differentiation-PPIN activity score used as the primary comparator
    to SR scores
  repo: https://github.com/danielasenraoka/ORIGINS
- name: dplyr
  role: Aggregates mean scores per annotated cell type and ranks cell types
- name: R
  role: Runtime environment in which LandSCENT, CytoTRACE and ORIGINS are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:0ed62846f6456f16469eb1de02a13a12b0013445f173c4599f59e56085a0170d
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# landscent-signal-rank-computation

## Summary

Computes LandSCENT Signal Rank (SR) pluripotency scores per cell from a gene-by-cell scRNA-seq expression matrix, as benchmarked in Senra et al. (2022) against ORIGINS differentiation activity and CytoTRACE on breast, colon, hematopoietic (GSE117498) and lung (GSE161383) datasets.

## When to use

Use when scRNA-seq count/normalized expression matrices (genes as rows, cells as columns) need per-cell stemness/pluripotency ranking, e.g. to select stem/progenitor origins for trajectory inference or to benchmark against ORIGINS activity scores.

## When NOT to use

- When the goal is to score PPIN activity for a biological process other than differentiation — the extended ORIGINS2 package or PyOrigins with a custom build_ppin network is the appropriate tool
- When RAM memory is severely constrained: the article reports ORIGINS requires less RAM memory than LandSCENT and CytoTRACE
- When only a highly-variable-feature (HVF)-derived network is available as scaffold — this comparator failed to order lung cell types by expected pluripotency

## Inputs

- Normalized scRNA-seq expression matrix with genes as rows and cells as columns (e.g. GEO GSE161383 lung sample)
- Cell-type annotations per cell (e.g. ~30 cell types in the lung dataset)
- differentiation_edges PPIN data frame bundled in the ORIGINS package data directory (used as the ORIGINS comparator score)

## Outputs

- Per-cell LandSCENT Signal Rank (SR) score vector
- Per-cell-type mean scores and cell-type rankings (e.g. lung_stemness_scores.csv)
- Comparator CytoTRACE and ORIGINS activity scores per cell for cross-method ranking

## How to apply

Format the normalized expression matrix with genes as rows and cells as columns, as required by LandSCENT and ORIGINS alike. Run the LandSCENT Signal Rank computation on the full expression matrix and, in parallel, compute CytoTRACE scores and ORIGINS differentiation activity via activity(expression_matrix, differentiation_edges) using the differentiation PPIN bundled in the ORIGINS package data directory. Aggregate mean scores per annotated cell type with dplyr and rank cell types under each score. In the lung dataset (GSE161383, D122, ~30 cell types), verify that AT2/club-like cells obtain the highest mean SR and activity, and that cells of interest fall within the top 8 mean ranks for SR, CytoTRACE and activity. Note that a HVF-derived network comparator is expected to fail to order cell types by expected pluripotency, which is itself a diagnostic signal.

## Related tools

- **LandSCENT** (Computes the Signal Rank (SR) pluripotency score per cell from the expression matrix; benchmarked against ORIGINS activity)
- **CytoTRACE** (Provides a third per-cell pluripotency score for cross-method comparison of cell-type rankings)
- **ORIGINS** (Computes the differentiation-PPIN activity score used as the primary comparator to SR scores) — https://github.com/danielasenraoka/ORIGINS
- **dplyr** (Aggregates mean scores per annotated cell type and ranks cell types)
- **R** (Runtime environment in which LandSCENT, CytoTRACE and ORIGINS are executed)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- In the lung sample (GSE161383), AT2/club-like cells must have the highest average SR (LandSCENT) and highest average ORIGINS activity among ~30 cell types
- All cells of interest fall within the top 8 mean ranks for SR, CytoTRACE and ORIGINS activity scores
- The HVF-derived activity comparator must fail to order cell types by expected pluripotency, confirming the full differentiation PPIN is required
- The expression matrix must be oriented genes-as-rows/cells-as-columns before calling activity()/LandSCENT, matching the ORIGINS function contract
- Results table written to lung_stemness_scores.csv contains mean SR, CytoTRACE and activity values per cell type

## Limitations

- ORIGINS was benchmarked only on four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); SR score behavior outside healthy human tissues is not covered
- Computing activity over an HVF-derived network instead of the full differentiation PPIN fails to order cell types by expected pluripotency
- ORIGINS is reported as more efficient than LandSCENT and less RAM-demanding than LandSCENT and CytoTRACE, so SR computation may be the resource bottleneck on large datasets

## Evidence

- [abstract] LandSCENT and CytoTRACE as benchmark comparators: [evidence span withheld — anchor status: not_found]
- [methods] ORIGINS activity function contract: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] README: expression matrix orientation: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [readme] README: differentiation PPIN bundled in package: "The differentiation PPIN is loaded within the package in the data directory."
- [abstract] Efficiency comparison motivating method choice: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [intro] ORIGINS purpose for trajectory origins: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
