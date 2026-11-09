---
name: skewness-minimization-via-scale-factors
description: Use when base-level RNA-seq coverage pileup data (e.g., SCISSOR TOY_data
  pileup matrix) is right-skewed and a log shift parameter must be chosen automatically
  to variance-stabilize data prior to SCISSOR global/local shape change detection.
when_to_use_negative:
- Input is already log-transformed or otherwise variance-stabilized coverage data,
  where an additional shift-and-log step would distort coverage shapes
- Analysis target is expression quantification rather than base-level coverage shape
  — SCISSOR operates on base-level pileups, not RSEM-style gene/transcript counts
- Sample-level quality filtering is the actual need (e.g., removing degraded RNA samples
  or non-expressed genes), which is a separate SCISSOR pre-processing step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SCISSOR
  role: Provides logtransform_data (log-transformation with logshiftVal parameter)
    and scale.factor routines used to transform pileups and quantify residual skewness
    for shift selection
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:dc6fe5930595e94a66e5ef884972dce66c5f9832fa3d470d61b565ef8dab4e91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# skewness-minimization-via-scale-factors

## Summary

Selects the log shift (pseudo-count) parameter for SCISSOR's log-transformation of base-level RNA-seq coverage pileups by scanning a grid of candidate shift values and picking the one minimizing overall skewness, so the SCISSOR test statistic becomes more symmetrical before shape change detection.

## When to use

Use when base-level RNA-seq coverage pileup data (e.g., SCISSOR TOY_data pileup matrix) is right-skewed and a log shift parameter must be chosen automatically to variance-stabilize data prior to SCISSOR global/local shape change detection.

## When NOT to use

- Input is already log-transformed or otherwise variance-stabilized coverage data, where an additional shift-and-log step would distort coverage shapes
- Analysis target is expression quantification rather than base-level coverage shape — SCISSOR operates on base-level pileups, not RSEM-style gene/transcript counts
- Sample-level quality filtering is the actual need (e.g., removing degraded RNA samples or non-expressed genes), which is a separate SCISSOR pre-processing step

## Inputs

- Base-level RNA-seq coverage pileup matrix (e.g., built by SCISSOR build_pileup from TOY_data or BAM-derived pileups)
- Grid of candidate log shift (pseudo-count) values to apply before log-transformation

## Outputs

- Log-transformed pileup matrix at the selected log shift value
- Table of candidate shift values versus mean overall skewness (e.g., logshift_grid_results.csv) with the selected minimizing shift

## How to apply

Load the base-level coverage pileup matrix (e.g., from the SCISSOR TOY_data example dataset) as produced by build_pileup. Construct a param.grid of candidate log shift (pseudo-count) values spanning small-to-large additions applied before log-transformation. For each candidate, call SCISSOR's logtransform_data with logshiftVal set to that candidate, then compute sample scale factors via the scale.factor routine and aggregate them into a mean overall skewness statistic quantifying residual asymmetry. Select the shift attaining minimum mean skewness across the grid as the automatically chosen log shift parameter, and record the shift-versus-skewness table. This variance stabilization is deliberately performed as pre-processing, before the main shape change detection procedure, because RNA-seq coverage data can retain skewness that distorts the SCISSOR test statistic.

## Related tools

- **SCISSOR** (Provides logtransform_data (log-transformation with logshiftVal parameter) and scale.factor routines used to transform pileups and quantify residual skewness for shift selection) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); pileup <- build_pileup(TOY_data); lt <- logtransform_data(pileup, logshiftVal = 200); sf <- scale.factor(lt)
```

## Evaluation signals

- logshift_grid_results.csv exists and contains one row per candidate shift value with a skewness field
- The reported selected shift equals the entry with the minimum skewness in the candidate-vs-skewness table (value_in_range check)
- After transformation with the selected shift, aggregated sample scale factors indicate reduced skewness relative to smaller/larger candidates in the grid
- Normalization and variance stabilization occur before any global shape change (GSC) or local shape change (LSC) detection is run
- Script completes (script_runs) with the transformed pileup matrix retaining base-level dimensions matching the input pileup and genomic ranges

## Limitations

- Log normalization (and the normality cutoff value) are tuning parameters that will likely require optimization when a significantly different wet lab RNA-seq protocol is used
- Choice of log shift addresses skewness of the test statistic but does not remove other artifactual signals such as those from variable fragment length or GC content effects
- Variance stabilization is a pre-processing step only; it does not substitute for SCISSOR's normalization or its subsequent GSC/LSC detection steps

## Evidence

- [results] Before the main shape change detection procedure, normalization and variance stabilization are performed: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [methods] logarithmic transformation with automatic log shift selection: "logarithmic transformation by automatically choosing the log shift parameter"
- [discussion] log normalization is a tuning parameter needing optimization: "SCISSOR has tuning parameters including log normalization and a normality cutoff value which will likely require some optimization"
- [abstract] SCISSOR is a series of transformation and normalization procedures: "a series of procedures for transforming and normalizing base-level RNA sequencing coverage data"
- [methods] toy dataset provided in the package for testing the pipeline: "A toy dataset is available in the `SCISSOR` package"
