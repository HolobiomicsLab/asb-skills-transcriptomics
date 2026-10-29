---
name: log-shift-parameter-grid-search
description: Use when base-level RNA-seq coverage pileup data retain strong right
  skew and a log transform is needed before SCISSOR shape change detection, but the
  pseudo-count added prior to log-transformation is unknown and must be chosen automatically.
when_to_use_negative:
- The input is not base-level coverage pileup data (e.g. it is already a gene-level
  count or RSEM expression matrix, where the SCISSOR base-level normalization and
  log-shift machinery does not apply)
- The transformed coverage is already sufficiently symmetrical, so an additional pseudo-count
  grid search adds no variance stabilization benefit
- Cohort size is below the empirically supported minimum (~30 samples) at which SCISSOR
  shape change detection has been successfully considered
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSOR
  role: Provides logtransform_data (logarithmic transformation with a tunable logshiftVal),
    scale.factor (sample scale factors aggregated into the mean skewness statistic),
    build_pileup/get_Ranges, and the TOY_data example dataset used to run the grid
    search.
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:81cd95eb0434bb027923211e74a402097898b52bd6da22c8fa2e6562289f73ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# log-shift-parameter-grid-search

## Summary

Grid-searches a pseudo-count (log shift) parameter for the logarithmic transformation of base-level RNA-seq coverage pileups, selecting the value that minimizes the mean sample skewness of the transformed matrix so that SCISSOR's shape change test statistic becomes more symmetrical.

## When to use

Use when base-level RNA-seq coverage pileup data retain strong right skew and a log transform is needed before SCISSOR shape change detection, but the pseudo-count added prior to log-transformation is unknown and must be chosen automatically.

## When NOT to use

- The input is not base-level coverage pileup data (e.g. it is already a gene-level count or RSEM expression matrix, where the SCISSOR base-level normalization and log-shift machinery does not apply)
- The transformed coverage is already sufficiently symmetrical, so an additional pseudo-count grid search adds no variance stabilization benefit
- Cohort size is below the empirically supported minimum (~30 samples) at which SCISSOR shape change detection has been successfully considered

## Inputs

- Base-level coverage pileup matrix (e.g. SCISSOR build_pileup output from a BAM-derived pileup or the TOY_data example dataset)
- param.grid of candidate log shift (pseudo-count) values

## Outputs

- Selected log shift parameter minimizing mean skewness
- Table of candidate shift values versus overall skewness (logshift_grid_results.csv), one row per candidate with a skewness field

## How to apply

Load the base-level coverage pileup matrix (e.g. built with SCISSOR's build_pileup from the TOY_data example dataset). Define a param.grid of candidate log shift values spanning small-to-large pseudo-counts. For each candidate, apply SCISSOR's logtransform_data function with logshiftVal set to that candidate, then compute sample scale factors with the scale.factor routine and aggregate them into a mean overall skewness statistic quantifying residual asymmetry. Select the shift attaining the minimum mean skewness and record the full shift-versus-skewness grid (logshift_grid_results.csv). The rationale is that normalization and variance stabilization before the main shape change detection procedure yield a more symmetrical distribution of the SCISSOR test statistic, which RNA-seq coverage data would otherwise violate.

## Related tools

- **SCISSOR** (Provides logtransform_data (logarithmic transformation with a tunable logshiftVal), scale.factor (sample scale factors aggregated into the mean skewness statistic), build_pileup/get_Ranges, and the TOY_data example dataset used to run the grid search.) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); transformed <- logtransform_data(data = pileup_matrix, logshiftVal = 2)
```

## Evaluation signals

- logshift_grid_results.csv exists and contains exactly one row per candidate shift value in param.grid, each with a skewness field
- The reported selected shift equals the entry with the minimum skewness value in the grid table
- The selected shift falls within the plausible range spanned by the candidate grid (value_in_range check)
- The pipeline executes end-to-end on the TOY_data pileup matrix without errors (script_runs check)
- Skewness varies monotonically or shows a clear minimum across the grid, consistent with variance stabilization behavior

## Limitations

- SCISSOR's tuning parameters, including the log normalization shift, likely require re-optimization when a significantly different wet-lab RNA-seq protocol is used, so the grid-searched value may not transfer across protocols
- The chosen shift only symmetrizes the test statistic; it does not address other coverage artifacts such as those from variable fragment length or GC content effects

## Evidence

- [methods] logshift step: "logarithmic transformation by automatically choosing the log shift parameter"
- [results] pre-processing rationale: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [methods] toy dataset for grid search: "A toy dataset is available in the `SCISSOR` package"
- [readme] README scope: "aims for unsupervised screening of a range of structural alterations in RNA-seq data"
