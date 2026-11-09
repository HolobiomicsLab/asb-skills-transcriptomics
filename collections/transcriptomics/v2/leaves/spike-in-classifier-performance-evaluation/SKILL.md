---
name: spike-in-classifier-performance-evaluation
description: Use when a trained multi-method DEG integration model (e.g. Naïve Bayes
  over per-gene p-value vectors) must be benchmarked on independent data with ground-truth
  DEG/non-DEG labels, such as spike-in experiments.
when_to_use_negative:
- The dataset has no ground-truth DEG/non-DEG labels (e.g. the Lafora disease or PMM2-CDG
  real datasets) — performance cannot be scored without known truth.
- The classifier was not trained on simulations matched to your dataset's characteristics
  — the paper trained the Naïve Bayes model specifically on simulations based on the
  Lafora disease dataset, so its votes may not transfer to other organisms, tissues,
  or experimental designs.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: Provides the DEgenes Hunter pipeline that runs the multiple DEG detection
    methods and the Naïve Bayes classifier used to combine their results on the spike-in
    data
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: R
  role: Runtime environment in which the classifier is applied and the F-measure is
    computed
- name: Bioconductor
  role: Package ecosystem through which ExpHunter Suite is distributed and executed
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:2818f67f344e841966a8f599740d49ebb94a0b35d1dad6650d9c41ffcc090e8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ExpHunter Suite
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spike-in-classifier-performance-evaluation

## Summary

Evaluates a machine-learning DEG-combination classifier (the ExpHunter Suite Naïve Bayes model trained on simulated p-value vectors) by applying it to a spike-in RNA-seq dataset with known truth labels and computing its F-measure against single methods and vote-threshold combinations.

## When to use

Use when a trained multi-method DEG integration model (e.g. Naïve Bayes over per-gene p-value vectors) must be benchmarked on independent data with ground-truth DEG/non-DEG labels, such as spike-in experiments.

## When NOT to use

- The dataset has no ground-truth DEG/non-DEG labels (e.g. the Lafora disease or PMM2-CDG real datasets) — performance cannot be scored without known truth.
- The classifier was not trained on simulations matched to your dataset's characteristics — the paper trained the Naïve Bayes model specifically on simulations based on the Lafora disease dataset, so its votes may not transfer to other organisms, tissues, or experimental designs.

## Inputs

- Fitted Naïve Bayes classifier serialized as naive_bayes_model.rds (trained on simulated per-method p-value vectors)
- Spike-in RNA-seq count tables processed through DEgenes Hunter
- Per-gene p-value vectors from multiple DEG detection methods (e.g. DESeq2, NOISeq, limma) on the spike-in dataset
- Ground-truth DEG/non-DEG labels for the spike-in dataset

## Outputs

- Per-gene predicted DEG/non-DEG class labels for the spike-in dataset
- F-measure of the Naïve Bayes classifier on the spike-in data (expected ~0.88)
- Comparison of the classifier's F-measure against single-method and vote-threshold baselines (optimum vote threshold = 2)

## How to apply

Apply the serialized Naïve Bayes model (naive_bayes_model.rds), previously trained in R/Bioconductor on simulated datasets generated from the characteristics of the Lafora disease dataset, to the spike-in count tables processed with DEgenes Hunter. Feed each gene's per-method p-value vector as classifier features, predict DEG/non-DEG classes, and compute the F-measure against the known spike-in truth labels. An F-measure around 0.88 is expected, which must be comparable to the best-performing single method (DESeq2) and to the vote-threshold combination, whose optimum cut-off on the spike-in data was 2. Compare the classifier's score against the Naive Bayes row of Table 1 of the article to confirm it performs as well as the best single and combination methods. The rationale is that simulated and spike-in experiments are the two benchmarks the paper uses to show that combining methods via consensus and classifiers leads to optimal results.

## Related tools

- **ExpHunter Suite** (Provides the DEgenes Hunter pipeline that runs the multiple DEG detection methods and the Naïve Bayes classifier used to combine their results on the spike-in data) — https://bioconductor.org/packages/ExpHunterSuite
- **R** (Runtime environment in which the classifier is applied and the F-measure is computed)
- **Bioconductor** (Package ecosystem through which ExpHunter Suite is distributed and executed) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- F-measure of the Naïve Bayes classifier on the spike-in dataset is ~0.88 and comparable to the best-performing single and combination methods (Naive Bayes row of Table 1).
- The classifier's F-measure is not worse than the vote-threshold combination at its spike-in optimum cut-off of 2, nor than DESeq2 alone.
- Predictions are produced for every gene in the spike-in count table with a complete per-method p-value vector as features.
- The model applied is the serialized naive_bayes_model.rds trained on simulations derived from the Lafora disease dataset characteristics, not a freshly refit model.
- The evaluation workflow runs end-to-end via the ExpHunterSuiteExamples scripts (launch.sh / get_results.sh) and generates the HTML results report.

## Limitations

- The optimum vote threshold of 2 may differ for other organisms, tissues, and experimental designs; the authors are hesitant to extrapolate these findings to a more general rule.
- A competing recommendation of much higher thresholds was based on the analysis of a single human dataset.
- More elaborate classifiers than Naïve Bayes are left for future work; the current evaluation is specific to the Naïve Bayes approach.

## Evidence

- [abstract] result integration via data-fusion, threshold based methods, and a Naïve Bayes classifier trained on simulated data: "result integration via data-fusion, threshold based methods, and a Naïve Bayes classifier trained on simulated data"
- [discussion] the Naïve Bayes classifier presented here performed as well as the best performing single and combination methods on the spike-in dataset: [evidence span withheld — anchor status: not_found]
- [discussion] for the spike-in data the optimum threshold was 2 in terms of F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [readme] Install and configure your R environment as is explained in https://github.com/seoanezonjic/ExpHunterSuite: [evidence span withheld — anchor status: external_doc]
