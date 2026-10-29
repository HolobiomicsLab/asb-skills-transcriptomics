---
name: vote-threshold-classifier-tuning
description: Use when combining results of multiple DEG detection methods (DESeq2,
  limma, NOISeq) in ExpHunter Suite and needing to select an optimal vote-threshold
  cut-off for robust DEG calling, benchmarked against known ground truth.
when_to_use_negative:
- Input data has no known ground truth (e.g. real disease datasets like Lafora or
  PMM2-CDG), so F-measure against spike-ins cannot be computed; the article notes
  the authors are hesitant to extrapolate the optimal threshold as a general rule
  since 'this may differ for other organisms, tissues and experimental designs'
- The analysis goal is co-expression module detection or functional enrichment rather
  than DEG detection benchmarking
- Only a single DEG method's output is available and no method combination is planned
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor framework executing the DEgenes Hunter pipeline and the vote-threshold
    combination of individual DEG method results
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: Individual DEG detection method whose calls feed into the vote-threshold combination;
    top single-method performer (F-measure 0.88)
- name: limma
  role: Individual DEG detection method contributing votes; higher precision and best
    logFC correlation (mean 0.92)
- name: NOISeq
  role: Individual DEG detection method contributing votes; F-measure 0.74 on the
    spike-in benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:028cdba07d5604485cadb5935b2e1ce5c8501d136b74b50d3067c9e7f00c2019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - ExpHunter Suite
  - NOISeq
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# vote-threshold-classifier-tuning

## Summary

Tunes the minimum vote threshold cut-off used to combine multiple DEG detection methods by evaluating F-measure against a ground-truth (spike-in) benchmark, and compares it with the Naïve Bayes classifier and combined-FDR strategies in ExpHunter Suite.

## When to use

Use when combining results of multiple DEG detection methods (DESeq2, limma, NOISeq) in ExpHunter Suite and needing to select an optimal vote-threshold cut-off for robust DEG calling, benchmarked against known ground truth.

## When NOT to use

- Input data has no known ground truth (e.g. real disease datasets like Lafora or PMM2-CDG), so F-measure against spike-ins cannot be computed; the article notes the authors are hesitant to extrapolate the optimal threshold as a general rule since 'this may differ for other organisms, tissues and experimental designs'
- The analysis goal is co-expression module detection or functional enrichment rather than DEG detection benchmarking
- Only a single DEG method's output is available and no method combination is planned

## Inputs

- spike-in RNA-seq count table (raw counts)
- known spike-in transcript ground truth (true DEG labels)
- per-method DEG results from DESeq2, limma, NOISeq produced by DEgenes Hunter

## Outputs

- Table 1: F-measure per single method and combination strategy (DESeq2 0.88, combined-FDR 0.88, limma 0.76, NOISeq 0.74, threshold-2 0.88)
- Table 2: logFC correlation per method (limma mean 0.92)
- CSV files of per-method F-measure and logFC correlation values

## How to apply

Run the DEgenes Hunter pipeline (ExpHunter Suite, R/Bioconductor) on the spike-in RNA-seq count table to obtain per-method DEG calls from DESeq2, limma, and NOISeq. For each candidate vote-threshold cut-off, call a gene differentially expressed only if at least that many methods support it, and compute the F-measure of each method and combination against the known spike-in transcripts. For the spike-in data the optimum threshold was 2 in terms of F-measure, yielding 0.88 for the threshold-2 consensus, matching the Naïve Bayes classifier and combined-FDR (both 0.88), versus limma 0.76 and NOISeq 0.74. Also compute per-method logFC correlations (limma mean 0.92) to assess fold-change accuracy. Write the F-measure and logFC correlation tables to CSV and verify values against published references.

## Related tools

- **ExpHunter Suite** (R/Bioconductor framework executing the DEgenes Hunter pipeline and the vote-threshold combination of individual DEG method results) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (Individual DEG detection method whose calls feed into the vote-threshold combination; top single-method performer (F-measure 0.88))
- **limma** (Individual DEG detection method contributing votes; higher precision and best logFC correlation (mean 0.92))
- **NOISeq** (Individual DEG detection method contributing votes; F-measure 0.74 on the spike-in benchmark)

## Examples

```
Rscript -e "devtools::install_github('seoanezonjic/ExpHunterSuite', dependencies=TRUE)"; bash launch_DEgenesHunter.sh
```

## Evaluation signals

- Optimal vote threshold on spike-in data equals 2 by F-measure
- Threshold-2 consensus F-measure reaches 0.88, matching Naïve Bayes classifier and combined-FDR performance
- Single-method F-measures fall near published values (DESeq2 0.88, limma 0.76, NOISeq 0.74)
- limma logFC correlation mean is approximately 0.92
- Per-method F-measure and logFC correlation CSV tables reproduce Table 1 and Table 2 within tolerance

## Limitations

- The optimal vote threshold may not generalize: 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- A competing recommendation of much higher thresholds was based on the analysis of a single human dataset
- More elaborate classifiers than Naïve Bayes are left for future work

## Evidence

- [discussion] optimum threshold 2 for spike-in data by F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [abstract] Combining multiple methods leads to optimal results: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [discussion] Naive Bayes performance on spike-in dataset: [evidence span withheld — anchor status: not_found]
- [readme] README spike-in replication workflow: [evidence span withheld — anchor status: external_doc]
- [discussion] vote threshold recommendation: "we would suggest the use of a minimum vote threshold cut-off where possible"
