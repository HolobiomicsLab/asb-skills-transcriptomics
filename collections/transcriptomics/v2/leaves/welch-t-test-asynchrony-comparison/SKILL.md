---
name: welch-t-test-asynchrony-comparison
description: Use when comparing whether peak-gene pairs ranked by a Granger-causal
  method show significantly longer chromatin-to-expression pseudotime lags than pairs
  ranked by Pearson or pseudocell correlation.
when_to_use_negative:
- Static or non-trajectory multimodal data with no meaningful pseudotime ordering,
  since lag computation depends on a cell-state DAG and diffusion pseudotime
- Peak-gene pairs outside the prioritized top/bottom 1% groups when the goal is to
  replicate the reported Fig. 2d,e group comparison
- Datasets where candidate pairs were assembled with the default 1 Mb cutoff instead
  of the 10x Multiome 250 kb cutoff, since lag statistics are not directly comparable
  to the reported result
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: Supplies Granger causal scores used to define the causally prioritized peak-gene
    pairs whose lags are compared; run via gridnet.run_gridnet on the DAG of cells
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15)
    and diffusion pseudotime over which lags are measured
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:1a87c7cb30ee4d48fc76586cfaefc16f94365a958ee5943aceecaa700bcafdbe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - Schema
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# welch-t-test-asynchrony-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical test comparing pseudotime lag ('asynchrony') distributions between GrID-Net-prioritized peak-gene pairs and correlation-prioritized pairs on the 10x Multiome corticogenesis dataset, using a Welch one-tailed t-test to establish the cell-state parallax effect (P < 10^-26).

## When to use

Use when comparing whether peak-gene pairs ranked by a Granger-causal method show significantly longer chromatin-to-expression pseudotime lags than pairs ranked by Pearson or pseudocell correlation.

## When NOT to use

- Static or non-trajectory multimodal data with no meaningful pseudotime ordering, since lag computation depends on a cell-state DAG and diffusion pseudotime
- Peak-gene pairs outside the prioritized top/bottom 1% groups when the goal is to replicate the reported Fig. 2d,e group comparison
- Datasets where candidate pairs were assembled with the default 1 Mb cutoff instead of the 10x Multiome 250 kb cutoff, since lag statistics are not directly comparable to the reported result

## Inputs

- Per-pair GrID-Net Granger causal scores for candidate peak-gene pairs (results_df from run_gridnet)
- Per-pair Pearson correlation and pseudocell correlation rankings of the same candidate pairs
- Diffusion pseudotime values per cell (from construct_dag, pseudotime_algo='dpt')
- Log(1+x)-normalized accessibility and expression matrices for computing per-pair pseudotime profiles of maximal accessibility and maximal expression

## Outputs

- asynchrony_lag_statistics.csv with per-pair lag values, method rankings, group assignments, test statistic, and Welch one-tailed P value
- Welch one-tailed t-test statistic and P value comparing lag distributions (expected P < 10^-26)
- Named example pairs such as the CBWD5 promoter and TRAF1 13.5-kb enhancer

## How to apply

After scoring candidate peak-gene pairs (250 kb TSS distance cutoff for 10x Multiome) with GrID-Net (via run_gridnet on a Schema/DPT-derived cell-state DAG) and with baselines (Pearson correlation and pseudocell correlation), select the top and bottom 1% of pairs by each method's score (n=14,802 pairs per group). For each pair compute the lag as the separation between the pseudotime value at which the peak's accessibility is maximal and the pseudotime value at which the gene's expression is maximal. Compare lag distributions between GrID-Net-prioritized and correlation-prioritized groups with a Welch one-tailed t-test, testing the directional hypothesis that GrID-Net pairs exhibit longer lags, since correlation-based prioritization overlooks the inherent causal time lag in gene regulation. Write per-pair lags, rankings, group assignments, test statistic, and P value to asynchrony_lag_statistics.csv.

## Related tools

- **GrID-Net** (Supplies Granger causal scores used to define the causally prioritized peak-gene pairs whose lags are compared; run via gridnet.run_gridnet on the DAG of cells) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15) and diffusion pseudotime over which lags are measured) — https://github.com/rs239/schema

## Examples

```
stats.ttest_ind(gridnet_lags, pearson_lags, equal_var=False, alternative='greater')
```

## Evaluation signals

- Welch one-tailed t-test on lag distributions returns P < 10^-26 for GrID-Net pairs vs. both Pearson and pseudocell correlation pairs
- Each comparison group contains exactly the top and bottom 1% of pairs (n=14,802 pairs per group)
- Lag for each pair equals the pseudotime separation between maximal peak accessibility and maximal gene expression, and lags for GrID-Net pairs are predominantly longer
- The CBWD5 promoter and TRAF1 13.5-kb enhancer examples are recoverable from asynchrony_lag_statistics.csv
- Candidate pairs were assembled with the 10x Multiome 250 kb distance threshold and gene/peak filters of 1% and 0.1% of cells, respectively

## Limitations

- Correlational analyses between scATAC-seq and scRNA-seq overlook the inherent causal time lag in gene regulation, so baseline comparisons are the point of the test rather than a validated null model
- Relies on multiomic single-cell data from matching tissue undergoing a dynamic process to provide pseudotemporal context
- Inferred causal interactions are likely indirect and mediated by additional biophysical events, such as transcription factor binding

## Evidence

- [results] Lag definition: [evidence span withheld — anchor status: not_found]
- [discussion] DAG construction: [evidence span withheld — anchor status: not_found]
