---
name: simulated-rnaseq-truth-label-comparison
description: Use when you need to evaluate or tune how multiple DEG detection methods
  (DESeq2, NOISeq, limma) are combined, by comparing vote-threshold or combined-FDR
  calls against simulated RNA-seq data with known truth labels.
when_to_use_negative:
- When analysing real RNA-seq data without ground-truth DE/non-DE labels — precision/recall
  against truth cannot be computed; use the vote-threshold recommendation directly
  instead
- When the goal is co-expression module detection or functional enrichment rather
  than benchmarking DEG method-combination strategies
- When extrapolating a single optimum threshold as a general rule — the authors note
  the optimum may differ for other organisms, tissues and experimental designs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEgenes Hunter pipeline used to generate
    individual DEG calls from each integrated method on simulated data
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline within ExpHunter Suite that runs DESeq2, NOISeq and limma and supports
    minimum-vote and combined-FDR integration of DEG calls
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: ExpHunter Suite Examples
  role: Repository with code to generate the synthetic results published in the paper,
    including launch.sh and get_results.sh for replication
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
- name: R/Bioconductor
  role: Execution environment in which the ExpHunter Suite pipeline and simulation/benchmarking
    scripts are run
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:a513b0bac311a4af559b5d1da1f8e4b21678515367379a827d3338128f3e0231
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DEgenes Hunter
  - ExpHunter Suite
  - ExpHunter Suite Examples
  - R
  - R/Bioconductor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulated-rnaseq-truth-label-comparison

## Summary

Benchmark DEG detection method-combination strategies (minimum-vote thresholds of 1-4 and a combined-FDR data-fusion criterion) against simulated RNA-seq datasets with known DE/non-DE truth labels, scoring each with precision, recall, and F-measure.

## When to use

Use when you need to evaluate or tune how multiple DEG detection methods (DESeq2, NOISeq, limma) are combined, by comparing vote-threshold or combined-FDR calls against simulated RNA-seq data with known truth labels.

## When NOT to use

- When analysing real RNA-seq data without ground-truth DE/non-DE labels — precision/recall against truth cannot be computed; use the vote-threshold recommendation directly instead
- When the goal is co-expression module detection or functional enrichment rather than benchmarking DEG method-combination strategies
- When extrapolating a single optimum threshold as a general rule — the authors note the optimum may differ for other organisms, tissues and experimental designs

## Inputs

- Simulated RNA-seq count datasets with known differentially-expressed and non-DE gene truth labels
- Per-method DEG calls (DESeq2, NOISeq, limma) from the DEgenes Hunter pipeline

## Outputs

- deg_metrics_comparison.csv with precision/recall/F-measure values in [0,1] per method-combination strategy
- Method-by-metric comparison tables suitable for Fig. 3 style reference summaries

## How to apply

Load simulated RNA-seq count datasets with known differentially-expressed and non-DE gene labels from the upstream simulation step. Run the DEgenes Hunter pipeline within ExpHunter Suite in R to obtain individual DEG calls from each integrated method (DESeq2, NOISeq, limma). Apply minimum-vote threshold cut-offs of 1, 2, 3 and 4, where a gene is called DE if at least that many methods voted for it, and also apply the combined-FDR data-fusion criterion that integrates per-method statistics. Compute precision, recall and F-measure for each strategy against the simulated truth labels. The expected behaviour is a precision-recall trade-off: raising the vote threshold increases precision at the cost of recall, and the combined-FDR criterion behaves similarly to a vote threshold of 2, which is optimal by F-measure.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEgenes Hunter pipeline used to generate individual DEG calls from each integrated method on simulated data) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (Pipeline within ExpHunter Suite that runs DESeq2, NOISeq and limma and supports minimum-vote and combined-FDR integration of DEG calls) — https://github.com/seoanezonjic/ExpHunterSuite
- **ExpHunter Suite Examples** (Repository with code to generate the synthetic results published in the paper, including launch.sh and get_results.sh for replication) — https://github.com/fmjabato/ExpHunterSuiteExamples
- **R/Bioconductor** (Execution environment in which the ExpHunter Suite pipeline and simulation/benchmarking scripts are run) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)' && ./launch.sh
```

## Evaluation signals

- deg_metrics_comparison.csv contains precision/recall/F-measure values, each within [0,1], for vote thresholds 1-4 and the combined-FDR criterion
- The combined-FDR criterion's metrics are close to (comparable with) those of minimum-vote threshold 2
- Raising the vote threshold monotonically shifts the trade-off: precision increases while recall decreases
- Vote threshold 2 yields the highest F-measure, matching the reported optimum for the spike-in/simulated data
- All DEG calls originate from the same simulated datasets whose truth labels were produced by the upstream simulation step (task_001)

## Limitations

- The optimal vote threshold may not generalize: 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- A competing study recommending much higher thresholds was based on a single human dataset, limiting cross-study comparison
- Metrics depend on the specific simulation parameters from the upstream simulation step; different simulation regimes may shift the optimum threshold

## Evidence

- [results] Raising the minimum-vote threshold increases precision at the cost of recall, and the combined-FDR criterion performs comparatively well across all measures, behaving similarly to a minimum-vote threshold of 2.: [evidence span withheld — anchor status: not_found]
- [discussion] for the spike-in data the optimum threshold was 2 in terms of F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: [evidence span withheld — anchor status: not_found]
- [intro] the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data: "the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data"
- [discussion] this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule: "this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule"
