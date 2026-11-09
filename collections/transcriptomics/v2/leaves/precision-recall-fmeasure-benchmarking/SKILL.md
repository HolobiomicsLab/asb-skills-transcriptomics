---
name: precision-recall-fmeasure-benchmarking
description: Use when you have simulated RNA-seq count data with known DE/non-DE gene
  labels and need to compare DEG-calling consensus strategies (vote thresholds of
  1–4 or combined-FDR) by precision, recall and F-measure.
when_to_use_negative:
- Evaluating DEG methods on real experimental data (e.g. Lafora or PMM2-CDG datasets)
  where no simulated truth labels exist to compute precision/recall/F-measure against
- Choosing a method for a single well-characterized dataset rather than benchmarking
  combination strategies — the authors note the optimum threshold 'may differ for
  other organisms, tissues and experimental designs'
- Input is raw FASTQ or unprocessed reads; this skill operates downstream on count
  tables with truth labels
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEgenes Hunter pipeline that runs the
    individual DEG detection methods whose combined calls are benchmarked
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline that produces individual DEG calls from DESeq2, NOISeq and limma
    on the simulated datasets
- name: DESeq2
  role: One of the integrated DEG detection methods whose votes feed the consensus
    thresholds
- name: NOISeq
  role: One of the integrated DEG detection methods whose votes feed the consensus
    thresholds
- name: limma
  role: One of the integrated DEG detection methods whose votes feed the consensus
    thresholds
- name: R/Bioconductor
  role: Execution environment for running the pipeline and computing precision/recall/F-measure
    metrics
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:07604679941790fd4cd62c8f9c7a6cf9660b81349b530fc148a1b36496338f24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - R/Bioconductor
  - limma
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# precision-recall-fmeasure-benchmarking

## Summary

Benchmarks DEG detection method-combination strategies (minimum-vote thresholds vs combined-FDR data fusion) by computing precision, recall and F-measure against known truth labels on simulated RNA-seq datasets, to identify the optimal integration cut-off (vote threshold 2).

## When to use

Use when you have simulated RNA-seq count data with known DE/non-DE gene labels and need to compare DEG-calling consensus strategies (vote thresholds of 1–4 or combined-FDR) by precision, recall and F-measure.

## When NOT to use

- Evaluating DEG methods on real experimental data (e.g. Lafora or PMM2-CDG datasets) where no simulated truth labels exist to compute precision/recall/F-measure against
- Choosing a method for a single well-characterized dataset rather than benchmarking combination strategies — the authors note the optimum threshold 'may differ for other organisms, tissues and experimental designs'
- Input is raw FASTQ or unprocessed reads; this skill operates downstream on count tables with truth labels

## Inputs

- Simulated RNA-seq count datasets with known DE and non-DE gene labels (from upstream simulation step)
- Per-method DEG call sets / statistics from DESeq2, NOISeq and limma via the DEgenes Hunter pipeline

## Outputs

- deg_metrics_comparison.csv with precision/recall/F-measure values in [0,1] for each vote threshold and the combined-FDR criterion
- Method-by-metric comparison table cross-checkable against Fig. 3 style reference summaries

## How to apply

Load the simulated RNA-seq count datasets produced by the upstream simulation step, which carry known differentially-expressed and non-DE gene labels. Run the DEgenes Hunter pipeline within ExpHunter Suite in R to obtain individual DEG calls from each integrated method (DESeq2, NOISeq, limma). Apply minimum-vote threshold cut-offs of 1, 2, 3 and 4 — a gene is called DE if at least that many methods vote for it — and also apply the combined-FDR data-fusion criterion that integrates per-method statistics into a single call set. Compute precision, recall and F-measure for each threshold and the combined-FDR criterion against the simulated truth labels, writing values in [0,1] to deg_metrics_comparison.csv. Expect the combined-FDR criterion to behave similarly to a minimum-vote threshold of 2, the optimum by F-measure on the spike-in/simulated data; raising the threshold trades recall for precision.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEgenes Hunter pipeline that runs the individual DEG detection methods whose combined calls are benchmarked) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (Pipeline that produces individual DEG calls from DESeq2, NOISeq and limma on the simulated datasets)
- **DESeq2** (One of the integrated DEG detection methods whose votes feed the consensus thresholds)
- **NOISeq** (One of the integrated DEG detection methods whose votes feed the consensus thresholds)
- **limma** (One of the integrated DEG detection methods whose votes feed the consensus thresholds)
- **R/Bioconductor** (Execution environment for running the pipeline and computing precision/recall/F-measure metrics) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e "devtools::install_github('seoanezonjic/ExpHunterSuite', dependencies=TRUE)"; export PATH=path_to_install_folder:$PATH; DEgenes_Hunter.sh -i simulated_counts.tsv -o deg_metrics_comparison.csv
```

## Evaluation signals

- Precision, recall and F-measure values all fall in [0,1] and are populated for each of thresholds 1–4 and the combined-FDR criterion
- Raising the minimum-vote threshold monotonically increases precision while recall decreases
- Combined-FDR criterion metrics are similar to (comparable with) the vote threshold of 2, which is the optimum by F-measure
- Results written to deg_metrics_comparison.csv match the Fig. 3 style reference summaries
- The synthetic experiment reproduces end-to-end via launch.sh/get_results.sh from ExpHunterSuiteExamples, generating the expected HTML results report

## Limitations

- The optimum vote threshold (2 by F-measure on spike-in data) may not generalize: 'this may differ for other organisms, tissues and experimental designs' and the authors are 'hesitant to extrapolate these findings to a more general rule'
- A competing recommendation of much higher thresholds was based on the analysis of a single human dataset, limiting comparability
- Findings derive from simulated and spike-in experiments; behaviour on real rare-disease datasets (Lafora, PMM2-CDG) is assessed qualitatively rather than against truth labels

## Evidence

- [results] Raising the minimum-vote threshold increases precision at the cost of recall, and the combined-FDR criterion performs comparatively well: [evidence span withheld — anchor status: not_found]
- [discussion] Optimum vote threshold of 2: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [discussion] Recommendation of minimum vote threshold: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [abstract] Consensus combination yields optimal results: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [readme] README instructions for replicating synthetic experiments: [evidence span withheld — anchor status: external_doc]
- [discussion] Generalization caveat for vote threshold: "this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule"
