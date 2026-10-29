---
name: rna-seq-coverage-variance-stabilization
description: Use when raw base-level RNA-seq coverage pileup matrices are skewed and
  must be variance-stabilized (log-transform with tuned pseudo-count) prior to SCISSOR
  global/local shape change detection.
when_to_use_negative:
- Input is already normalized/log-transformed expression values (e.g., normalized
  RSEM values used for gene expression filtering), not base-level coverage pileup
  data
- The analysis targets gene-level differential expression rather than base-resolution
  coverage shape changes, where SCISSOR's pre-processing is not the appropriate transform
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: provides logtransform_data (log shift selection/transformation), scale.factor
    (skewness statistic), and build_pileup (input pileup construction) for variance
    stabilization of base-level coverage
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:8d6cdd6a2ae22bc59082ccc3952460bf7b2df9e2d6a418e648ff3d869415662c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# rna-seq-coverage-variance-stabilization

## Summary

Pre-processes base-level RNA-seq coverage pileup data by base-level normalization and a logarithmic transformation whose shift (pseudo-count) parameter is chosen automatically to minimize residual skewness, yielding more symmetric distributions of the SCISSOR test statistic before shape change detection.

## When to use

Use when raw base-level RNA-seq coverage pileup matrices are skewed and must be variance-stabilized (log-transform with tuned pseudo-count) prior to SCISSOR global/local shape change detection.

## When NOT to use

- Input is already normalized/log-transformed expression values (e.g., normalized RSEM values used for gene expression filtering), not base-level coverage pileup data
- The analysis targets gene-level differential expression rather than base-resolution coverage shape changes, where SCISSOR's pre-processing is not the appropriate transform

## Inputs

- base-level RNA-seq coverage pileup matrix (e.g., TOY_data pileup from SCISSOR)
- param.grid of candidate log shift (pseudo-count) values

## Outputs

- log-transformed, variance-stabilized pileup matrix at the selected logshiftVal
- table of candidate shift values versus overall mean skewness (logshift_grid_results.csv) with the selected minimizing shift

## How to apply

Load the base-level pileup matrix (e.g., from SCISSOR's TOY_data, built via build_pileup) and construct a grid of candidate log shift (pseudo-count) values spanning small-to-large additions applied before log-transformation. For each candidate, run logtransform_data with logshiftVal set to the candidate, then compute sample scale factors with scale.factor and aggregate them into a mean overall skewness statistic quantifying residual asymmetry of the transformed coverage. Select the shift value attaining the minimum mean skewness across the grid and record the shift-versus-skewness table (e.g., logshift_grid_results.csv). This step sits upstream of SCISSOR's two-step GSC/LSC procedure: normalization and variance stabilization are performed before the main shape change detection so that the test statistic's distribution is more symmetrical.

## Related tools

- **SCISSOR** (provides logtransform_data (log shift selection/transformation), scale.factor (skewness statistic), and build_pileup (input pileup construction) for variance stabilization of base-level coverage) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); for (s in param.grid) logtransform_data(pileup, logshiftVal=s) |> scale.factor() -> pick shift with min skewness
```

## Evaluation signals

- logshift_grid_results.csv exists and contains one row per candidate shift value with a skewness field
- the reported selected log shift corresponds to the entry with minimum skewness in the shift-versus-skewness table
- mean skewness after transformation at the selected shift is lower than at grid extremes (small and large pseudo-counts)
- the selected shift value falls within the candidate param.grid range and all rows have non-missing skewness values

## Limitations

- SCISSOR tuning parameters including log normalization and a normality cutoff value likely require re-optimization when a significantly different wet lab RNA-seq protocol is used
- The paper notes tuning parameters 'will likely require some optimization when a significantly different wet lab RNA-seq protocol is used' (discussion), so the automatic shift choice may not transfer across protocols

## Evidence

- [results] Before the main shape change detection procedure, normalization and variance stabilization are performed: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [methods] logarithmic transformation with automatic log shift: "* logarithmic transformation by automatically choosing the log shift parameter"
- [discussion] tuning parameter optimization caveat: "SCISSOR has tuning parameters including log normalization and a normality cutoff value which will likely require some optimization"
- [methods] toy dataset availability: "A toy dataset is available in the `SCISSOR` package:

r
library(SCISSOR)
data(TOY_data)"
