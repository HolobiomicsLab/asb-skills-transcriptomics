---
name: spike-in-benchmark-evaluation
description: Use when a synthetic RNA-seq dataset with known ground truth (e.g. External
  RNA Controls Consortium spike-in transcripts) is available and you need to quantify
  which DEG method or combination maximizes F-measure and logFC accuracy.
when_to_use_negative:
- The input is a real biological dataset with no ground truth (e.g. the Lafora or
  PMM2-CDG disease experiments) — no spike-in labels exist, so F-measure cannot be
  computed
- You only need a single DEG method's calls and do not intend to compare methods or
  combination strategies against a benchmark
- The expression data is not RNA-seq count data from DEgenes Hunter (e.g. already-normalized
  feature tables from proteomics or microarrays)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose DEgenes Hunter pipeline runs the individual DEG
    methods and combination strategies on the spike-in count table
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DESeq2
  role: single DEG method evaluated; achieved top F-measure of 0.88 on the spike-in
    dataset
- name: limma
  role: single DEG method evaluated; F-measure 0.76 and highest logFC correlation
    (mean 0.92) on the spike-in dataset
- name: NOISeq
  role: single DEG method evaluated; F-measure 0.74 on the spike-in dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:9b607b0c6e733aabfa413b79e674c2689dc0e46c34ce6a2d557f1212d3910918
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - ExpHunter Suite
  - NOISeq
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spike-in-benchmark-evaluation

## Summary

Benchmarks individual DEG detection methods (DESeq2, limma, NOISeq) and their combination strategies (vote-threshold, combined-FDR, Naïve Bayes classifier) against known spike-in transcript ground truth, computing F-measure and logFC correlation per method as in the ExpHunter Suite evaluation.

## When to use

Use when a synthetic RNA-seq dataset with known ground truth (e.g. External RNA Controls Consortium spike-in transcripts) is available and you need to quantify which DEG method or combination maximizes F-measure and logFC accuracy.

## When NOT to use

- The input is a real biological dataset with no ground truth (e.g. the Lafora or PMM2-CDG disease experiments) — no spike-in labels exist, so F-measure cannot be computed
- You only need a single DEG method's calls and do not intend to compare methods or combination strategies against a benchmark
- The expression data is not RNA-seq count data from DEgenes Hunter (e.g. already-normalized feature tables from proteomics or microarrays)

## Inputs

- spike-in RNA-seq count table (fastq files converted to counts via clean&map)
- known spike-in transcript ground-truth labels
- ExpHunterSuiteExamples evaluation scripts and supplementary files

## Outputs

- Table 1: per-method and per-combination F-measure values (CSV)
- Table 2: per-method logFC correlation values (CSV)
- DEgenes Hunter HTML results report

## How to apply

Download the spike-in RNA-seq example data and evaluation scripts from the ExpHunterSuiteExamples repository, then run the DEgenes Hunter pipeline (ExpHunter Suite, R/Bioconductor) to execute the individual DEG methods (DESeq2, limma, NOISeq) on the spike-in count table. Apply the combination strategies to the per-method results — the vote-threshold method (the reported optimum threshold was 2 for spike-in data by F-measure) and the combined-FDR strategy — plus the Naïve Bayes classifier trained on simulated data. Compute the F-measure for each single method and combination against the spike-in ground truth and reproduce the published Table 1 values (DESeq2 0.88, combined-FDR 0.88, threshold-2 0.88, limma 0.76, NOISeq 0.74). Separately compute per-method logFC correlations to reproduce Table 2 (e.g. limma mean 0.92), write each table to CSV, and check every metric falls within tolerance of the reference values. The rationale is that spike-in transcripts are the only genes with a known differential-expression status, so precision/recall against them is a direct measure of method performance.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose DEgenes Hunter pipeline runs the individual DEG methods and combination strategies on the spike-in count table) — https://bioconductor.org/packages/ExpHunterSuite
- **DESeq2** (single DEG method evaluated; achieved top F-measure of 0.88 on the spike-in dataset)
- **limma** (single DEG method evaluated; F-measure 0.76 and highest logFC correlation (mean 0.92) on the spike-in dataset)
- **NOISeq** (single DEG method evaluated; F-measure 0.74 on the spike-in dataset)

## Examples

```
bash launch_DEgenesHunter.sh  # run DEgenes Hunter on the spike-in count table, then execute get_results.sh to generate the HTML results report
```

## Evaluation signals

- Reproduced Table 1 F-measures fall within tolerance: DESeq2 0.88, combined-FDR 0.88, threshold-2 0.88, limma 0.76, NOISeq 0.74
- Reproduced Table 2 logFC correlations match reference values (limma mean 0.92)
- Vote-threshold sweep confirms threshold 2 is optimal for the spike-in data by F-measure
- Per-method F-measure and logFC correlation tables are written to CSV with all metrics populated
- DEgenes Hunter execution generates an HTML results report via get_results.sh

## Limitations

- The optimum vote threshold of 2 may not generalize — it may differ for other organisms, tissues and experimental designs, and the authors are hesitant to extrapolate a general rule
- Competing recommendations of much higher thresholds were based on the analysis of a single human dataset
- More elaborate classifiers than Naïve Bayes were left for future work

## Evidence

- [discussion] the Naïve Bayes classifier presented here performed as well as the best performing single and combination methods on the spike-in dataset: [evidence span withheld — anchor status: not_found]
- [discussion] optimum threshold 2: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [abstract] combination leads to optimal results: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [readme] README spike-in workflow: [evidence span withheld — anchor status: external_doc]
