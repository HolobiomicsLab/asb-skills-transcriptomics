---
name: r-package-function-implementation
description: Use when reconstructing or extending published R package functionality
  — e.g. re-implementing SCISSOR's logtransform_data/scale.factor routines on base-level
  pileup matrices — and you need concrete function signatures, grid-search logic,
  and validation checks.
when_to_use_negative:
- Input is already normalized/transformed count data (e.g. normalized RSEM values
  or a feature table) rather than raw base-level coverage pileups — the log shift
  grid search assumes untransformed pileup magnitudes
- The goal is differential isoform or junction usage with known transcript models
  (DEXSeq, SpliceTrap, MISO, rMATS territory) rather than base-resolution shape change
  detection
- Cohorts are very small — the article reports ~30 samples as the smallest successfully
  considered cohort, determined only empirically
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0085
tools:
- name: SCISSOR
  role: R package providing the logtransform_data, scale.factor, build_pileup, get_Ranges,
    and Scissor functions being implemented/invoked for variance stabilization and
    shape change detection
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:83a7d2690869b4ea094332f961351a0fbc9bae610546162827e012b6c5be77b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# r-package-function-implementation

## Summary

Implement and validate R package functions that reproduce a published method's internal routines — here, SCISSOR's variance-stabilization step that log-transforms base-level RNA-seq coverage pileups after automatically choosing the log shift (pseudo-count) parameter by minimizing residual skewness.

## When to use

Use when reconstructing or extending published R package functionality — e.g. re-implementing SCISSOR's logtransform_data/scale.factor routines on base-level pileup matrices — and you need concrete function signatures, grid-search logic, and validation checks.

## When NOT to use

- Input is already normalized/transformed count data (e.g. normalized RSEM values or a feature table) rather than raw base-level coverage pileups — the log shift grid search assumes untransformed pileup magnitudes
- The goal is differential isoform or junction usage with known transcript models (DEXSeq, SpliceTrap, MISO, rMATS territory) rather than base-resolution shape change detection
- Cohorts are very small — the article reports ~30 samples as the smallest successfully considered cohort, determined only empirically

## Inputs

- Base-level RNA-seq coverage pileup matrix (e.g. built via SCISSOR build_pileup, or the TOY_data pileup from the SCISSOR package)
- Vector/grid of candidate log shift (pseudo-count) values applied before log transformation
- Genomic ranges object (from SCISSOR get_Ranges) used as input to Scissor functions

## Outputs

- Log-transformed, variance-stabilized pileup matrix at the selected logshiftVal
- Table mapping each candidate log shift value to its overall skewness statistic (e.g. logshift_grid_results.csv)
- Selected minimizing log shift parameter and the mean overall skewness at that shift

## How to apply

Load the base-level coverage pileup matrix from the SCISSOR TOY_data example dataset, then construct a param.grid of candidate log shift (pseudo-count) values spanning small-to-large additions applied before log transformation. For each candidate, call SCISSOR's logtransform_data with logshiftVal set to the candidate, compute per-sample scale factors with the scale.factor routine, and aggregate them into a mean overall skewness statistic that quantifies residual asymmetry of the transformed distributions. Select the shift attaining the minimum mean skewness across the grid — this automates the log shift choice so the SCISSOR test statistic becomes more symmetrical, addressing skewness that RNA-seq coverage data retains. Write the candidate-versus-skewness table (e.g. logshift_grid_results.csv) and validate that one row exists per candidate, a skewness field is present, and the reported selected shift equals the minimum-skewness entry. Install the package with devtools::install_github("hyochoi/SCISSOR"); note dependencies Rsamtools, GenomicRanges, refGenome, nloptr, and zoo.

## Related tools

- **SCISSOR** (R package providing the logtransform_data, scale.factor, build_pileup, get_Ranges, and Scissor functions being implemented/invoked for variance stabilization and shape change detection) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); transformed <- logtransform_data(pileup, logshiftVal=selected_shift)
```

## Evaluation signals

- logshift_grid_results.csv exists and contains exactly one row per candidate log shift value with a skewness field
- The reported selected log shift equals the candidate with the minimum skewness in the grid table (value_in_range check)
- Mean overall skewness of the transformed matrix at the selected shift is lower than at extreme (smallest and largest) grid candidates
- Function chain reproduces documented behavior: build_pileup -> get_Ranges -> logtransform_data -> scale.factor inputs match the SCISSOR methods description
- Package installs cleanly with install_github("hyochoi/SCISSOR") and TOY_data loads via data(TOY_data)

## Limitations

- SCISSOR tuning parameters including log normalization and a normality cutoff value likely require optimization when a significantly different wet-lab RNA-seq protocol is used
- The GSC detection algorithm loses information about the order of base pairs, and LSC results may change with different window sizes
- No precise effect-size experiments exist for the log shift selection; the ~30-sample minimum cohort size is only empirically established

## Evidence

- [results] Normalization and variance stabilization performed as pre-processing before shape change detection: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [methods] log shift parameter chosen automatically: "logarithmic transformation by automatically choosing the log shift parameter"
- [methods] TOY_data available in the SCISSOR package: "A toy dataset is available in the `SCISSOR` package"
- [readme] installation instructions from README: "install_github("hyochoi/SCISSOR")"
- [methods] pileup matrix and genomic ranges as function inputs: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
