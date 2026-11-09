---
name: gene-set-size-stratified-summary-statistics
description: Use when comparing gene-set scoring methods across multiple BP sets and
  you need overall plus Ngenes-stratified (e.g., Ngenes>200) summary statistics such
  as mean Pearson correlation.
when_to_use_negative:
- Input is already a summary statistics table with no per-gene-set scores — no stratification
  can be recomputed
- Only a single gene set is available, making size-stratified or across-set correlation
  summaries meaningless
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: R package providing activity() to compute per-cell PPIN activity for BP gene
    sets, whose outputs feed the stratified correlation summaries
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub before computing scores
- name: dplyr
  role: Attached to manipulate the correlation table and compute grouped (Ngenes-stratified)
    summary metrics
- name: magrittr
  role: Pipe operator support for dplyr-based summary computations in R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:a1ba69f9fd1f8b7a57a7f9dec7ed609a9f26de12783c6a903f4c5d5c20bd450e
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Gene-set-size-stratified summary statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives summary metrics (e.g., mean Pearson correlations between scoring methods) across biological-process gene sets, and stratifies them by gene-set size (Ngenes), as done when comparing ORIGINS2, AUCell, and Average scores on the 17 BP PPIN sets in the TNBC single-cell sample.

## When to use

Use when comparing gene-set scoring methods across multiple BP sets and you need overall plus Ngenes-stratified (e.g., Ngenes>200) summary statistics such as mean Pearson correlation.

## When NOT to use

- Input is already a summary statistics table with no per-gene-set scores — no stratification can be recomputed
- Only a single gene set is available, making size-stratified or across-set correlation summaries meaningless

## Inputs

- Per-gene-set, per-cell score matrices from multiple scoring methods (ORIGINS2 activity, AUCell, Average) on the same BP gene sets
- BP gene-set annotation table with Ngenes per set (17 biological process PPIN edge sets from ORIGINS2)
- Pairwise Pearson correlation table across gene sets

## Outputs

- correlation_summary.csv containing per-BP pairwise Pearson correlations with Ngenes annotation
- Summary metrics: mean rho for each method pair overall and for the Ngenes>200 subset

## How to apply

After computing per-gene-set scores for each method (e.g., ORIGINS2 activity via activity(expression_matrix, adjacency_edges, FALSE), AUCell, and mean-expression Average on the same BP gene sets), compute pairwise Pearson correlation coefficients across the gene sets and annotate each BP with its Ngenes. Then derive summary metrics: mean rho for each method pair overall, and repeat restricted to the Ngenes>200 subset to control for gene-set size effects. Write the correlation table and summary metrics to a file such as correlation_summary.csv. The rationale is that larger PPIN gene sets can behave differently under edge-sum activity scoring, so size-stratified summaries reveal whether agreement between methods (e.g., ORIGINS2-Average = 0.79 vs ORIGINS2-AUCell = 0.59) holds regardless of set size.

## Related tools

- **ORIGINS2** (R package providing activity() to compute per-cell PPIN activity for BP gene sets, whose outputs feed the stratified correlation summaries) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub before computing scores)
- **dplyr** (Attached to manipulate the correlation table and compute grouped (Ngenes-stratified) summary metrics)
- **magrittr** (Pipe operator support for dplyr-based summary computations in R)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- All pairwise correlations across the 17 BP sets are positive, with mean rho matching reported values (ORIGINS2-AUCell = 0.59, ORIGINS2-Average = 0.79, AUCell-Average = 0.66)
- Each BP row in correlation_summary.csv carries a valid Ngenes annotation and Pearson r within [-1, 1]
- The Ngenes>200 subset mean rho differs from (or is reported alongside) the overall mean, confirming stratification was applied
- Output file contains both the per-BP correlation table and the summary metrics block

## Limitations

- Normalization choice in activity() matters: with FALSE the output is the raw sum of all PPIN edges, so larger gene sets yield larger activities and can dominate unstratified summaries
- Only 17 BP gene sets shipped with ORIGINS2 are used, so subset-level means (e.g., Ngenes>200) rest on few points
- Pearson correlation assumes linear agreement between scoring methods

## Evidence
