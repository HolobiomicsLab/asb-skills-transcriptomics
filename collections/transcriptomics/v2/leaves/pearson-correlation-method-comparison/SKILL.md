---
name: pearson-correlation-method-comparison
description: Use when multiple gene-set activity scoring methods (e.g. ORIGINS2 network
  activity, AUCell, mean expression) have been run on the same gene sets and you need
  to quantify their agreement with Pearson correlations and summary means.
when_to_use_negative:
- You need per-cell normalized activity (normalization=TRUE) rather than raw edge-sum
  activity for cross-method comparison on equal footing
- Your gene sets are not shared across all methods being compared, since Pearson correlations
  must be computed over the same BP sets
- You want cell-type label assignment itself rather than agreement/benchmarking of
  scoring methods
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS2
  role: computes per-cell PPIN activity scores via activity(expression_matrix, adjacency_edges,
    FALSE); installed from GitHub
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: required to install ORIGINS2 from GitHub via remotes::install_github
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: data manipulation of the correlation table and Ngenes annotations
- name: magrittr
  role: pipe operator used with dplyr in the correlation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:ed824337a00d0c1adba0b32a36f037338d4f150bdb47ed41007faed9e4ccd5b5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS2
  - R
  - dplyr
  - magrittr
  - remotes
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pearson-correlation-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark ORIGINS2 PPIN activity scores against AUCell and mean-expression (Average) scoring by computing pairwise Pearson correlation coefficients across biological-process gene sets, as done for 17 BPs on a TNBC single-cell sample.

## When to use

Use when multiple gene-set activity scoring methods (e.g. ORIGINS2 network activity, AUCell, mean expression) have been run on the same gene sets and you need to quantify their agreement with Pearson correlations and summary means.

## When NOT to use

- You need per-cell normalized activity (normalization=TRUE) rather than raw edge-sum activity for cross-method comparison on equal footing
- Your gene sets are not shared across all methods being compared, since Pearson correlations must be computed over the same BP sets
- You want cell-type label assignment itself rather than agreement/benchmarking of scoring methods

## Inputs

- normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g. the TNBC sample matrix
- PPIN adjacency edge data frames for 17 biological processes shipped with ORIGINS2 (e.g. differentiation_edges, cell_cycle_edges)
- AUCell per-cell scores computed on the same BP gene sets
- mean-expression (Average) per-cell scores computed on the same BP gene sets

## Outputs

- correlation table of pairwise Pearson rho per BP with Ngenes annotation
- correlation_summary.csv with mean rho per method pair overall and for the Ngenes>200 subset

## How to apply

Install ORIGINS2 via remotes::install_github("danielasenraoka/ORIGINS2"), load it with dplyr and magrittr, and load the normalized TNBC expression matrix (rows = genes, columns = cells) plus the PPIN adjacency edge sets for the biological processes shipped with the package. For each BP gene set, run activity(expression_matrix, adjacency_edges, FALSE) so ORIGINS2 activity is the sum of all PPIN edges, and in parallel compute AUCell and mean-expression (Average) scores on the same gene sets. Compute pairwise Pearson correlation coefficients between the three scoring methods across the 17 BP sets, annotating each BP with its Ngenes. Derive summary metrics (mean rho per method pair overall and for the Ngenes>200 subset) and write the table to correlation_summary.csv. In the TNBC sample the expected agreement pattern was ORIGINS2-AUCell mean rho = 0.59, ORIGINS2-Average = 0.79, AUCell-Average = 0.66.

## Related tools

- **ORIGINS2** (computes per-cell PPIN activity scores via activity(expression_matrix, adjacency_edges, FALSE); installed from GitHub) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (required to install ORIGINS2 from GitHub via remotes::install_github) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (data manipulation of the correlation table and Ngenes annotations)
- **magrittr** (pipe operator used with dplyr in the correlation workflow)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- All three method pairs yield positive Pearson correlations across the 17 BP gene sets
- Mean rho values fall near the reported pattern: ORIGINS2-AUCell ~0.59, ORIGINS2-Average ~0.79, AUCell-Average ~0.66
- Each BP row in correlation_summary.csv carries a valid Ngenes annotation and the Ngenes>200 subset is reported separately
- ORIGINS2 activity was computed with normalization=FALSE, i.e. the output vector equals the sum of all PPIN edges

## Limitations

- ORIGINS2 activity with normalization=FALSE is the sum of all PPIN edges, so raw scores depend on the number of edges per PPIN; only subset analyses like Ngenes>200 partly mitigate size effects
- Correlations were measured across gene sets on a single TNBC sample, not across cells or datasets, so the mean rho values may not generalize
- The comparison covers only the biological-process PPINs shipped with the package (17 BP used in this benchmark)

## Evidence
