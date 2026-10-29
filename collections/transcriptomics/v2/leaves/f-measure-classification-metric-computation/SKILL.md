---
name: f-measure-classification-metric-computation
description: Use when evaluating a DEG-calling method or combined classifier (e.g.
  Naïve Bayes over multi-method p-value vectors) against a spike-in dataset with known
  DEG/non-DEG truth labels, to quantify performance as an F-measure.
when_to_use_negative:
- The dataset has no known DEG/non-DEG ground truth (e.g. the Lafora disease or PMM2-CDG
  model datasets), so an F-measure cannot be computed — use consensus/vote-threshold
  heuristics instead
- The goal is choosing a single DEG detection method for a new dataset rather than
  quantifying classification performance of a trained combiner
- Evaluating ranking-based or continuous score outputs without thresholding to binary
  DEG calls, since F-measure requires discrete class predictions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose Naïve Bayes classifier combining multiple DEG
    detection method results is evaluated by F-measure on the spike-in dataset
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: Baseline single-method DEG caller whose spike-in performance the classifier's
    F-measure is compared against
- name: ExpHunterSuiteExamples
  role: Repository with code and reports for replicating the synthetic/spike-in experiments
    whose results are scored by F-measure
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:32e926d586c2b36c65176a3dff71e8a89431053aac7042600b5091846d3e689b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - ExpHunter Suite
  - ExpHunterSuiteExamples
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# f-measure-classification-metric-computation

## Summary

Computes the F-measure of DEG-calling methods or combined classifiers (e.g. the ExpHunter Suite Naïve Bayes model trained on simulated p-value vectors) against spike-in data with known DEG/non-DEG ground truth, enabling comparison against single methods like DESeq2 and vote-threshold combinations.

## When to use

Use when evaluating a DEG-calling method or combined classifier (e.g. Naïve Bayes over multi-method p-value vectors) against a spike-in dataset with known DEG/non-DEG truth labels, to quantify performance as an F-measure.

## When NOT to use

- The dataset has no known DEG/non-DEG ground truth (e.g. the Lafora disease or PMM2-CDG model datasets), so an F-measure cannot be computed — use consensus/vote-threshold heuristics instead
- The goal is choosing a single DEG detection method for a new dataset rather than quantifying classification performance of a trained combiner
- Evaluating ranking-based or continuous score outputs without thresholding to binary DEG calls, since F-measure requires discrete class predictions

## Inputs

- Per-gene binary DEG/non-DEG predictions from the method or classifier under evaluation (e.g. predict() output of naive_bayes_model.rds applied to spike-in p-value vectors)
- Spike-in dataset with known DEG/non-DEG ground-truth labels
- Optionally, predictions from baseline methods (DESeq2, vote-threshold cut-off of 2) for comparison

## Outputs

- F-measure value quantifying classification performance on the spike-in DEG/non-DEG labels
- Comparison of the F-measure against baseline single methods and combination strategies (Table 1-style evaluation)

## How to apply

Assemble per-gene predictions from the method under evaluation — e.g. predictions of the Naïve Bayes classifier trained on simulated p-value vectors, or DEG calls from a single method such as DESeq2 — and pair them with the spike-in dataset's DEG/non-DEG ground-truth labels. Treat DEG detection as binary classification and compute the F-measure (harmonic mean of precision and recall) over the DEG class. Benchmark the result against the reference values reported in the ExpHunter Suite evaluation: the Naïve Bayes classifier reached an F-measure of ~0.88 on the spike-in data, performing as well as the best single and combination methods; a minimum vote threshold cut-off of 2 was the optimum for the spike-in data in terms of F-measure, and DESeq2 was comparably strong. Report the F-measure alongside these baselines so the trained classifier's performance can be judged relative to Table 1-style comparisons.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose Naïve Bayes classifier combining multiple DEG detection method results is evaluated by F-measure on the spike-in dataset) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (Baseline single-method DEG caller whose spike-in performance the classifier's F-measure is compared against)
- **ExpHunterSuiteExamples** (Repository with code and reports for replicating the synthetic/spike-in experiments whose results are scored by F-measure) — https://github.com/fmjabato/ExpHunterSuiteExamples

## Evaluation signals

- F-measure on the spike-in dataset should be ~0.88 for the Naïve Bayes classifier trained on simulated p-value vectors
- Computed F-measure must be comparable to the Naive Bayes row of Table 1 (best performing single and combination methods) as reported for the spike-in data
- F-measure should be in the same range as the vote threshold of 2 and DESeq2 baselines on the spike-in data
- Predictions must be binary DEG/non-DEG labels aligned per-gene with the spike-in ground truth; misaligned gene IDs invalidate the metric
- Sensitivity to the decision threshold should be checked — the optimum vote threshold was 2 in terms of F-measure for the spike-in data

## Limitations

- The F-measure-based optimum vote threshold (2) was established on the spike-in data and may not generalize — 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- The expected ~0.88 F-measure is specific to the spike-in experiment; performance on the simulated Lafora-derived training data or real rare-disease datasets is not directly transferable
- Comparison is limited to the datasets in the paper's evaluation (simulated, spike-in, Lafora, PMM2-CDG)

## Evidence

- [discussion] the Naïve Bayes classifier presented here performed as well as the best performing single and combination methods on the spike-in dataset: [evidence span withheld — anchor status: not_found]
- [discussion] for the spike-in data the optimum threshold was 2 in terms of F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [abstract] Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [other] Apply the trained classifier to the spike-in dataset with known truth labels and compute its F-measure (expected ~0.88, comparable to the vote threshold of 2 and DESeq2): [evidence span withheld — anchor status: not_found]
