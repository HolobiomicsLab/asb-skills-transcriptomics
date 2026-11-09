---
name: f1-metric-benchmark-evaluation
description: Use when a classifier's predicted cell labels must be scored against
  held-out true labels, e.g. comparing Symphony-based predictions to benchmarked classifiers
  via per-cell-type F1 and median F1 across train-test experiments.
when_to_use_negative:
- No held-out true labels exist for the query cells, so F1 against ground truth cannot
  be computed
- The goal is novel cell type discovery in the query — Symphony does not replace de
  novo analyses and unmapped populations would make label-based F1 misleading
- Evaluating mapping/conservation quality (e.g., LISI or embedding geometry) rather
  than supervised classification accuracy
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Maps query cells into the reference harmonized embedding (buildReference,
    mapQuery) and transfers annotations via downstream classifiers (e.g., knnPredict
    with k=5) whose predictions are then F1-scored
  repo: https://github.com/immunogenomics/symphony
- name: tabdelaal/scRNAseq_Benchmark
  role: Provides the evaluate utility that returns accuracy, median F1-score, per-cell-type
    F1-scores, % unlabeled cells, and confusion matrix from true/pred label CSVs
  repo: https://github.com/tabdelaal/scRNAseq_Benchmark
- name: Harmony
  role: Performs the donor-variable integration underlying the Symphony reference
    whose embedding coordinates feed the classifiers being evaluated
  repo: https://github.com/immunogenomics/symphony
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:5d231d197879fe04ff71ea7e3152fcc8c800ce75d838b244d83e084a3a425500
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - tabdelaal/scRNAseq_Benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# f1-metric-benchmark-evaluation

## Summary

Benchmark-evaluation skill that scores predicted single-cell type annotations against true labels using per-cell-type F1-scores, taking the median F1 across cell types per train-test experiment and averaging across experiments. It is the evaluation utility used by the Abdelaal et al. classifier benchmark and reused to quantify Symphony-based classification on PbmcBench.

## When to use

Use when a classifier's predicted cell labels must be scored against held-out true labels, e.g. comparing Symphony-based predictions to benchmarked classifiers via per-cell-type F1 and median F1 across train-test experiments.

## When NOT to use

- No held-out true labels exist for the query cells, so F1 against ground truth cannot be computed
- The goal is novel cell type discovery in the query — Symphony does not replace de novo analyses and unmapped populations would make label-based F1 misleading
- Evaluating mapping/conservation quality (e.g., LISI or embedding geometry) rather than supervised classification accuracy

## Inputs

- CSV file of true cell type labels per test cell
- CSV file of predicted cell type labels per test cell
- Per-experiment train-test partitions (48 PbmcBench protocol/donor splits)
- Per-cell-type F1-scores from the classifier runs

## Outputs

- Accuracy, median F1-score, per-cell-type F1-scores, % unlabeled cells, and confusion matrix per experiment
- Per-experiment F1 summary table (e.g., f1_benchmark_results.csv)
- Average median F1 range across experiments for comparison against benchmark ranges (0.79–0.87 vs 0.77–0.83)

## How to apply

Run the classifier on each train-test split (e.g., 48 protocol/donor PbmcBench partitions where a held-out protocol or donor serves once as query), writing true and predicted labels to CSV per experiment. Call the evaluate utility from tabdelaal/scRNAseq_Benchmark on the true-label and pred-label CSVs to obtain accuracy, per-cell-type F1-scores, median F1, % unlabeled cells, and the confusion matrix. Take the median F1 across cell types per experiment, then average the 48 per-experiment medians. Compare the averaged range (Symphony classifiers 0.79–0.87 average median F1) against the reference benchmark range (top classifiers scmap-cell, SingleCellNet, SCINA at 0.77–0.83, Fig. 3a). Persist the per-experiment F1 table (e.g., f1_benchmark_results.csv) for reproducibility.

## Related tools

- **Symphony** (Maps query cells into the reference harmonized embedding (buildReference, mapQuery) and transfers annotations via downstream classifiers (e.g., knnPredict with k=5) whose predictions are then F1-scored) — https://github.com/immunogenomics/symphony
- **tabdelaal/scRNAseq_Benchmark** (Provides the evaluate utility that returns accuracy, median F1-score, per-cell-type F1-scores, % unlabeled cells, and confusion matrix from true/pred label CSVs) — https://github.com/tabdelaal/scRNAseq_Benchmark
- **Harmony** (Performs the donor-variable integration underlying the Symphony reference whose embedding coordinates feed the classifiers being evaluated) — https://github.com/immunogenomics/symphony

## Examples

```
result <- evaluate('f1_benchmark/true_labels.csv', 'f1_benchmark/pred_labels.csv')
```

## Evaluation signals

- evaluate() returns all expected fields: accuracy, median F1-score, F1-scores for all cell populations, % unlabeled cells, and confusion matrix
- Average median cell type F1 across the 48 PbmcBench experiments falls within 0.79–0.87, comparable to the top classifiers' reported 0.77–0.83 (Fig. 3a)
- A per-experiment F1 table (f1_benchmark_results.csv) is produced with one median-F1 row per train-test partition
- Each experiment's evaluation consumes exactly the true-label and pred-label CSVs output by that classifier run

## Limitations

- Annotation transfer from reference to query remains an open area of research; F1 against reference-defined labels penalizes genuinely novel query populations
- Mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone

## Evidence

- [readme] evaluate utility returns F1 summary: [evidence span withheld — anchor status: external_doc]
- [readme] evaluate input contract: "Evaluate the classifier prediction by 

R
result <- evaluate('~/Results/TM/scPred_True_Labels.csv', '~/Results/TM/scPred_Pred_Labels.csv')"
- [discussion] annotation transfer is benchmarked prediction task: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [intro] benchmark comparison context: "This is in contrast to automated cell type classifiers, such as scmap31, which assign rigid annotations based on reference datasets in a supervised manner"
- [methods] knnPredict produces scored predictions: "query = knnPredict(query,       # query object"
